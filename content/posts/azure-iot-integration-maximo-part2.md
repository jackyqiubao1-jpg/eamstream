---
title: "Azure IoT integration with Maximo — Part 2"
date: 2023-02-06
tags: [maximo, iot, azure, integration]
draft: false
---


In the previous [part 1](/posts/azure-iot-integration-maximo-part1/) blog, we built Azure IoT Hub in Azure cloud, developed an IoT device simulator running in local machine to connect and send ramdom temperature data to Azure IoT Hub , created an Azure function, and configure Azure event grid to pass through Azure IoT Hub telemetry data (temperature) to Azure Function. In Azure Function, we parsed event data to retrieve device and temperature information from event data and log them. In this part 2 blog, we will continue to configure and build the rest of the components inside the red rectangle in the following architecture diagram.

![](/images/1*URTLMTnozmN8Sa7UAuX9wg.png)

The following are the high level of steps to configure asset and meter data in Maximo, update Azure function to call Maximo NextGen Restful API to post temperature data which is generated from IoT Device Simulator to Maximo.

- Setup Asset and Meter in Maximo, and review MIF (Maximo Integration Framework), Maximo NextGen Restful API and Object Structure (OS).
- Update Azure Function to post temperature data which receive from Azure Event Grid to Maximo Asset Reading through Object Structure MXASSET Restful API.

The following steps assume that you have Maximo environment with a maxadmin user group permission which can access to OS MXASSET to query and update asset and meter reading data in Maximo.

- **Setup Asset and Meter in Maximo, and review MIF (Maximo Integration Framework), Maximo NextGen Restful API and Object Structure (OS).**

Login maximo, and open Asset application.

![](/images/1*P4Vr-DNbcrPhS7XQo_txRw.png)

Click New Asset or “+” icon at the top tool bar, to open new Asset page. For assetnum field, we will use the same value of DeviceID we created in Azure IoT Device in [Part 1](/posts/azure-iot-integration-maximo-part1/), which is TempSimulator. Fill in the rest of required fields on the new Asset page, click “Save” button.

![](/images/1*DGHhcZlFUlpibQYpxHtjxg.png)

Click Meters tab, and then click “New Row” button at the end of the page.

![](/images/1*4Vtsfy7hxNHD9CMZokWtyw.png)

Select “TEMP-C” meter to the meter field to assign TEMP-C meter to the Asset TempSimulator we created in previous step. And save the change.

![](/images/1*R-eoRRByGzBJ1Qj-uoEVxA.png)

Change Asset TempSimulator status to be Active by click “Change Status” under Common Actions section in the left panel, on the popup page, select Active as new status, and then click “OK” button to save the status change.

![](/images/1*UJkJyvTvvdV34MMKq2YfuQ.png)

Now we created a new Asset with assetnum=TempSimulator (Same as device ID in Azure IoT Device), and assign a new meter “TEMP-C” to this asset.

The following step we will leverage the out of box MXASSET Object Structure through Maximo Integration Framework (MIF) NextGen Restful API to post temperature data to Asset TempSimulator meter reading in Maximo. The following are the high level introduction of OS, MIF and Maximo NextGen Restful API. You can learn more detail information on them through the links if you are interesting.

(1) Object Structure (OS): An object structure is the common data layer that the integration framework components use for sending and receiving data in Maximo® Asset Management. An object structure consists of one or more related business objects that define the content of an integration message.
An object structure provides the message content for channels and services and enables application-based import and export. In addition, an object structure, on its own, can be called as a service, supporting the Create, Update, Delete, Sync, and Query operations.

[https://www.ibm.com/docs/es/mam/7.6.0?topic=os-modification-predefined-object-structure](https://www.ibm.com/docs/es/mam/7.6.0?topic=ic-object-structures)

We will use out of box OS MXASSET in Maximo to post temperature data through Maximo NextGen Restful API in Azure function. The following is the MXASSET OS setting in Maximo:

![](/images/1*v3dH2e0X29FtzA505DnXTw.png)

(2) Maximo Integration Framework (MIF): The Maximo Integration Framework (MIF) is an integral part of the Tivoli Process Automation Engine (TPAE) that allows the synchronization and integration of data and of applications between TPAE and external systems in real time or batch mode by using a variety of communication protocols.

[https://www.ibm.com/docs/en/maximo-eam-saas?topic=integrating-data-external-applications](https://www.ibm.com/docs/en/maximo-eam-saas?topic=integrating-data-external-applications)

(3) Maximo NextGen Restful API: Maximo NextGen REST APIs are a complete rewrite of the existing REST apis that was released around Maximo version 7.1. Maximo NextGen REST apis are in Maximo releases
starting 7602 version. The NextGen apis are often referred to as the REST/JSON apis for the end to end support for JSON data format.

[https://www.ibm.com/support/pages/system/files/inline-files/Maximo_NextGen_REST_API_-_nextgen.pdf](https://www.ibm.com/support/pages/system/files/inline-files/Maximo_NextGen_REST_API_-_nextgen.pdf)

2.** Update Azure Function to post temperature data which receive from Azure Event Grid to Maximo Asset Reading through Object Structure MXASSET Restful API.**

In the following section, we will update Azure function code in [Part 1](/posts/azure-iot-integration-maximo-part1/)to post temperate data from IoT device simulator to Maximo Asset meter reading through object structure MXASSET restful API.

Open Azure portal and locate Azure function EventGridDataEventParser we built in [part 1](/posts/azure-iot-integration-maximo-part1/).

![](/images/1*JB1hstSucezmLR_QlFoU7Q.png)

Click “Code + Test” in the left panel, update the run.csx code as the following:

```
#r "Azure.Messaging.EventGrid"
#r "System.Memory.Data"
#r "Newtonsoft.Json"

using Azure.Messaging.EventGrid;
using Newtonsoft.Json.Linq;
using Newtonsoft.Json;
using System.Net.Http;
using System.Net.Http.Headers;
using System;
using System.Text;
public static async  void Run(EventGridEvent eventGridEvent, ILogger log)
{
    var eventData=eventGridEvent.Data.ToString();
    log.LogInformation("Functionlog: "+eventData);
    JObject json = JObject.Parse(eventData);
    JToken token = json.SelectToken("systemProperties.iothub-connection-device-id");    
    var deviceID=token?.ToString();
    log.LogInformation("Device ID: "+deviceID);
    //token = json.SelectToken("systemProperties.dt-subject");  
    token = json.SelectToken("body.metername");    
    var meterID=token?.ToString();
    log.LogInformation("Meter ID: "+meterID);    
    token = json.SelectToken("body.temperature");
    var meterReading=token?.ToString();
    log.LogInformation("Meter Reading: "+meterReading);
    token = json.SelectToken("systemProperties.iothub-enqueuedtime");
    var eventTime=token?.ToString(Formatting.None);
    log.LogInformation("Event Time: "+eventTime);
    //call maximo api to get resouce uri
    var baseUrl = "http://{replace with your maximo server name}/maximo/oslc/os/mxasset";    
    HttpClient client = new HttpClient();
    //update the following maxauth header value to your maximo account value. 
    client.DefaultRequestHeaders.Add("maxauth","{User credentials encoded to base64 format}");
    var apimUrl=baseUrl+"?lean=1&oslc.where=assetnum=\""+deviceID+"\"";
    log.LogInformation($"Get Url: {apimUrl}");
   
    try
    {
        /*The following code is to query Asset from Maximo nextgen rest API 
        to get resource ID by assetnum (DeviceID in the event message body),
        which can be used to post meter reading to Maximo through nextgen Rest API.
        */
        using HttpResponseMessage result =await client.GetAsync(apimUrl);
        result.EnsureSuccessStatusCode();
        var content = await result.Content.ReadAsStringAsync();
        log.LogInformation($"Get Request result: {content}");         
        JObject jsonResult = JObject.Parse(content);        
        JToken tokenMember = jsonResult.SelectToken("member");    
        String strMembers= tokenMember?.ToString();
        log.LogInformation($"Query Member result: {strMembers}");         
        JArray jArrayResult= JArray.Parse(strMembers);
        if(jArrayResult is null || jArrayResult.Count==0)
        {
            log.LogError("\nNo result was found for Asset Num:"+deviceID);  
        } 
        else
        {
            JObject jObjecturl = jArrayResult.First.ToObject<JObject>();
            JToken tokenUrl=jObjecturl.SelectToken("href");
            log.LogInformation("Found Asset URL:"+tokenUrl?.ToString()); 
            String assetUrl=tokenUrl?.ToString();
            if(assetUrl.Length>0)
            {
                String[] strList=assetUrl.Split(new string[] { "/" }, StringSplitOptions.None);
                String resourceID=strList[strList.Length-1];
                log.LogInformation($"ResourceID:{resourceID}"); 
                //Post meter reading to Maximo
                dynamic jObjectMeter=new JObject();
                jObjectMeter.metername=meterID;
                jObjectMeter.linearassetmeterid=0;
                jObjectMeter.newreading=meterReading;
                jObjectMeter.newreadingdate=eventTime.Substring(1,19)+"+00:00";
                dynamic jObjectAsset=new JObject();
                dynamic jArrayMeters=new JArray();
                jArrayMeters.Add(jObjectMeter);
                jObjectAsset.assetmeter=jArrayMeters;
                String postJson=JsonConvert.SerializeObject(jObjectAsset);
                log.LogInformation($"Post Json Content: {postJson}");
                using (var postContent = new StringContent(postJson, System.Text.Encoding.UTF8, "application/json"))
                    {
                        var postUrl=baseUrl+"/"+resourceID+"?lean=1";
                        log.LogInformation($"Post Url: {postUrl}");
                        //add header for Maximo nextgen rest API update scenario 
                        client.DefaultRequestHeaders.Add("x-method-override","PATCH");
                        client.DefaultRequestHeaders.Add("patchtype","MERGE");                       
                        HttpResponseMessage resultPost =await client.PostAsync(postUrl, postContent);
                        resultPost.EnsureSuccessStatusCode();
                        var postResultContent = await resultPost.Content.ReadAsStringAsync();
                        log.LogInformation($"Post request result: {postResultContent}");
                    }
            }
            else
            {
                log.LogError($"Error: Asset Num:{deviceID} url is empty");  
            }
        } 
    }
    catch (HttpRequestException e)
    {
        log.LogError("\nHttp Exception Caught!");
        log.LogError("Message :{0} ", e.ToString());
    }
    catch (Exception e)
    {
        log.LogError("\nUnexpected Exception Caught!");
        log.LogError("Message :{0} ", e.ToString());
    }

}
```

There are 2 places you need to update on above code:

(1) *var baseUrl = “http://{replace with your maximo server name}/maximo/oslc/os/mxasset”;*

Update {replace with your maximo server name} to be your Maximo server name.

(2) *client.DefaultRequestHeaders.Add(“maxauth”,”{User credentials encoded to base64 format}”);*

Update* *{User credentials encoded to base64 format} to be your maximo user account and password in base64 format encode. For example, if your maximo user account is maxadmin, password is maxadmin, the user credential format is *maxadmin:maxadmin. *And then use base64 encryption tool (for example [https://www.base64encode.org/](https://www.base64encode.org/)) to encode the string *maxadmin:maxadmin. *The encode string is *bWF4YWRtaW46bWF4YWRtaW4=*

![](/images/1*gYhBL5FFeu6ksH9Jnh9BnQ.png)

Save the change of run.csx file. Click “Monitor” in the left panel, and then click Logs tab to open the log window.

![](/images/1*kvwl1MMvAGEtsPGiFXk2xg.png)

Run the IoT device simulator python code in the local machine to send random template data to Azure IoT hub.

![](/images/1*FKDlkvmg-ba_nIwkvhccpQ.png)

On the Azure function monitor logs window, we will receive these event data from Azure IoT hub through Azure Event Grid, and are parsed by the code in Azure function EventGridDataEventParser:

![](/images/1*7hlc5PGEyvdfe0xKaqf44w.png)

And open Asset application in Maximo, locate asset “TempSimulator”, click “Manage Meter Read History” under “More Action” section, and click “Characteristic and Gauge Meter Readings” tab, you will see the temperature data from IoT device simulator are posted to Maximo.

![](/images/1*5UKyzmNBWNA7PlC7vtjNNA.png)

In the [part 1](/posts/azure-iot-integration-maximo-part1/) & this part 2 blogs, we leveraged Azure IoT solution to created IoT device simulator in local machine to generate random temperature data, connect and send them to Azure IoT hub, through Azure Event Grid to pass telemetry to Azure Function. In Azure Function, call Maximo NexGen Rest API to post telemetry data to Maximo Asset meter reading.

Of course this POC can be expanded to add data analysis module in Azure Cloud, for example, all of the telemetry data will be stored in Azure Cosmos DB, build Azure machine learning module to analyze the telemetry data in Azure Cosmos, raise work order in Maximo through Maximo NextGen Restful API base on analysis result if need. This expansion solution will be covered in my future blog.

You can get full source code from GitHub:

If you think this blog helping you, welcome to buy me a coffee through below Give a Tip button after this blog :)

---
