---
title: "Turn Your Lego EV3 Robot to Azure IoT Device — Part 2 create work order in Maximo"
date: 2023-03-23
tags: [iot, azure, maximo, integration]
draft: false
---


In previous part 1 blog, we demonstrated how to connect Lego EV3 to Azure IoT hub through MQTT, and send EV3 color sensor data to IoT hub. In this blog, we will experiment to update Azure Function which we built on blog “[Azure IoT integration with Maximo — Part 2](https://jackyqiubao.medium.com/azure-iot-integration-with-maximo-part-2-1200d2e2ad5b?sk=b79fe9dca04db0ce3d69e27f9471a571)” to create work order in Maximo through Maximo NextGen Rest API based on the color sensor data sent from Lego EV3.

The following is the high level of the steps we will following in this experiment:

(1) Configure required data in Maximo, such as Asset, Object Structure (OS).

(2) Update Azure Function to parse color sensor telemetry data sent by Lego EV3, and call Maximo NextGen Rest API to create work order if the Lego EV3 robot detected color is red.

#### **1. Prepare data and configuration in Maximo**

Open Asset application in Maximo, click “New Asset” Link or button. Input EV3 as ASSETNUM, and then Save the change.

![](https://cdn-images-1.medium.com/max/800/1*aDqqzMF-ZbnFMXKLeutpSw.png)

Change Asset Status to be Active

![](https://cdn-images-1.medium.com/max/800/1*IlMG_oqdiQuWtxigpgtnaA.png)

Open Object Structure application in Maximo, click “New Asset” Link or button to create a new Object Structure. Object Structure (OS) is composed by Maximo Business Objects (MBO). OS is the foundation in Maximo Integration Framework, you can understand MBO as table in database, and OS as view in database which combines multiple tables (MBOs) to form the data you would like to explore to external system for integration. We will create an OS called MXEV3WO for this experiment.

![](https://cdn-images-1.medium.com/max/800/1*c6nQU2sJKBG99blbEcKcww.png)

In Source Objects section, we will use two MBOs for this MXEV3WO OS: Workorder and WOActivity. In the blew steps, we will configure this OS to allow us create a work order and a task through Maximo NextGen Rest API.

Click New Row button under Source Objects section, select WORKORDER MBO from the popup list.

![](https://cdn-images-1.medium.com/max/800/1*l17wLmvW5HW5eVE3jIsA7g.png)

Click New Row button again, select WOActivity MBO from the list, select WORKORDER at Parent Object field, and pick up NEWACTIVITY in the Relationship list. It will join WOActivity and Workorder MBOs through NEWACTIVITY relationship. If you are familiar with relationship database sql query, it is similar to use left join the Workorder and WOActivity tables in the sql query.

![](https://cdn-images-1.medium.com/max/800/1*WcmBkVjBxT7DwM0UXBq5eg.png)

The purpose we build this OS is to use it as the structure to create workorder, and a new task under this work through Maximo NextGen Rest API.

#### 2. Update Azure function

Save above changes in Maximo, open Azure portal to open Azure Function. We will update Azure Function which we built in my previous serial blogs “[Azure IoT integration with Maximo — Part 1](https://jackyqiubao.medium.com/azure-iot-integration-with-maximo-part-1-b141c4210db0?sk=5345c390bd704aff8349231bbef7ec7d)” and “[Part 2](https://jackyqiubao.medium.com/azure-iot-integration-with-maximo-part-2-1200d2e2ad5b?sk=b79fe9dca04db0ce3d69e27f9471a571)”. If you don’t get chance to read it yet, please follow the instruction on that serial blogs to create an Azure Function “EventGridDataEventParser” first. I will not repeat the same steps in this blog.

Open “EventGridDataEventParser” Azure function, click on the “Code + Test”

![](https://cdn-images-1.medium.com/max/800/1*ZpbF5ytqy1OwKpR2uvuGRA.png)

Modify the “EventGridDataEventParser” Azure function to be following code. The comments in the code should explain the logic of the code.

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
    //token = json.SelectToken("body.temperature");
    token = json.SelectToken("body.telemetry");
    var meterReading=token?.ToString();
    log.LogInformation("Meter Reading: "+meterReading);
    token = json.SelectToken("systemProperties.iothub-enqueuedtime");
    var eventTime=token?.ToString(Formatting.None);
    log.LogInformation("Event Time: "+eventTime);
    if(String.Equals(meterReading.ToUpper(),"RED"))
    { //Warning: EV3 color sensor detects red color, need to raise work order in Maximo to fix the device
        
        var baseUrl = "http://{replace with your maximo server name}/maximo/oslc/os/mxev3wo?lean=1";    
        HttpClient client = new HttpClient();        
        //update the following maxauth header value to your maximo account value. 
        client.DefaultRequestHeaders.Add("maxauth","{User credentials encoded to base64 format}");
        try
        {
            //The following code is to post work order payload against Object Structure (OS) MXEV3WO to Maximo through nextgen Rest API, 
            //to create a new work order and a task under the work order

            //Structure Work Order Json information
            dynamic jObjectWO=new JObject();
            eventTime=eventTime.Substring(1,eventTime.Length-2);
            //Default wonum format as "WO_EV3_{eventTime}"
            jObjectWO.wonum="WO_EV3_"+eventTime;
            jObjectWO.description="workorder for EV3 at "+eventTime;
            //Default Site ID to be BEDFORD
            jObjectWO.siteid="BEDFORD";
            //Default work order status WAPPR
            jObjectWO.status="WAPPR";
            jObjectWO.assetnum=deviceID;
            //Structure WOActivity information inside the work order
            dynamic jArrayWOActivities=new JArray();
            dynamic jObjectWOActivity=new JObject();
            jObjectWOActivity.taskid=5;
            jObjectWOActivity.wosequence=5;
            jObjectWOActivity.description="Fixed device for detecting red color";
            jArrayWOActivities.Add(jObjectWOActivity);
            //Assign WOActivity to work order
            jObjectWO.woactivity=jArrayWOActivities;
            String postJson=JsonConvert.SerializeObject(jObjectWO);
            log.LogInformation($"Post Json Content: {postJson}");
            using (var postContent = new StringContent(postJson, System.Text.Encoding.UTF8, "application/json"))
            {
                //add header for Maximo nextgen rest API update scenario                                    
                HttpResponseMessage resultPost =await client.PostAsync(baseUrl, postContent);
                resultPost.EnsureSuccessStatusCode();
                var postResultContent = await resultPost.Content.ReadAsStringAsync();
                log.LogInformation($"Post request result: {postResultContent}");
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
    
}
```

There are 2 places you need to update on above code:

(1) *var baseUrl = “http://{replace with your maximo server name}/maximo/oslc/os/mxev3wo”;*

Update {replace with your maximo server name} to be your Maximo server name.

(2) *client.DefaultRequestHeaders.Add(“maxauth”,”{User credentials encoded to base64 format}”);*

Update* *{User credentials encoded to base64 format} to be your maximo user account and password in base64 format encode. For example, if your maximo user account is maxadmin, password is maxadmin, the user credential format is *maxadmin:maxadmin. *And then use base64 encryption tool (for example [https://www.base64encode.org/](https://www.base64encode.org/)) to encode the string *maxadmin:maxadmin. *The encode string is *bWF4YWRtaW46bWF4YWRtaW4=*

![](https://cdn-images-1.medium.com/max/800/0*GqAhWKxZZn2YfHqa.png)

Save the change of run.csx file. Click “Monitor” in the left panel, and then click Logs tab to open the log window.

![](https://cdn-images-1.medium.com/max/800/0*0WBSpIytQcDvTvpm.png)

Turn your Lego EV3 on and connect to network, and open VS Code to open folder we built for EV3 in previous blog “Turn Your Lego EV3 Robot to Azure IoT Device — Part 1”. Connect EV3 to VS code through click “Click here to connect to a device” under EV3DEV Device Browser.

![](https://cdn-images-1.medium.com/max/800/1*wog0Drw1351xSk0P-5zBXA.png)

Select your EV3 Device from the list, once it is connected, expand the folder node to locate main.py file

![](https://cdn-images-1.medium.com/max/800/1*u774OwIbtXLboLfS_HgbfA.png)

Right click on main.py file, select “Run” to execute the main.py in Ev3 device. On the VS Code output window will be able to see the log as:

![](https://cdn-images-1.medium.com/max/800/1*c7IIEjL75aqLzu5JteZfpw.png)

Put some red color object under EV3 Color sensor

![](https://cdn-images-1.medium.com/max/800/0*YckX9nR8mezU-5pI)

You should be able to see some red color telemetry data to be sent to IoT hub. Let’s go back to Azure Function Monitor Log tab, we should be able see the log

![](https://cdn-images-1.medium.com/max/800/1*UVAP0modyxMGpY7-wOcx-A.png)

And if we open Work Order Tracking application in Maximo, filter wonum with “WO_EV3%”, we will be able to find a work order was created through Azure Function.

![](https://cdn-images-1.medium.com/max/800/1*QW3rTCQvtxfxNxv_noChrA.png)

Click the work order link to open the work order, and then click Plan tab, you will see there is a Task being created under this work order.

![](https://cdn-images-1.medium.com/max/800/1*iTqaM0PCyuaOnzvL08enhw.png)

Let’s recap what we learn in this experiment. We setup Object Structure (OS) to accept payload to create work order and task inside it in Maximo, in Azure Function create work order through Maximo NextGen Rest API when we receive telemetry color data is red from Lego EV3. Of course you can extend this experiment to your real business case to build Azure IoT solution integration with Maximo or your EAM solution.

If you think this blog helping you, welcome to buy me a coffee through below Green *Give a Tip* button :)

---
