---
title: "Introduction of Maximo NextGen Rest APIs — Part 2 Update Maximo Data"
date: 2023-03-30
tags: [maximo, rest-api, integration]
draft: false
---


In previous blog “[Introduction of Maximo NextGen Rest APIs — Part 1 Query Maximo Data](https://jackyqiubao.medium.com/introduction-of-maximo-nextgen-rest-apis-part-1-query-maximo-data-6c082087e8f2?sk=a3eb2b199c4fabb83a54c901376e2370)”, I introduced how to query Maximo data through NextGen Rest APIs. In this blog, we will have experiment on how to update Maxim data in through NextGen Rest APIs.

We will continue using the MXASSETSPEC OS we built in previous blog for the this experiment. If you don’t get chance going through my [previous blog](https://jackyqiubao.medium.com/introduction-of-maximo-nextgen-rest-apis-part-1-query-maximo-data-6c082087e8f2?sk=a3eb2b199c4fabb83a54c901376e2370), please follow the section “**2. Object Structure (OS)” **to create MXASSETSPEC OS first.

On this blog, we will cover the following scenarios through Maximo NextGen Rest APIs:

(1) Create

(2) Update

(3) Delete

(4) Action

Before getting start, we might need to understand the essential http methods in Restful: Get, Post, Put, Patch and Delete. Please go through this link if you are not familiar with it.

[https://www.techtarget.com/searchapparchitecture/tip/The-5-essential-HTTP-methods-in-RESTful-API-development](https://www.techtarget.com/searchapparchitecture/tip/The-5-essential-HTTP-methods-in-RESTful-API-development)

Before we start the experiment, we need to make some changes to OS MXASSETSPEC to allow update asset status through the inbound message. Open Object Structure application in Maximo, locate “MXASSETSPEC” OS and open it. Click “Inbound Setting Restrictions” at the left panel, and click on “ASSET” in the MBOs list. In the Inbound Setting Restrictions for MBO ASSET list, STATUS and STATUSDATE fields are default to be noneditable (Restricted checkbox is checked). Check Override checkbox for both of the fields, and then uncheck Restricted checkbox. Save the change by click OK button.

![](https://cdn-images-1.medium.com/max/800/1*xdf_fG7mVPSMaRIgbf3sSw.png)

**1. Create**

Creating resources are almost always done using the collection URI— the same uri you would use to query the resources. For example, we can use the following URI to create an Asset along with Asset Specfication with the OS MXASSETSPEC we built in the [*part 1 blog*](https://jackyqiubao.medium.com/introduction-of-maximo-nextgen-rest-apis-part-1-query-maximo-data-6c082087e8f2?sk=a3eb2b199c4fabb83a54c901376e2370).

[*http://*](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)*{*[*your maximo server url*](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)*}*[*/maximo/oslc/os/mxassetspec?lean=1*](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)

Open the postman, create a new tab, add a new line for ““maxauth” and base64 encryption string of user name and password on headers tab (please refer to Authentication section in the [*part 1 blog*](https://jackyqiubao.medium.com/introduction-of-maximo-nextgen-rest-apis-part-1-query-maximo-data-6c082087e8f2?sk=a3eb2b199c4fabb83a54c901376e2370)*)*

![](https://cdn-images-1.medium.com/max/800/0*N3ROOcNuB95YXwvl.png)

At the new row of the Headers, add a new key “properties”, value as “*”. This setting is used to ask Maximo NextGen Rest API to return all of the information for the new Asset is created by the payload. Of course you can restrict the return information by identify the columns in the value of this properties header.

![](https://cdn-images-1.medium.com/max/800/1*KxEo5ZpqL56B9dfGapF5FA.png)

Paste the below url in postman address textbox, change *{*[*your maximo server url*](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)*} *to your Maximo server name and port.

[*http://*](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)*{*[*your maximo server url*](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)*}*[*/maximo/oslc/os/mxassetspec?lean=1*](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)

Click Body tab, click on “raw” option, select json from the format dropdown list. Paste the blew payload to the body text box.

```
{
    "siteid": "BEDFORD",
    "assetnum": "DEMOASSET",
    "description": "NextGen Rest API Demo asset",
    "status": "ACTIVE",    
    "classstructureid":1015
}
```

Select the method to be “POST”, Click “Send” button, you will see the response message along with 201 Created status.

![](https://cdn-images-1.medium.com/max/800/1*idb5fZntAdLoPI3wni9RIQ.png)

In the response content, one of the information we need to pay attention is the new asset resource ID, which we need to use it for the rest of experiment on update and delete scenario. You can get the new asset resource ID from the following node in the response content:

*“href”: “http://MaximoSV.upgk5qkznuru1ao1jz40nyn2pb.bx.internal.cloudapp.net:80/maximo/oslc/os/mxassetspec/_REVNT0FTU0VUL0JFREZPUkQ-"*

The string “*_REVNT0FTU0VUL0JFREZPUkQ-” *after OS in above node is the new asset resource ID.

Let’s open Asset application in Maximo and locate asset “DEMOASSET” to validate the new asset.

![](https://cdn-images-1.medium.com/max/800/1*pQddfDRHaAaCq0X-J-scXg.png)

And click Specifications tab, we can see the classstructureid=1015 (Classification as TTAXLE) is assigned to the new asset DEMOASSET.

![](https://cdn-images-1.medium.com/max/800/1*Utd7BPez-Tg_A0vxa_m5Zw.png)

For the next experiment, we will demostrate how to update Asset Specifications through NextGen APIs.

**2. Update**

Let’s go back to Postman Headers tab, append a new header “x-method-override” with the value of “PATCH”. This instructs the server side to update the resource.

![](https://cdn-images-1.medium.com/max/800/1*AoLxBIc4Q7fo5KfjO6sabg.png)

Click Body tab, update the payload as following. In the updated payload, we will update the values Asset Specification “FUELTP” and “FUELCAP”.

```
{
    "siteid": "BEDFORD",
    "assetnum": "DEMOASSET",
    "description": "NextGen Rest API Demo asset",
    "status": "ACTIVE",    
    "classstructureid":1015,
    "assetspec": [
                {
                    "assetattrid": "FUELTP",
                    "alnvalue": "#93",
                    "linearassetspecid":0
                },
                {
                    "assetattrid": "FUELCAP",
                    "numvalue": 15,
                    "linearassetspecid":0
                }
    ]
}
```

Update url to append Asset Resource ID “*_REVNT0FTU0VUL0JFREZPUkQ-” *which we retrieved it from Create scenario response content after the mxassetspec OS:

*http://{your maximo server url}/maximo/oslc/os/mxassetspec/_REVNT0FTU0VUL0JFREZPUkQ-?lean=1*

Click Send button, the following response along with Status 200 will be returned.

![](https://cdn-images-1.medium.com/max/800/1*qhWCYcliqgszQmi7s5HQjQ.png)

Now let’s go back to Asset application in Maximo and locate asset “DEMOASSET”, click Specifications tab, you will see the values Asset Specifications “FUELTP” and “FUELCAP” were updated.

![](https://cdn-images-1.medium.com/max/800/1*Zg7ZcfujCXzYn3kb9ls1gg.png)

But if you compare this screenshot with the screenshot when the Asset created, you might notice other 3 Specifications which was not in the payload were removed.

To avoid remove the child objects which are not in the payload to be removed, we can use additional header “patchtype” with the value of “MERGE”. Now let’s add this header to the postman headers:

![](https://cdn-images-1.medium.com/max/800/1*H_J3vcCZPPJRc8WoSd17Ug.png)

And in the payload, remove “FUELTP” specification node from the payload, update “FUELCAP” specification’s value to be 21.

```
{
    "siteid": "BEDFORD",
    "assetnum": "DEMOASSET",
    "description": "NextGen Rest API Demo asset",
    "status": "ACTIVE",    
    "classstructureid":1015,
    "assetspec": [                
                {
                    "assetattrid": "FUELCAP",
                    "numvalue": 15,
                    "linearassetspecid":0
                }
    ]
}
```

Click send button to post the payload. If return status code is 200, open Asset application in Maximo and locate asset “DEMOASSET”, click Specifications tab, we will see “FUELTP” specification is still in the list even it is not included in the payload, and the “FUELCAP” specification’s value was updated to be 21.

![](https://cdn-images-1.medium.com/max/800/1*w4_p8ilbn5JSL8cM24q1uA.png)

3. Delete

To delete child object, for our example, to delete selectively asset specifications, we can use action in the payload. The following payload is to used to remove asset specification “FUELCAP”.

```
{
    "siteid": "BEDFORD",
    "assetnum": "DEMOASSET",
    "description": "NextGen Rest API Demo asset",
    "status": "ACTIVE",    
    "classstructureid":1015,
    "assetspec": [                
                {
                    "assetattrid": "FUELCAP",
                    "numvalue": 21,
                    "linearassetspecid":0,
                    "_action":"Delete"
                }
    ]
}
```

After post the above payload, the asset “DEMOASSET” specification “FUELCAP” was deleted.

![](https://cdn-images-1.medium.com/max/800/1*BK-bvObtfqFA_bpnCOBIWA.png)

To delete MBO, for example, if we want to delete “DEMOASSET”, we can use DELETE Method. Let’s change the payload method to be “DELETE” in postman, and update the payload as following:

```
{
    "siteid": "BEDFORD",
    "assetnum": "DEMOASSET",
    "description": "NextGen Rest API Demo asset",
    "status": "ACTIVE",    
    "classstructureid":1015
}
```

You will receive 204 status since asset is deleted if the payload is processed successfully.

![](https://cdn-images-1.medium.com/max/800/1*Jn4aG1F6wyTAy65NdEOYIQ.png)

And open Asset application in Maximo, try to locate Asset “DEMOASSET”, you will not be able to find it.

**4. Action**

Sometime back in Maximo we had introduced the functionality of “Standard Services”. The name may sound confusing the but the intent was fairly simple. Maximo has a bunch of application services — aka Appservices which provide a bunch of service methods that act on MBOs to perform some business task. While most of these calls end up modifying the state of the system, some of these calls are merely for getting information. We have leveraged the JSR 181 annotations to expose these methods as “WebMethods” which can be accessed using SOAP and REST calls. For the purpose of this discussion, we are only going to focus on the RESTFul aspects of these methods. Simply put, we can add a WebMethod to any existing Maximo App service by extending the service class and adding a method like below. This example is from say extending the Workorder Service (psdi.app.workorder.WOService):

```
@WebMethod
public void approve(@WSMboKey(value="WORKORDER") MboRemote wo, String memo)
{
wo.changeStatus("APPR",MXServer.getMXServer().getDate(),memo);
}
```

Next you want to make sure that your method is now available for REST API calls. To do that you can select any workorder record and use the rest api to get the set of allowed actions on that resource. For example the query below will do the job.

GET [*http://{your*](http://%7Byour)* maximo server url}/oslc/os/mxwo?oslc.where=wonum=”1001”&oslc,select=allowedactions*

You will see the list of allowed actions, which are nothing but the list of WebMethods for the service corresponding to the root object of the Object Structure along with the list of actions registered with the Object structure (using the application menu “Action Definition”). Theresulting json would provide the json schema for web method as well as the HTTP method and the name of the action, which can be used to invoke the action. A sample call below will demonstrate how these methods can be invoked.

POST <uri of the workorder 1001>?action=wsmethod:approve

x-method-override: PATCH

```
{
"memo":"Testing"
}
```

Note we need to set the request header x-method-override as PATCH for invoking this action api, as this is operating on an existing Mbo (the first parameter to the method). As expected, this MBO reference is derived from the request URI (say — uri of the workorder 1001) and passed into the method as part of the api invocation. The json schema for the payload will reflect the parameter data types (in the method) and the parameter names.

As you can figure out, the query parameter action=wsmethod:approve is the fully qualified name of the action. The format is <action type>:<name> where name is the java method name for the action type wsmethod.

Let’s recap what we discuss on the above experiments. We learn how to use Maximo NextGen Rest API to create, update and delete data, and how to call OS action for the open web methods.

If you think this blog helping you, welcome to buy me a coffee through below Green *Give a Tip* button :)

---
*Originally published on [Medium](https://medium.com/@jackyqiubao/introduction-of-maximo-nextgen-rest-apis-part-2-update-maximo-data-e3db83751709).*
