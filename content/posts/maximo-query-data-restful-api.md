---
title: "How to query data in Maximo through Restful API"
date: 2020-12-06
tags: [maximo, rest-api, integration]
draft: false
---


For Maximo integration with external system, there are two directions from Maximo’s perspective: Inbound and outbound interfaces. There is a lot of business scenarios for the requirement of query data from Maximo, which we consider it as outbound interface for Maximo. For example, external system ServiceNow might need to query service request data in Maximo to pull out the service request current status.

There are couple ways to query data from Maximo. Maximo support Web Service and Restful. The return result can be xml or json format. In this article, will show you how to leverage Maximo Restful API to query data in Maximo.

The REST API exposes business objects and integration object structures as REST resources. The REST API can provide resource data in either XML or JavaScript Object Notation (JSON) format. External applications can use the REST API to query and update application data. We will focus on the query in this article, will have other blog to explain on the update.

The following diagram provides an overview of how the REST API handles requests.

![](https://cdn-images-1.medium.com/max/800/0*6VRf0LPFc9cZ_1zE.gif)

As you can see from the above diagram, to allow external system to query data from Maximo, you need to create your Object Structure (OS) to expose the business data you would like to allow external system to call, or external system can access to Maximo Business Object (MBO) data through Restful API if there is not complex data requirement. When an external consumer initiates a request, a REST API controller directs the resource request to the appropriate resource handler. The resource handler interacts with resources to execute the request. When the request is complete, the resulting resource or resource collection is processed by the resource serializer. The resource serializer returns the representation as the resource response. Serializers are provided for XML and JSON.

Now let’s take a look how to query Maximo business object (MBO) and Object Structure (OS) through the examples.

The query will use GET action, the follow is an example to query locations MBO for the location of PT100, and the return format is Json

```
https://yourmaximoserver/maxrest/rest/mbo/locations?location=PT100&_format=json
```

The following will be return:

*{*

*“LOCATIONSMboSet”: {*

*“rsStart”: 0,*

*“rsCount”: 1,*

*“LOCATIONS”: [*

*{*

*“rowstamp”: “4874304”,*

*“Attributes”: {*

*“LOCATION”: {*

*“content”: “PT100”*

*},*

*“DESCRIPTION”: {*

*“content”: “Demo data”*

*},*

*“TYPE”: {*

*“content”: “FPOS”*

*},*

*“GLACCOUNT”: {*

*“content”: “0000–000000–0000–000000000000000000000000”*

*},*

*“CHANGEBY”: {*

*“content”: “MAXADMIN”*

*},*

*“CHANGEDATE”: {*

*“content”: “2020–12–06T21:24:42+00:00”*

*},*

*“DISABLED”: {*

*“content”: ****false***

*},*

*“SITEID”: {*

*“content”: “DEMOSITE”*

*},*

*“ORGID”: {*

*“content”: “DEMOORG”*

*},*

*“ISDEFAULT”: {*

*“content”: ****false***

*},*

*“CHILDREN”: {*

*“content”: ****false***

*},*

*“STATUS”: {*

*“content”: “OPERATING”*

*},*

*“LOCATIONSID”: {*

*“content”: 1351,*

*“resourceid”: ****true***

*},*

*“HASCHILDREN”: {*

*“content”: ****false***

*},*

*“HASPARENT”: {*

*“content”: ****false***

*},*

*“OBJECTNAME”: {*

*“content”: “LOCATIONS”*

*},*

*“USEINPOPR”: {*

*“content”: ****false***

*},*

*“LANGCODE”: {*

*“content”: “EN”*

*},*

*“ADDTOSTORESITEID”: {*

*“content”: “DEMOSITE”*

*},*

*“HASLD”: {*

*“content”: ****false***

*},*

*“AUTOWOGEN”: {*

*“content”: ****false***

*},*

*“STATUSIFACE”: {*

*“content”: ****false***

*},*

*“STATUSDATE”: {*

*“content”: “2020–12–06T21:24:42+00:00”*

*},*

*“ISREPAIRFACILITY”: {*

*“content”: ****false***

*},*

*“EXPECTEDLIFE”: {*

*“content”: 0*

*},*

*“REPLACECOST”: {*

*“content”: 0.0*

*},*

*“PLUSCLOOP”: {*

*“content”: ****false***

*},*

*“PLUSCPMEXTDATE”: {*

*“content”: ****false***

*},*

*“SHOWFROMDATE”: {*

*“content”: “2020–12–06T00:00:00+00:00”*

*},*

*“ASSIGNED”: {*

*“content”: ****false***

*},*

*“PLUSGPERMITREQ”: {*

*“content”: ****false***

*},*

*“PLUSGEXREGISTERED”: {*

*“content”: ****false***

*},*

*“PLUSGRISKASSESSED”: {*

*“content”: ****false***

*},*

*“PLUSGSAFETYCRIT”: {*

*“content”: ****false***

*},*

*“PLUSGENVCRIT”: {*

*“content”: ****false***

*},*

*“PLUSGCOMCRIT”: {*

*“content”: ****false***

*},*

*“PLUSGHISTORYPERIOD”: {*

*“content”: 365*

*},*

*“PLUSGOPENOMUID”: {*

*“content”: 1351*

*},*

*“PLUSGISTEMP”: {*

*“content”: ****false***

*},*

*“PLUSGISPASSVALVE”: {*

*“content”: ****false***

*},*

*“PLUSGLEAKING”: {*

*“content”: ****false***

*},*

*“PLUSGQUALITYCRITICAL”: {*

*“content”: ****false***

*},*

*“PLUSPUSAGETORECORD”: {*

*“content”: ****false***

*},*

*“PLUSSISGIS”: {*

*“content”: ****false***

*},*

*“BRDPLOCPRESINFO”: {*

*“content”: ****false***

*},*

*“BRDPRESTRICTED”: {*

*“content”: ****false***

*}*

*}*

*}*

*]*

*}*

*}*

From above return, we can see there are some redundant content. You can use parameter `_compact=1` to remove them:

```
https://yourmaximoserver/maxrest/rest/mbo/locations?location=PT100&_format=json&_compact=1
```

---
