---
title: "Customized Inbound Message Processing Logic for Maximo Enterprise Service through Automation Script"
date: 2024-07-06
tags: [maximo, automation-script, integration, mif]
draft: false
---


*If you don’t have Medium account, you can read this blog through this *[*friend link*](https://jackyqiubao.medium.com/customized-inbound-message-processing-logic-for-maximo-enterprise-service-through-automation-script-1d07d3f9d425?sk=e828db62e7c8aea1cff6b09453419031)*.*

In my previous blog “[Introduce Maximo Enterprise Service to Accept Inbound Messages from Other Systems](https://medium.com/eamstream/introduce-maximo-enterprise-service-to-accept-inbound-messages-from-other-systems-fcd4a9dce729)”, I introduced how to configure Maximo Enterprise Service for accepting inbound messages. In this blog, we will discuss how to implement customized logic in Maximo Enterprise Service through automation script to meet some special requirement.

### **1. Automation Script VS Java**

Maximo provides multiple options to allow you customize out of box (OOO) Maximo function / feature. For the Maximo Integration Framework (MIF), You can develop your customized integration logic through Java or Automation Script. The major difference between the java and Automation Script options is that Automation Script doesn’t require redeploy EAR file, but Java code need to rebuild EAR and redeploy it to Maximo Server.

In this blog, we will only focus on the option to use automation script to implement customize inbound integration logic for Maximo Enterprise Service.

### **2. Requirement**

Assume we have a System A is to send new / updated asset data to Maximo, asset description will default to concatenate asset description in the payload and current system date. And System A would like to received the response information from Maximo to know if the asset is created or updated in Maximo for the current payload.

### **3. Solution**

Based on above requirement, we can build a synchronized Enterprise Service in Maximo to receive the new / updated asset data from System A, and build automation script for enterprise service request user exist class to implement the default asset description, and build automation script for enterprise response user exist class to provide the action (Create or Update) back to System A.

### **4. Implementation**

**Step 1: Create Object Structure, Enterprise Service and External System**

Follow the steps in Implementation section on [this Enterprise Services introduction blog](https://medium.com/eamstream/introduce-maximo-enterprise-service-to-accept-inbound-messages-from-other-systems-fcd4a9dce729) to create Object Structure, Enterprise Service and External System.

**Step 2: Create Automation Script for Enterprise Services Request User Exist**

Go to System Configuration => Platform Configuration => Automation Scripts. Click on Select Action=> Create=>Script for Integration. Select Enterprise Service option on the popup create script window, and select the Enterprise service “ESASSET” which created on step 1. Select the other options as highlighted on the below screenshot.

![](https://cdn-images-1.medium.com/max/800/1*BudbQm22PCgXImSAIpGyLg.png)

On the Script Language, I selected jython, which is Python in Java, providing the benefit of writing the Python script which can run on the JVM and access to classes written in Java. You can also choose other script language in the following list which you are familiar to:

![](https://cdn-images-1.medium.com/max/800/1*mRRKVM4RrmJA1XPDyWXTBg.png)

At the bottom of this Create Script for Integration page, there is a script box. Type in the following script:

```
#AUTOSCRIPT:SYNC.ESASSET.USEREXIT.IN.AFTER
#DESCRIPTION:Demo script to demostrate Enterprise Service Request customized logic 
from psdi.server import MXServer
#Get current server time string
currentTime=MXServer.getMXServer().getDate().toString();
#This command breaks the input XML so the data can be manipulated. 
#This line should not be necessary, because the framework should do this for you.
#Only add this breakData() function if you have java.lang.NullPointerException when you call getCurrentData function
irData.breakData();
#Get Asset Description field from payload
assetDescription = irData.getCurrentData("DESCRIPTION");
#Update Asset Description field to append current server time
irData.setCurrentData("DESCRIPTION",assetDescription + " - " + currentTime);
```

The comments in the code already explain the code logic. Click “Create” button to create the automation script to build customized logic for Enterprise Service request user exist to append the system time to the asset description field.

**Step 3: Validate the Enterprise Service Request Automation Script**

Open the postman, refer to the “**Step 4: Verify the configuration” **on my previous blog “[Introduce Maximo Enterprise Service to Accept Inbound Messages from Other Systems](https://medium.com/eamstream/introduce-maximo-enterprise-service-to-accept-inbound-messages-from-other-systems-fcd4a9dce729)”, to setup postman, use the following payload to test the automation script:

```
<?xml version="1.0" encoding="UTF-8"?>
<SyncOSASSET xmlns="http://www.ibm.com/maximo" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <OSASSETSet>
    <ASSET action="AddChange">
      <ASSETNUM>DEMOASSET</ASSETNUM>
      <DESCRIPTION>Add asset description Asset for demo</DESCRIPTION>
      <SITEID>AMTRAK</SITEID>      
      <STATUS>ACTIVE</STATUS>      
    </ASSET>
  </OSASSETSet>
</SyncOSASSET>
```

After you execute the payload, you should see the following response from Maximo:

![](https://cdn-images-1.medium.com/max/800/1*-t6uubVMuXjxqwVM-NpxLQ.png)

Open Asset application in Maximo, search for the asset created / updated through the payload, you should be able to see the asset description automatically appended system time stamp to it.

![](https://cdn-images-1.medium.com/max/800/1*Ft34gzngVoxpSGdgEwZTbQ.png)

### **Summary:**

In this blog, we learn how to use automation script to implement a simple customize logic for processing inbound message in Maximo Enterprise Service. Of cause, you can expand this option to implement more complicate logic through automation script in Maximo.

---
*Originally published on [Medium](https://medium.com/@jackyqiubao/customized-inbound-message-processing-logic-for-maximo-enterprise-service-through-automation-script-1d07d3f9d425).*
