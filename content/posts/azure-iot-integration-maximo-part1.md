---
title: "Azure IoT integration with Maximo — Part 1"
date: 2023-02-06
tags: [maximo, iot, azure, integration]
draft: false
---


IoT (Internet of Things) is getting more and more popular in the world, especially when 5G technology was rolled out to our real life and industries, more and more IoT use cases were adopted in different industries. In the following content, I would like to introduce a POC to leverage IoT technology in EAM (Enterprise Asset Management) area.

IBM Maximo is the major player in EAM market, and Azure IoT is the most popular IoT platform, In this POC, I would introduce how to integrate Azure IoT platform with Maximo through Maximo NextGen Rest API. The purpose of this POC is to demonstrate sending IoT device sensor data (for example, temperature data) to Azure IoT hub, through Azure Event Grid to pass telemetry to Azure Function. In Azure Function, call Maximo NexGen Rest API to post telemetry data to Maximo Asset meter reading. The following is the high level of the architecture diagram:

![](/images/1*nWfVeFLfhBx3q-ZUc6WdZA.png)

*Diagram 1 — High level architecture diagram*

If you would like to follow this instruction to have hand-on experience on it, the following is the prerequisites:

(1) An Azure account. You can register an Azure account with $200 credit through this link:[https://azure.microsoft.com/en-us/free/](https://azure.microsoft.com/en-us/free/)

(2) Maximo environment. (Only require in [Part 2](/posts/azure-iot-integration-maximo-part2/))

If you are only interested in how to send IoT device sensor data (for example, temperature data) to Azure IoT hub, through Azure Event Grid to pass telemetry to Azure Function, this part 1 will provide step by step instruction along with all the source code.

In this Part 1 content, we will focus on creating an IoT device simulator to connect to Azure IoT hub and send temperature data to IoT hub. Once Azure IoT hub receives telemetry data from IoT device, will trigger Azure Event Grid to send the event to Azure Function. In Azure Function, we will parse the event data to get telemetry data (temperature data from simulate IoT device).

In [Part 2](/posts/azure-iot-integration-maximo-part2/) blog, we will update Azure function to call Maximo NextGen Rest API to post temperature data to Maximo Asset meter reading table.

The following are the high level steps we will follow to build IoT device simulator connect and send telemetry data to Azure IoT Hub, and trigger event in Azure Event Grid to send to Azure Function, in Azure Function, we will parse the event data to get the telemetry data is sent from IoT device simulator.

- Setup Azure IoT Hub
- Create IoT Device in IoT hub, and build IoT device simulator code in local environment to simulate IoT device sending temperature data to Azure IoT Hub.
- Create Azure Function to receive event from IoT Hub.
- Setup Azure Event Grid to connect to Azure IoT Hub to receive telemetry events, and pass event message to Azure Function.
- Develop code in Azure Function to parse event message to get telemetry related data.

Let’s getting start to setup Azure hub in Azure portal (Assume you already created / had Azure account).

- **Setup Azure IoT Hub**

(1) Login Azure Portal through [https://portal.azure.com/](https://portal.azure.com/)

(2) Click “Create a resource” link under Azure services section:

![](/images/1*nl0hkWmtbnYMJNXAQkY_5w.png)

(3) Type IOT Hub in the search box and click “Create” link under IOT Hub Azure Service.

![](/images/1*hoItELIz9MrnDWs3M-4PtQ.png)

(4) Fill in required fields in the IoT hub creation page, and click “Review + create” button.

![](/images/1*VVHlppIpSCwukb64GWu-iA.png)

(5) Click “Create” once you review all of information:

![](/images/1*CBmkm6kgTEAMqTYwJ2NZEA.png)

(6) After Azure IoT hub creation complete, you should be able find it from your resources list in your Azure portal

![](/images/1*1oir04P-0x1-ytlqmvfQLg.png)

**2. Create IoT Device in IoT hub, and build IoT device simulator code in local environment to simulate IoT device sending temperature data to Azure IoT Hub.**

The following steps will create a new device in the Azure IoT Hub we created in previous steps, and develop device simulator python code to simulate IoT device from local machine to connect and send data to IoT Hub through IoT device connection string.

(1) Click at the IoT hub “JACKYIOTHUBDEMO” we just created, click the “Devices” link under Device management at the left panel, and then click + Add Device link to add a new IoT device.

![](/images/1*j2ibAQnYzo2tlB-ZdlCMSg.png)

(2) On the create device screen, we will create a device call TempSimulator, we will select Symmetric Key as the Authentication Type. Don’t recommend this Authentication Type for production environment. Would suggest to use X.509 certificate for production. You can refer to this offical link for more detail on it: [https://learn.microsoft.com/en-us/azure/iot-dps/concepts-device-oem-security-practices](https://learn.microsoft.com/en-us/azure/iot-dps/concepts-device-oem-security-practices). Click Save button to add this device to IoT hub.

![](/images/1*aiyoSXSl0I6u4YZLafcVCg.png)

Once the new device creation complete, click refresh button, you should be able to find it in the device list:

![](/images/1*6_wPzssQk2DP_C0cfPIhpA.png)

(3) In the following instruction, we will develop a simulator of IoT device in Python in the local machine (Windows 10) to connect and send out random temperature data to Azure IoT Hub. We will simplify our development through leveraging Azure IoT Device SDK. You can get more detail information through Microsoft offical site “[https://learn.microsoft.com/en-us/azure/iot-develop/about-iot-sdks](https://learn.microsoft.com/en-us/azure/iot-develop/about-iot-sdks)” on Azure IoT Device SDK. And we will use Visual Studio Code as code editor.

To start the following section, you need to install the following prerequisites in your local machine, assume it is windows 10.

(I)Python 3.7+. To ensure that your Python version is up to date, run `python --version`

(II) Install Visual Source Code (VS Code). You can download the latest version of VS Code from [https://code.visualstudio.com/download](https://code.visualstudio.com/download)

And install VS Code Python extension. Refer to [https://code.visualstudio.com/docs/python/python-tutorial](https://code.visualstudio.com/docs/python/python-tutorial)

(III) Install IoT Device SDK. Open command window with Administrator permission, execute the below command:

```bash
pip install azure-iot-device
```

After above prerequisites installations were done, Open VS Code, and click “File” -> “Open Folder”, Locate an empty folder where you would like to hold your IoT Device simulator Python code.

Click “File”-> “New File” in VS Code after open the empty folder, Click Python File Python option.

![](/images/1*Cp_0qWxF8caGLusFERMcfw.png)

It will open a new empty Python file, copy the below code and paste it to the new Python file.

```
# -------------------------------------------------------------------------
# Copyright (c) Jacky Qiu. All rights reserved.
# Licensed under the MIT License. 
# --------------------------------------------------------------------------
import asyncio
import time
import uuid
import random
import json
from azure.iot.device.aio import IoTHubDeviceClient
from azure.iot.device import Message


async def send_recurring_telemetry(device_client):
    # Connect the client.
    await device_client.connect()

    # Send recurring telemetry
    i = 0
    while True:
        i += 1
        curr_temp= random.randrange(10, 50)
        temperature_msg = {"temperature": curr_temp,"metername":"TEMP-C"}
        msg = Message(json.dumps(temperature_msg))
        msg.message_id = uuid.uuid4()
        msg.correlation_id = "correlation-1234"
        #msg.custom_properties["tornado-warning"] = "yes"
        msg.content_encoding = "utf-8"
        msg.content_type = "application/json"
        print("sending message #" + str(i))
        await device_client.send_message(msg)
        time.sleep(10)


def main():
    # The connection string for a device, you can get it from your IOT Hub -> Devices
    conn_str = "{Your Device Connection String}"
    # The client object is used to interact with your Azure IoT hub.
    device_client = IoTHubDeviceClient.create_from_connection_string(conn_str)

    print("IoTHub Device Client Recurring Telemetry Sample")
    print("Press Ctrl+C to exit")
    loop = asyncio.get_event_loop()
    try:
        loop.run_until_complete(send_recurring_telemetry(device_client))
    except KeyboardInterrupt:
        print("User initiated exit")
    except Exception:
        print("Unexpected exception!")
        raise
    finally:
        loop.run_until_complete(device_client.shutdown())
        loop.close()


if __name__ == "__main__":
    main()
```

The above code is to simulate an IoT device to connect and send temperature data to Azure IoT Hub for every 10 seconds. You can adjust intervals by change the number in this line:

```
time.sleep(10)
```

And in the above code, you need to update the device connecting with your IoT device connection string in Azure IoT Hub. Login your Azure portal, open IOT Hub “JACKYIOTHUBDEMO” created in Step 1 , under Device management section, click “Devices” to open device list.

![](/images/1*_8VQd73CyNluXOrHXuCn9Q.png)

Click the device “TempSimulator” in the device list to open device detail information. At the end of Primary connection string field, click copy button to copy device connection string.

![](/images/1*2TcY1gFIeSdLI2U-DhzXlg.png)

Now let’s switch back to VS Code to update connection string in the IoT device simulator Python code. Replace the content at *{Your Device Connection String} *with the connection string copied from IoT Device Primary connection string.

*conn_str = “{Your Device Connection String}”*

Save Python code in VS code, for example IoTDevice.py. Select VS Code menu “Run”-> “Start Debugging”, or just press F5, will start to debug the IoTDevice.py code. And select Python File option.

![](/images/1*opDJmjkryj6MEE9movn8zg.png)

After the code execute, in the VS Code terminal window, you should see the random temperature data is created and sent to Azure IoT hub through the IoT device simulator python code.

![](/images/1*Atbr_wS4QezGxLyD0FqO4w.png)

Now if you go back to your Azure portal, locate Azure IoT hub, at the IoT hub overview page, you should see some message has been sent from your local IoT device simulator code to Azure IoT hub:

![](/images/1*vpdyieirrj7-N4UhAovvoA.png)

Now we completed building IoT device simulator, connected it to Azure IoT hub, and sent simulate temperature telemetry data to IoT hub. Next step we will create Azure Event Grid to receive the event from Azure IoT Hub when it receive telemetry data from IoT Device, and pass the event data to Azure Function which will parse event data to retrieve the information such as device id, temperature data, etc. Since creating Event Grid instance is required to define an end point which is Azure Function, we will create an Azure function first.

3. Create Azure Function to receive event from IoT Hub.

Click “Create a resource” link in your Azure portal, type “function app” in the search box, click Create button under Function App in the search result list.

![](/images/1*klW_J9QEwmbJ--3Jiemt2g.png)

On Create Function App page, input the information and click “Review + Create” button. And then click “Create” button to create a new Azure Function. (Note: Consumption (serverless) option in Plan Type is good enough for testing purpose. You should select other advance plan for your production environment)

![](/images/1*fRtbSNCC40WhJJBgd09EsQ.png)

After Azure Function App deployment complete, you can find it in the resource list through the resource group you assigned it to.

![](/images/1*m7oax_uN3nvTzNMvFqX4hg.png)

Click Azure Function App we just created, select Functions in the left panel, and then click “+ Create” to create a new function.

![](/images/1*wZkTcYLoWcEZcd-KwHveyA.png)

On Create function page, we will select development environment as “Development in portal”. In actual project, usually we will use VS Code or other code editor for Azure function development. Scroll down the template list to select Azure Event Grid trigger, fill in New Function name, and then click Create button to create a empty function.

![](/images/1*Ewyxtw965a82QjCgzKW4zA.png)

After new function creation complete, you can click “Code + Test” at left panel to open online editor to modify Azure function code. We will keep the default code at run.csx, and update this function later.

![](/images/1*e1Xolur-IPwRQng-S9wsDw.png)

Now let switch back to Azure IoT Hub “JACKYIOTHUBDEMO”.

**4. Setup Azure Event Grid to connect to Azure IoT Hub to receive telemetry events, and pass event message to Azure Function.**

In Azure IoT Hub, click Events in the left panel, and then click “+ Event Subscription” to create a new Azure Event Grid Subscription.

![](/images/1*v8FLSfOMLvDcaajdubxyeg.png)

In the create event subscription page, fill in the required fields.

![](/images/1*s4mI8knOEKsN-JE8e5hXaA.png)

In the Event Types list, since we are only interested in Device Telemetry event data, will only select Device Telemetry option.

![](/images/1*bpNUwQAgAemiyWDqDLgEEw.png)

At Endpoint Detail section, select Azure Function as Endpoint Type

![](/images/1*UUZj_ZGuT1iBmzj5-g30XA.png)

At the right popup page, select the Azure function we created in previous step, and click Confirm Selection Button.

![](/images/1*S_RtBnAs5ZWjSD26A66StA.png)

Click Create button to create a new Event Grid Subscription to be event channel between IoT hub and Azure Function.

After Event Grid Subscription creation and deployment complete, you will see its status:

![](/images/1*xsmtMwTjfBJekn5ZZK8DUw.png)

Now let’s ran the local IoT device Simulator code again to send temperature telemetry data to Azure IoT hub, which will trigger Telemetry Event data pass through Azure Event Grid to Azure function. Let’s open Azure Function “EventGridDataEventParser” we built on previous step, click “Monitor” at the left panel, and click Logs tab. In the log window, we can see the IoT device simulator data events had been passed through Azure Event Grid.

![](/images/1*9dDP3ISSB6Yw_WX2c1nULw.png)

Since we only have one line code in Azure Function “EventGridDataEventParser” Run method in run.csx file.

```
log.LogInformation(eventGridEvent.EventType.ToString());
```

We will update the Azure function code to parse event data to retrieve temperature data, device ID, etc.

Click “Code + Test” in Azure Function “EventGridDataEventParser”, select run.csx file from file list.

![](/images/1*x4ZgKnYQvSICow7Tb0lWjg.png)

Copy the below code and paste it to replace the code for run.csx. And click Save button to save the change.

```
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
}
```

The above code is to parse the information in the event data section which is Jason format. Click on the Monitor at the left panel, and select Logs tab again. Now you will see the detail information such as Device ID, Meter ID, Temperature data, event time information have been parsed and logged.

![](/images/1*LsLyPHVp5Kfacb6Ljpz7xg.png)

Let’s recap what we have done in above steps. We built Azure IoT Hub in Azure cloud, developed an IoT device simulator running in local machine to connect and send random temperature data to Azure IoT Hub , created an Azure function, and configure Azure event grid to pass through Azure IoT Hub telemetry data (temperature) to Azure Function. In Azure Function, we parsed event data to retrieve device and temperature information from event data and log them. We archived the below components inside the red rectangle in the architecture diagram.

![](/images/1*exCZAzfzWrxgGsQRQiyyFQ.png)

In next [part 2](/posts/azure-iot-integration-maximo-part2/) blog, we will demonstrate how to integrate Azure IoT to Maximo through Maximo Integration Framework (MIF). We will setup asset and meter data in Maximo, continue update Azure function code to post temperature data to Maximo meter reading through Maximo NextGen Rest API.

You can get full source code from GitHub:

If you think this blog helping you, welcome to buy me a coffee through below Give a Tip button after this blog :)

---
