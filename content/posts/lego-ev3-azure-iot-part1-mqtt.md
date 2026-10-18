---
title: "Turn Your Lego EV3 Robot to Azure IoT Device — Part 1 Connect to IoT Hub through MQTT"
date: 2023-03-23
tags: [iot, azure, mqtt, maximo]
draft: false
---


In my previous serial blogs “[Azure IoT integration with Maximo](https://jackyqiubao.medium.com/azure-iot-integration-with-maximo-part-1-b141c4210db0?sk=5345c390bd704aff8349231bbef7ec7d)”, introducing how to build a python IoT device simulator sending IoT device sensor data (for example, temperature data) to Azure IoT hub, through Azure Event Grid to pass telemetry to Azure Function. In Azure Function, call Maximo NexGen Rest API to post telemetry data to Maximo Asset meter reading.

In this serial blogs, I will introduce how to turn your Lego EV3 robot to be IoT device to replace the IoT simulator in [previous blog](https://jackyqiubao.medium.com/azure-iot-integration-with-maximo-part-1-b141c4210db0?source=friends_link&sk=5345c390bd704aff8349231bbef7ec7d), and how to connect EV3 through MQTT instead of Azure IoT Device SDK. In the [part 2 blog](https://jackyqiubao.medium.com/turn-your-lego-ev3-robot-to-azure-iot-device-part-2-create-work-order-in-maximo-6f7427ffcbaf?sk=9489b17541c2d307327a8201a15af53f), I will demonstrate how to create Work Order in Maximo through Maximo NextGen Rest API based on the color sensor data from Lego EV3 Robot.

The following is the high level architect diagram for this experiment.

![](https://cdn-images-1.medium.com/max/800/1*1xEPxOu9Q0baaB5lten4lw.png)

To be able to perform this experiment, you need to have:

(1) [Lego EV3 robot](https://www.amazon.com/LEGO-MINDSTORMS-Building-Interactive-Redesigned/dp/B016SF8GWY/ref=sr_1_4?crid=6Z8YBP75DDRQ&keywords=lego+ev3&qid=1676925845&sprefix=lego+ev3%2Caps%2C87&sr=8-4)

LEGO Mindstorms EV3 is the third generation robotics kit in LEGO’s Mindstorms line. You can build a lot of interesting projects with Lego EV3 essential kit or extension set. You can program on its original built-in OS with Scratch language, or program it on external Debian OS launched through SD card with MicroPython / Pythhon. In this experiment, I will demonstrate how to use the second option to develop customized code to send color sensor data from Lego EV3 robot to Azure IoT hub through MQTT with MicroPython.

![](https://cdn-images-1.medium.com/max/800/0*TIexihCLbT0u3aLH)

*Lego Ev3 Robot*

(2) A microSD or microSDHC card (2GB or larger). microSDXC is not supported on the EV3. All cards larger than 32GB will not work with the EV3.

The reason we need miscroSD / miscroSDHC card is to run ev3dev stretch (Debian OS) in this SD card without messing up original OS of EV3, and also will allow us to run customized MicroPython code in the ev3dev stretch (Debian OS).

(3) An Azure account. You can register an Azure account with $200 credit through this link:[https://azure.microsoft.com/en-us/free/](https://azure.microsoft.com/en-us/free/)

(4) Maximo Environment. If you would like to try the whole experiment to develop Azure function to call Maximo NextGen Rest API to create work order in Maximo when EV3 robot color sensor detect red color, you also need to have an Maximo environment to complete the whole experiment. But if you don’t have Maximo environment, or you are not interested in EAM section, follow the sections from 1 to 4 in this blog still give your the idea on how to connect your real IoT device to Azure IoT hub through [MQTT](https://en.wikipedia.org/wiki/MQTT) with [MicroPython](https://en.wikipedia.org/wiki/MicroPython).

In this blog, you will have hand-on experience to prepare your EV3 to be launched through ev3dev stretch Debian OS in SD card instead of EV3 built-in OS. And also learn how to connect your EV3 to internet. You will learn how to use MQTT client in MicroPython to connect to Azure IoT hub instead of using Azure IoT Device SDK. The purpose of this experiment is to develop customized MicroPython code to turn your EV3 to be Azure IoT device to send telemetry data (for example EV3 sensor data) to Azure IoT Hub, and in Azure IoT Function, we will update the Azure function which we built in my previous blog “[Azure IoT integration with Maximo — Part 1](https://jackyqiubao.medium.com/azure-iot-integration-with-maximo-part-1-b141c4210db0?sk=5345c390bd704aff8349231bbef7ec7d)“ to call Maximo NextGen Rest API to create work order when the EV3 color sensor data value is “Red”.

**1. Launch your EV3 through ev3dev stretch Debian OS in SD card**

You can follow the steps on ev3dev.org to write the ev3dev stretch (Debian OS) image to the SD card.

[https://pybricks.com/ev3-micropython/startinstall.html](https://pybricks.com/ev3-micropython/startinstall.html)

After burn ev3dev stretch image to SD card, insert SD card to your EV3 SD card slot, turn on your EV3, it will launch ev3dev stretch (Debian OS) from SD card instead of the build-in EV3 OS.

**2. Connect your EV3 to internet**

Once your EV3 complete start, follow the steps in below link to choose the internet connection option which is suitable for you to connect your EV3 to internet.

[https://www.ev3dev.org/docs/networking/](https://www.ev3dev.org/docs/networking/)

For Bluetooth option, you can connect your EV3 to internet with win10 which is the option I am using. You just need to turn on your mobile hotspot with bluetooth on your windows 10, and connect your EV3 Bluetoothto this Bluetooth mobile hotspot.

![](https://cdn-images-1.medium.com/max/800/1*bhkNIRZtTxLeSI87MR5SYw.png)

**3. Setup Development Environment**

We will develop the MicroPython programs using Visual Studio Code. Follow the steps below to download, install, and configure this application:

(1) Download [Visual Studio Code](https://code.visualstudio.com/Download).

(2) Follow the on-screen instructions to install the application.

(3) Launch Visual Studio Code.

(4) Open the extensions tab.

(5) Install the EV3 MicroPython extension as shown below

![](https://cdn-images-1.medium.com/max/800/0*s6GbZHMVhEIrG846.png)

*Installing the extension from the Visual Studio Code marketplace*

**4. Start to code**

The following section we will start to develop MicroPython code to connect EV3 to IoT hub through [MQTT](https://en.wikipedia.org/wiki/MQTT). You might ask why we don’t use Azure IoT Device SDK in this experiment like what we did in my previous blog “[Azure IoT integration with Maximo — Part 1](https://jackyqiubao.medium.com/azure-iot-integration-with-maximo-part-1-b141c4210db0?source=friends_link&sk=5345c390bd704aff8349231bbef7ec7d)”. The major reason is Azure IoT Device SDK is required at least Python 3.6 version, but in current Ev3Dev Stretch Image (Debian Stretch OS), it is only support Python3.5.3.

And let’s talking about a little bit about MicroPython. MicroPython is a software implementation of a programming language largely compatible with Python 3, written in C, that is optimized to run on a microcontroller. MicroPython consists of a Python compiler to bytecode and a runtime interpreter of that bytecode. If you are familiar with Python, MicroPython looks almost the same from development perspective.

Now let’s getting start to have fun on coding.

(1) Create a new project.

To create a new project, open Visual Studio Code, and then open the EV3 MicroPython tab and click *create a new project*, as shown in below figure. Enter a project name in the text field that appears and press *Enter*. When prompted, choose a location for this program and confirm by clicking *choose folder*.

![](https://cdn-images-1.medium.com/max/800/0*XIQyRf8N3jam3G8m.png)

*Create a new project*

When you create a new project, it already includes a file called *main.py*. To see its contents and to modify it, open it from the file browser as shown in below figure. This is where you’ll write your programs.

![](https://cdn-images-1.medium.com/max/800/0*e_F623zqfQtCksnf.png)

*Default main.py program.*

The following is the *project folders structure*, as shown in below figure. A project folder is a directory on your computer that contains the main program (**main.py**) and other optional scripts or files. This project folder and all of its contents will be copied to the EV3 Brick, where the main program will be run.

![](https://cdn-images-1.medium.com/max/800/0*muuHG-9rMY1-YhwO.png)

*Folder structure*

Open main.py in VS Code, and update the code to be the following:

```
#!/usr/bin/env pybricks-micropython
from pybricks.hubs import EV3Brick
from pybricks.ev3devices import ( ColorSensor)
from pybricks.parameters import Port, Color
import time
from util import create_mqtt_client, get_telemetry_topic, get_c2d_topic, parse_connection
import json
from json import dumps

# This program requires LEGO EV3 MicroPython v2.0 or higher.
# Click "Open user guide" on the EV3 extension tab for more information.
# Write your program here.
HOST_NAME = "HostName"
SHARED_ACCESS_KEY_NAME = "SharedAccessKeyName"
SHARED_ACCESS_KEY = "SharedAccessKey"
SHARED_ACCESS_SIGNATURE = "SharedAccessSignature"
DEVICE_ID = "DeviceId"
## Parse the connection string into constituent parts
dict_keys = parse_connection("<YOUR CONNECTION STRING>")
shared_access_key = dict_keys.get(SHARED_ACCESS_KEY)
shared_access_key_name =  dict_keys.get(SHARED_ACCESS_KEY_NAME)
hostname = dict_keys.get(HOST_NAME)
device_id = dict_keys.get(DEVICE_ID)

## Create you own shared access signature from the connection string that you have
## You can use Azure IOT HUB VS Code extenstion to generate SAS token, but it has time limit. 
## Need to implement token auto generate service for real use case
sas_token_str = "<YOUR SAS TOKEN STRING>"

## Create username following the below format '<HOSTNAME>/<DEVICE_ID>'
username = hostname + '/' + device_id 

## Create UMQTT ROBUST or UMQTT SIMPLE CLIENT
mqtt_client = create_mqtt_client(client_id=device_id, hostname=hostname, username=username, password=sas_token_str, port=8883, keepalive=120, ssl=True)

print("connecting")
mqtt_client.reconnect()

def callback_handler(topic, message_receive):
    print("Received message")
    print(message_receive)

subscribe_topic = get_c2d_topic(device_id)
mqtt_client.set_callback(callback_handler)
mqtt_client.subscribe(topic=subscribe_topic)

## Send telemetry from EV3 color sensor
ev3 = EV3Brick()
topic = get_telemetry_topic(device_id)
## Send a C2D message and wait for it to arrive at the device
color_sensor = ColorSensor(Port.S1)
while True:
    curColor =color_sensor.color()
    colorStr=str(curColor).split(".")[1]
    print("Current color:" +colorStr)
    color_msg = {"telemetry": colorStr,"metername":"COLOR_SEN2"}
    print("Sending message: " + str(color_msg))    
    mqtt_client.publish(topic=topic,  msg=str(color_msg))
    time.sleep(10)
```

In the above code, update the following line with your Azure IoT Device Connection String.

```
dict_keys = parse_connection("<YOUR CONNECTION STRING>")
```

To gain your Azure IoT Device Connection String, Login your Azure portal, open your IOT Hub (you can refer to my previous blog “[Azure IoT integration with Maximo — Part 1](https://jackyqiubao.medium.com/azure-iot-integration-with-maximo-part-1-b141c4210db0?sk=5345c390bd704aff8349231bbef7ec7d)” on how to create Azure IoT hub) , under Device management section, click “Devices” to open device list. Create a new device call “EV3”.

![](https://cdn-images-1.medium.com/max/800/1*I2-2L9fo-TY9SdV8ZnFCHg.png)

Click the device “EV3” in the device list to open device detail information. At the end of Primary connection string field, click copy button to copy device connection string.

![](https://cdn-images-1.medium.com/max/800/1*vpryNFmwSlPgf7iPFPVvAw.png)

Now let’s switch back to VS Code to update connection string in the IoT.

And we also need to update the SAS Token String in the following code

```
sas_token_str = "<YOUR SAS TOKEN STRING>"
```

To retrieve your SAS token, go back to your Azure IoT hub, select “Shared access policies” at the left panel, and click device policy, and then click Copy button to copy primary Connection string.

![](https://cdn-images-1.medium.com/max/800/1*7KiZteE84bq8ZxqR61sktg.png)

Follow the below step in VS Code to generate a temperary SAS token with the device share access policy connection string.

(1) Install Azure IoT Hub Extension in VS Code.

(2) Expand the AZURE IOT HUB DEVICES tab in the bottom left corner of Visual Studio Code.

(3) Right-click your device and select Generate SAS Token for Device.

(4) Set expiration time and press ‘Enter’.

(5) The SAS token is created and copied to clipboard. The SAS token that’s generated has the following structure:

`HostName={iotHub-hostname};DeviceId=EV3;SharedAccessSignature=SharedAccessSignature sr={iotHub-hostname}%2Fdevices%2FMyDevice01%2Fapi-version%3D2016-11-14&sig=vSgHBMUG.....Ntg%3d&se=1456481802`

The part of this token to use as the Password (SAS token) field to connect using MQTT is: `SharedAccessSignature sr={iotHub-hostname}%2Fdevices%2FMyDevice01%2Fapi-version%3D2016-11-14&sig=vSgHBMUG.....Ntg%3d&se=1456481802`

Paste the above SAS token to replace the following code section of <YOUR SAS TOKEN STRING>

```
sas_token_str = "<YOUR SAS TOKEN STRING>"
```

*Notes: Above approach to generate a temporary SAS token is not the right way to create SAS token for actual production environment, because the SAS token will be expired for curtain period, it doesn’t make sense to update SAS token in production manually every coupe dates. To automatically retrieve SAS token, you might need to build a SAS token service in Azure, and call this SAS token service from your device to retrieve a new SAS token once it is expired. To implement SAS token service, you can refer to *[*this instruction*](https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-dev-guide-sas?tabs=python)* to build a customized SAS token service in Azure.*

In VS Code, click “File -> New File…”, typed util.py to create a new python file and enter.

![](https://cdn-images-1.medium.com/max/800/1*m33T06j_gslzBW8WcjjOog.png)

In util.py file, copy and paste the below code to it.

```
from umqtt.robust import MQTTClient

def create_mqtt_client(client_id, hostname, username, password, port=8883, keepalive=120, ssl=True):
    if not keepalive:
        keepalive = 120
    if not ssl:
        ssl = True
    c = MQTTClient(client_id=client_id, server=hostname, port=8883, user=username, password=password, keepalive=120, ssl=True)
    c.DEBUG = True
    return c

def get_telemetry_topic(device_id):
    return get_topic_base(device_id) + "/messages/events/$.ct=application%2Fjson%3Bcharset%3Dutf-8"
    
def get_c2d_topic(device_id):
     return get_topic_base(device_id) + "/messages/devicebound/#"

def get_topic_base(device_id, module_id=None):
    if module_id:
        base_str = "devices/" + device_id + "/modules/" + module_id
    else:
        base_str = "devices/" + device_id
    return base_str

DELIMITER = ";"
VALUE_SEPARATOR = "="

def parse_connection(connection_string):
    cs_args = connection_string.split(DELIMITER)
    dictionary = dict(arg.split(VALUE_SEPARATOR, 1) for arg in cs_args)
    return dictionary
```

After save the above code to util.py, you can click the download button at EV3Dev Device Browser extension to download the code to your EV3 robot device.

![](https://cdn-images-1.medium.com/max/800/1*2SJwjVuQK-A-dIT5zp5z0A.png)

Expand the folder to locate main.py file, right click on it, select “Run” to start running the code from EV3.

![](https://cdn-images-1.medium.com/max/800/1*E7LnhHhH9njXDTdEwUaxyg.png)

On the VS Code output window will be able to see the log as:

![](https://cdn-images-1.medium.com/max/800/0*LyVOzYXe4-w_Y346.png)

If you open your IoT Hub, on overview page, you will be able to see the messages were received by the IoT hub.

![](https://cdn-images-1.medium.com/max/800/1*hWLFrVMNS4r034yJ8w3DYg.png)

In this blog, we learned how to turn your EV3 robot to be Azure IoT device to connect to Azure IoT hub, and sent color sensor data to Azure IoT hub through MQTT.

In [next blog](https://jackyqiubao.medium.com/turn-your-lego-ev3-robot-to-azure-iot-device-part-2-create-work-order-in-maximo-6f7427ffcbaf?sk=9489b17541c2d307327a8201a15af53f), we will learn how to parse the telemetry color data from EV3 Robot device, and create work order in Maximo through NextGen Rest API.

If you think this blog helping you, welcome to buy me a coffee through below Green *Give a Tip* button :)

---
