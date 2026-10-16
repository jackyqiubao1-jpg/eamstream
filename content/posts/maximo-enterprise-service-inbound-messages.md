---
title: "Introduce Maximo Enterprise Service to Accept Inbound Messages from Other Systems"
date: 2024-05-07
tags: [maximo, integration, mif]
draft: false
---


*If you don’t have Medium member, you can try this *[*friend link*](https://medium.com/eamstream/introduce-maximo-enterprise-service-to-accept-inbound-messages-from-other-systems-fcd4a9dce729?sk=c1e6f1ebfd965a48856ef130d1ff19e0)* to read.*

In my previous blog [*Maximo Publish Channel for Sending Outbound Messages*](https://medium.com/eamstream/introduce-maximo-publish-channel-for-sending-outbound-messages-11b4bfdbe971) introduces outbound message from Maximo. How Maximo can accept inbound message from other system? There are 2 ways Maximo can receive outbound message from outside system to update the data in Maximo: (1) Through Object Structure Restful API (You can refer to my other blog “[*Introduction of Maximo NextGen Rest APIs — Part 2 Update Maximo Data*](https://medium.com/eamstream/introduction-of-maximo-nextgen-rest-apis-part-2-update-maximo-data-e3db83751709?source=your_stories_page-------------------------------------)*”*, or (2) Through Enterprise Service, which will provide step by step instruction in this blog.

Maximo Enterprise is part of Maximo Integration Frame (MIF) to receive outbound message / data from external system. This article is to provide the instruction on how to configure Maximo Enterprise Service along with other MIF modules to receive the data from other systems.

You will gain the following knowledges on the following modules in MIF after following this instruction, which was introduced on my other Publish Channel intruction blog too:

(a) Object Structure (OS)

(b) Enterprise Service (ES)

(c) External System

#### 1. Requirement:

Assume that we have the integration requirement to reeive the asset information from other system (for example SAP). When any new asset is created, updated in external system, like SAP, it will call Maximo interface to create or update asset data in Maximo. That means SAP is the system of record for asset data, but need sync up asset data to Maximo for asset maintenance process, like work order management.

**2. Technical Solution:**

To implement the above requirement, we can leverage Maximo NextGen Restful API which I introduced in my this article to update asset data in Maximo:

[*Introduction of Maximo NextGen Rest APIs — Part 2 Update Maximo Data*](https://medium.com/eamstream/introduction-of-maximo-nextgen-rest-apis-part-2-update-maximo-data-e3db83751709?source=your_stories_page-------------------------------------)

But there is a limitation with NextGen Restful API solution to update / delete data in Maximo. You need to query the record to obtain resource ID, and then use resource ID to form the URI to update the record. It required to access Maximo twice to update one record. With the Enterprise Service, you just need to post once, and you can customize your interface logic through customized Enterprise Service User Exit class. This blog will not cover customized interface process logic through User Exit class, might have other blog in future to cover this topic.

An enterprise service is a pipeline for querying system data and importing data into Maximo IT from an external system. Data processing layers that transform data and apply business processing rules to data before it reaches the system objects. These processing layers include:
(a) Processing rules — The integration framework provides a rule engine where you can filter and transform the XML message.
(b) User exit — The user exit specifies a Java™ class that you can use to filter, transform data, and implement business logic. You can use this class as part of an installation-customization.
(c) Data processing class — The processing class specifies a Java class that you can use to filter, transform data, and implement business logic.
(d) XSL map — The XSL map specifies an XSLT style sheet that you can use to transform data and map the XML data message to another format.

![](https://cdn-images-1.medium.com/max/800/1*ML8Z-mSUdYM2jYwbvI2GKQ.png)

*Enterpise Service*

**3. Implementation**

The following is the step by step instruction along with explanation to use publish channel to implement above requirement.

**Step 1: Create Object Structure (OS) to organize the inbound data structure**

An object structure is the common data layer that the integration framework components use for outbound and inbound application message processing. An object structure consists of one or more related business objects that define the content of an XML or JSON message (schema).

Open Object Structures application through Integration -> Object Structures menu. Click New Object Structure Button to create a new Object Structure to organize the asset data we plan to send out to other system. Fill out the required information for the OS.

![](https://cdn-images-1.medium.com/max/800/0*Q6H_mvgBenutyB86.png)

In Source Objects section, click New Row button to add a new Maximo Business Object (MBO), in our case, we will add Asset MBO to this object structure.

![](https://cdn-images-1.medium.com/max/800/0*qjlbUB2WwB62ceXb.png)

Add Asset MBO. Click New Row again in Source Objects section, to add a child MBO ServiceAddress to send address information along with Asset to other system. Make sure select Parent Object to be Asset which we added before, and pick up the relationship “SERVICEADDRESS” for Asset and ServiceAddress MBO. You can understand relationship is how to join Asset and ServiceAddress tables in SQL Query.

![](https://cdn-images-1.medium.com/max/800/0*YZpnGRTc9N6awcZV.png)

Save the above OS settings. Click Select Action -> Exclude/Include Fields menu from Object Structures Application to open Exclude/Include Fields window. This setting is to configure what fields we would like to explore to the external system in Asset and ServiceAddress MBOs we added to the OSASSET object structure in previous step. Select ASSET MBO in the Source Object section, check all of the fields in Persistent Fields section except for AssetNum , Status and Description fields, which we plan to send only those fields to the external system.

![](https://cdn-images-1.medium.com/max/800/0*-lTvnynzdKm2iTj4.png)

Select ServiceAddress MBO and set AddressLine, City, County to be uncheck which we plan to explore asset service address information along with Asset information to other system .

![](https://cdn-images-1.medium.com/max/800/0*9qrY336-IWQqg9z9.png)

Click OK to save the Exclude/Include Fields setting to complete OS configuration.

**Step 2: Create Enterprise Service**

Open Enterprise Services application through the menu Integration -> Enterprise Services. Click New button to create a new Enterprise Service. Input the Enterprise Service name, description, and pick up the Object Structure OSASSET we created in previous step. After select OSASSET as object structure for the Publish Channel, the asset and serviceaddress MBOs in the object structure OSASSET is pulled to Object Structure Sub-Records section. Click Save button to save the configuration.

![](https://cdn-images-1.medium.com/max/800/1*h_qSoJ4yNI4MbY6Ig-q-xQ.png)

Click menu “Select Action -> Message Tracking” to open Publish Channel Message Tracking Setting, check Enable Message Tracking and Store Message checkboxes, which will allow us to view the outbound message for this PC in Message Tracking application.

![](https://cdn-images-1.medium.com/max/800/1*q-dnPGIbTvL1Cjdh-TtCdw.png)

You can configure process rule at Object Structure Business Processing Rules section. For example, if you want to skip to process the asset with status is decommissioned, you can click “New Row” button to add a skip rule as following:

![](https://cdn-images-1.medium.com/max/800/1*cUpBPxcu9cY0DHiIGhP1DQ.png)

And click “Add/Modify Conditions” button to define asset =DECOMMISSIONED condition as below:

![](https://cdn-images-1.medium.com/max/800/1*U5Ycj7k6hfpCm4LGPAx6qg.png)

Click OK button to save the configuration.

**Step3: Create an External System to receive Asset data from external application.**

An external system in Maximo is any business application that sends data to Maximo or receives data from Maximo Asset Management. You can use and configure enterprise services and publish channels to exchange data with one or more external systems. The Maximo system sends messages to an external system or application. You create an external system to exchange with external applications.

Open the External Systems Application through Integration -> External Systems. Input system name and description. Check Enable checkbox. For Enterprise Service, there are 2 types of inbound queue: Sequential and Continuous. If your inbound messages need to be processed in the order based on the arrive time sequence, for example, the external system sends work order process data to Maximo with different status based on the defined work flow process, Maximo should process the work order status with the messages arrive time sequence, then you should choose sequential queue for the enterprise. If the inbound messages to be processed doesn’t care about the order, you can choose continual inbound queue. Another big difference between Sequential and Continuous queue is that when there is error happen, the sequential queue will be blocked (cannot process the following messages until the error message is cleaned up), but continual queue can continue processing the following messages.

For our case, the Asset inbound messages can be processed without sequence, we can set Inbound Continuous Queue with the out of box inbound queue: jms/maximo/int/queues/cqin. Of course you can configure your own inbound sequential / continuous queue, which is not in this instruction scope.

![](https://cdn-images-1.medium.com/max/800/1*fmFVo1jkHuoCvCiTb4AikQ.png)

Click on Enterprise Services tab, select the enterprise service ESASSET we created at **Step 2**. And check Enable checkbox to enable the enterprise service. And check “Use Continuous Queue” to configure this enterprise service will use continuous queue. If you don’t check this checkbox, it will default to use sequential queue.

![](https://cdn-images-1.medium.com/max/800/1*ExvmIa3mCQDzLlwfPAkcTw.png)

**Step 4: Verify the configuration.**

Now we are ready to test enterprise service to receive the payload from external system. We will use postman to simulate SAP to send payload to external.

Open postman, create a new request, input the following url on the new request url box, select request type to be POST. In the below url, the first “ESASSET” is the external system name we created on step 3, and the last “ESASSET” is the enterprise service name we created on step 2. (I should use better naming convention for it.)

[https://{yourmaximoserver}.com/meaweb/esqueue/ESASSET/ESASSET](https://amtrak-test.maximo.com/meaweb/esqueue/ESASSET/ESASSET)

On the postman Authorization tab, select your Maximo integration authorization type. For this case, we use basic auth to input username and password.

![](https://cdn-images-1.medium.com/max/800/1*G0veb1OXWkL_XsfKG9udcA.png)

You can get more information regarding Maximo integration authorization at the **Authentication **section on my other blog: [*Introduction of Maximo NextGen Rest APIs — Part 1 Query Maximo Data*](https://medium.com/eamstream/introduction-of-maximo-nextgen-rest-apis-part-1-query-maximo-data-6c082087e8f2)

Click the Body tab on postman request, select raw option, and pickup XML as format from the dropdown list.

![](https://cdn-images-1.medium.com/max/800/1*HkJFDjwAt36ze2uPshQ2bQ.png)

Input the following XML payload on the body textbox.

```
<?xml version="1.0" encoding="UTF-8"?>
<SyncOSASSET xmlns="http://www.ibm.com/maximo" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <OSASSETSet>
    <ASSET action="AddChange">
      <ASSETNUM>DEMOASSET</ASSETNUM>
      <DESCRIPTION>Add asset description Asset for demo</DESCRIPTION>
      <SITEID>DEMOSITE</SITEID>      
      <STATUS>ACTIVE</STATUS>
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
</SyncOSASSET>
```

The above XML payload is to create / update an Asset with Assetnum= DEMOASSET. You might notice there is an action property at the Asset node in this payload. The following action value you can leverage in your payload for the differernt scenario.

![](https://cdn-images-1.medium.com/max/800/1*GaqdTBf90NcjcyeOWxrOYA.png)

Click Send button on the request page, if everything is good, you will receive 200 response code from Maximo.

![](https://cdn-images-1.medium.com/max/800/1*em8sVzaZywN5NMMoIeqZ6g.png)

To check what have been sent to Maximo, open Message Tracking application in Maximo, filter Integration Component with Enterprise name “ESASSET”, we would see the following payload have been processed

![](https://cdn-images-1.medium.com/max/800/1*tel2QqgrHOZz_JkNOIk75A.png)

And open the Asset application in Maximo, look for the DEMOASSET record, we can find it is created in Maximo.

![](https://cdn-images-1.medium.com/max/800/1*BpbPc6kO2sqDv0DMa1Y4-A.png)

**4. Summary**

Let’s recap what we learn on this blog: We create an Object Structure to organize the data structure to be processed, and then create an Enterprise Service to define the process rules for this interface, and use External Service to define what type of the message queue to be used to process this interface. Last we learn how to use postman to simulate the integration testing.

Hope you enjoy this blog, and feel free to leave your comments if you have any question / comment on it.

---
*Originally published on [Medium](https://medium.com/@jackyqiubao/introduce-maximo-enterprise-service-to-accept-inbound-messages-from-other-systems-fcd4a9dce729).*
