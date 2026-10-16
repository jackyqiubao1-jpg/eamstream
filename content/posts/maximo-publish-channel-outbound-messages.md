---
title: "Introduce Maximo Publish Channel for Sending Outbound Messages"
date: 2024-01-16
tags: [maximo, integration, mif]
draft: false
---


Maximo Publish Channel is part of Maximo Integration Frame (MIF) to send outbound message / data from Maximo to external system. This article is to provide the instruction on how to configure Maximo Publish Channel along with other MIF modules to send data from Maximo to other systems.

You will gain the following knowledges on the following modules in MIF after following this instruction:

(a) Object Structure (OS)

(b) Publish Channel (PC)

(c) End Point

(d) External System

#### **1. Requirement:**

Assume that we have the integration requirement to send the asset information from Maximo to other system (for example ServiceNow) when any new asset is created, updated in Maximo. That means Maximo is the system of record for asset data.

**2. Technical Solution:**

To implement the above requirement, we can leverage Maximo NextGen Restful API which I introduced in my this article:

[*Introduction of Maximo NextGen Rest APIs — Part 1 Query Maximo Data*](https://medium.com/@jackyqiubao/introduction-of-maximo-nextgen-rest-apis-part-1-query-maximo-data-6c082087e8f2?sk=a3eb2b199c4fabb83a54c901376e2370)

But the downside of this solution is that you need to build a batch job to query new / updated asset data through Maximo Restful API periodically. And it is not the real time integration. If we need to implement Asset data in time (event trigger) integration with other system, Publish Channel is the better option.

Publish Channels are used in Maximo to send data asynchronously from the system to an external system. They are the outbound integration backbone of Maximo. Publish Channels can be event-enabled to trap events when some MBOs get added, updated or deleted, and publish a message to the outbound JMS queues. The following is the architecture diagram for Maximo Publish Channel.

![](https://cdn-images-1.medium.com/max/800/0*3AOMd5ooecM_SJDn)

*Publish Channel*

**3. Implementation**

The following is the instruction along with explanation to use publish channel to implement above requirement.

**Step 1: Create Object Structure (OS) to organize the outbound data structure**

An object structure is the common data layer that the integration framework components use for outbound and inbound application message processing. An object structure consists of one or more related business objects that define the content of an XML or JSON message (schema).

Open Object Structures application through Integration -> Object Structures menu. Click New Object Structure Button to create a new Object Structure to organize the asset data we plan to send out to other system. Fill out the required information for the OS.

![](https://cdn-images-1.medium.com/max/800/1*XPPq-1KQ4bfwg8chle3axQ.png)

In Source Objects section, click New Row button to add a new Maximo Business Object (MBO), in our case, we will add Asset MBO to this object structure.

![](https://cdn-images-1.medium.com/max/800/1*rgBMK1-Q-i-T_P-DMC9Log.png)

*Add Asset MBO*

Click New Row again in Source Objects section, to add a child MBO ServiceAddress to send address information along with Asset to other system. Make sure select Parent Object to be Asset which we added before, and pick up the relationship “SERVICEADDRESS” for Asset and ServiceAddress MBO. You can understand relationship is how to join Asset and ServiceAddress tables in SQL Query.

![](https://cdn-images-1.medium.com/max/800/1*wcOTrPHCCN7Fg-xY2UZj1g.png)

Save the above OS settings. Click Select Action -> Exclude/Include Fields menu from Object Structures Application to open Exclude/Include Fields window. This setting is to configure what fields we would like to explore to the external system in Asset and ServiceAddress MBOs we added to the OSASSET object structure in previous step. Select ASSET MBO in the Source Object section, check all of the fields in Persistent Fields section except for AssetNum , Status and Description fields, which we plan to send only those fields to the external system.

![](https://cdn-images-1.medium.com/max/800/1*9kv7LgEjbPMe41VF6x1oEQ.png)

Select ServiceAddress MBO and set AddressLine, City, County to be uncheck which we plan to explore asset service address information along with Asset information to other system .

![](https://cdn-images-1.medium.com/max/800/1*slE2A6HeOvHM4eqkW6k34g.png)

Click OK to save the Exclude/Include Fields setting to complete OS configuration.

**Step 2: Create Publish Channel to configure the outbound trigger event and filter condition**

A publish channel is the pipeline for sending data asynchronously from Maximo® Asset Management to an external system. Events that initiate publish channel processing are object events (insert, update, and delete), application-initiated calls,and data export. The content of a publish channel XML message is based on the associated object structure.

Open Publish Channels application through the menu Integration -> Publish Channels. Click New button to create a new Publish Channel. Input the publish channel name, description, and pick up the Object Structure OSASSET we created in previous step. After select OSASSET as object structure for the Publish Channel, the asset and serviceaddress MBOs in the object structure OSASSET is pulled to Object Structure Sub-Records section.

![](https://cdn-images-1.medium.com/max/800/1*wE3MfDtormIdfBsl_7rQHQ.png)

Assume we only need to send Asset information when the asset is created and updated in Maximo, and only need to be sent in curtain Asset status, such as Active, Inactive and Decommissioned. For other Asset status, we will skip the outbound message. To achieve it, we need to turn on the publish channel event listening, and configure processing rule.

Save the publish channel, click “Select Action-> Enable Event Listener” menu, you will found the Enable Listern checkbox is check for the publish channel PCASSET.

![](https://cdn-images-1.medium.com/max/800/1*I3x6gu1LJa5bL9WyyRmtEQ.png)

Click New Row button under the Processing Rules for Sub-Record section, input rule’s name, description, and select SKIP for action, check Primary Object Insert and Update checkboxes, which mean only apply this process rule when insert or update asset record in Maximo.

![](https://cdn-images-1.medium.com/max/800/1*oCoZCvSESmItykHWp53U2Q.png)

Click Add/Modify Conditions button, click New Row button under conditions section to add a new condition, and then click New Row in XML Field Tab to create a new XML Field Evaluation. Select Status for the field value, evaluation type as NOTEQUALS, Select VALUE option and ACTIVE as the value. This evaluation means if ASSET STATUS != ACTIVE, then Skip sending outbound message.

![](https://cdn-images-1.medium.com/max/800/1*v6lMLDPaG2SewKlpliSMzg.png)

We need to configure the same evaluation for other 2 asset status: INACTIVE and DISCOMMISSIONED.

Click New Row in XML Field Tab to create a new XML Field Evaluation for Asset status is not equal to INACTIVE, and click New Row in XML Field Tab again to create a new XML Field Evaluation for Asset status is not equal to DISCOMMISSIONED. The final result looks like following:

![](https://cdn-images-1.medium.com/max/800/1*H1eniirb72Am-xi4cbVhmg.png)

Click OK button to save the setting.

Click menu “Select Action -> Message Tracking” to open Publish Channel Message Tracking Setting, check Enable Message Tracking and Store Message checkboxes, which will allow us to view the outbound message for this PC in Message Tracking application.

![](https://cdn-images-1.medium.com/max/800/1*yUN-RFzXmfbwCV2sHLWdjg.png)

**Step 3: Create an ENDPOINT for the url which to receive outbound message payload from Maximo.**

An endpoint in IBM Maximo Asset Management is a location to which an outbound queue sends data. It specifies how outbound transactions are delivered and the handler that routes them. You can create an endpoint to identify a target location and the transport mechanism that the integration framework, or deployment manager uses to publish data, or to invoke a service.

Click Integration-> End Points to open End Points application. Input end point and description, select HTTP as handler. Input url information (The url in the below screenshot is just for demo purpose).

![](https://cdn-images-1.medium.com/max/800/1*8dnLoEp-iDkZEUGT1_D3Xw.png)

Save the end point setting by click Save button.

**Step 4: Create an External System to send Asset data to external application.**

An external system in Maximo is any business application that sends data to Maximo or receives data from Maximo Asset Management. You can use and configure enterprise services and publish channels to exchange data with one or more external systems. The Maximo system sends messages to an external system or application. You create an external system to exchange with external applications.

Open the External Systems Application through Integration -> External Systems. Input system and description. Check Enable checkbox. Set Outbound Sequential Queue with the out of box outbound queue: jms/maximo/int/queues/sqout. Of couse you can configure your own outbound sequential queue, which is not in this instruction scope.

![](https://cdn-images-1.medium.com/max/800/1*zpf20Jh_sEyOIvzRn0QqPA.png)

Click Publish Channels Tab, Click New Row button to add the publish channel PCAsset we created on Step2 to this external system. Check Enable checkbox, and select End Point EPSERVICENOW which we created on step3.

![](https://cdn-images-1.medium.com/max/800/1*wIg-rvctRgjJ_qFsu57QRg.png)

Click Save button to save the configuration.

**Step 5: Verify the configuration.**

So far we complete all the required configurations. Now we can start to verify our settings. Open Asset application to create a new asset.

![](https://cdn-images-1.medium.com/max/800/1*HWwFqae2JKuKgUwqzxDm6g.png)

Click Service Address tab to select a service address record for the asset

![](https://cdn-images-1.medium.com/max/800/1*OzMFHOUqn2w0QeNjizKL5w.png)

Change Asset Status through change status tool, and select Active status. Save the change, and Asset current status is Active now. Per our configuration, the Publish Channel PCAsset we configured before should be triggered and send outbound message through the Outbound Sequential Queue. Since we enable the message tracking for the Publish Channel PCAsset on Step2, we can use message tracking application to view the outbound message.

Open Message Tracking application through Integration -> Message Tracking, we would find a message is triggered by PCAsset in the list.

![](https://cdn-images-1.medium.com/max/800/1*r316c3vfLemhc5VvvAZUOw.png)

Click the pencil icon at the right end of the message, we can view the XML payload to be sent to external application.

```
<?xml version="1.0" encoding="UTF-8"?>
<PublishOSASSET xmlns="http://www.ibm.com/maximo" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" creationDateTime="2024-01-21T16:02:24-05:00" transLanguage="EN" baseLanguage="EN" messageID="4761787.1705870944896610290" maximoVersion="7 6 20220823-0909 V7613-344" event="1">
  <OSASSETSet>
    <ASSET action="Replace">
      <ASSETNUM>DEMOASSET</ASSETNUM>
      <DESCRIPTION>Asset for demo</DESCRIPTION>
      <SITEID>DEMOSITE</SITEID>
      <STATUS changed="1" maxvalue="OPERATING">ACTIVE</STATUS>
      <SERVICEADDRESS>
        <ADDRESSCODE>DEMOADDR</ADDRESSCODE>
        <CITY>ATLANTA</CITY>
        <COUNTRY>US</COUNTRY>
        <COUNTY />
        <DESCRIPTION>Demo Address</DESCRIPTION>
        <ORGID>DEMOORG</ORGID>
        <POSTALCODE>30038</POSTALCODE>
        <STREETADDRESS>1000 Demo Street</STREETADDRESS>
      </SERVICEADDRESS>
    </ASSET>
  </OSASSETSet>
</PublishOSASSET>
```

(6) JSON format?

The default outbound message is XML format. Does Maximo Publish Channel support JSON format payload? The answer is yes, and it is pretty easy to implement but checking one configuration on Publish Channel. What you need to do is just need to open the Publish Channel record and check “Publish JSON” checkbox.

![](https://cdn-images-1.medium.com/max/800/1*9cJGroRxf3nKytNUsrByRw.png)

**4. Summary**

Let’s recap what we have learned in previous section: We created an object structure to organize the data structure, and then create a publish channel to define the conditions to trigger the outbound message to send object structure data to external application. Configure external system to setup sequential outbound queue and end point to send outbound message to target system.

---
*Originally published on [Medium](https://medium.com/@jackyqiubao/introduce-maximo-publish-channel-for-sending-outbound-messages-11b4bfdbe971).*
