---
title: "Introduction of Maximo NextGen Rest APIs — Part 1 Query Maximo Data"
date: 2023-03-28
tags: [maximo, rest-api, integration]
draft: false
---


I received couple private messages from audiences for my previous blogs relate [Azure IoT integration with Maximo](/posts/azure-iot-integration-maximo-part1/) to ask for more detail on the instruction on Maximo NextGen Rest API. I would like to have these serial blogs to introduce how to query and update data through Maximo NextGen Rest API. In this part 1 blog, I will provide the information on Maximo NextGen Rest API, and how to construct Object Structure (OS) in Maximo and query OS data through Maximo NextGen Rest API. In [Part 2 blog](/posts/maximo-nextgen-rest-apis-part2-update-data/), I will leverage the same OS in part 1 blog to update Maximo data through NextGen Rest API. After you walk through the experiment of these serial blogs, you should have clear picture on how to query and update Maximo data through IBM recommended approach NextGen Rest API.

In one of my previous blogs “[How to query data in Maximo through Restful API](/posts/maximo-query-data-restful-api/)”, the rest APIs on this blog were released after Maximo Asset Management version 7.1. From [IBM Maximo](https://sedintechnologies.com/maximo-enterprise-asset-management/)version 7.6.0.2 onwards, IBM has provided a new REST APIs call NextGen Rest API which are a complete rewrite of the existing REST APIs. There are numerous benefits of using the NextGen APIs with a few highlighted here:

● Significantly enhanced support for querying maximo data — subselects, related object queries, multi-attribute text search, custom queries (java/scripting).

● Support for system level actions — bookmarking, notifications, e-sig, image association etc.

● Tight integration to automation scripts — query, actions, custom apis.

● Enhanced REST support for MIF Standard Services — support json data type for those service methods.

● Metadata support using json schema.

● Supports dynamic query views.

● Support for group by queries.

● Supports custom json elements appended to the Object structure JSON.

● Integration with Maximo Cache framework.

● Integration with Maximo formulas.

● Integration with Maximo Federated MBOs/Resources

The following is high level overview of the architecture

![](/images/1*bCecrQuhaYw9sob6g_C-Cg.png)

We will have the experiment on this blog in the following sections about Authentication, Object Structure and Options on NextGen APIs query.

And we will use Postman as the API tool in this experiment. You can download it through official site: [https://www.postman.com/downloads/](https://www.postman.com/downloads/)

- **Authentication**

Maximo NextGen Rest APIs support 2 authentication approaches: Maximo native authentication and LDAP based authentication.

(1) Maximo native authentication

This is where Maximo owns the user repository along with the user credentials. Maximo is responsible for authenticating the incoming REST call. The REST api expects the HTTP request with a “maxauth” request header that has a base64 encoded userid:password. For example, if your maximo user account is maxadmin, password is maxadmin, the user credential format is *maxadmin:maxadmin. *And then use base64 encryption tool (for example [https://www.base64encode.org/](https://www.base64encode.org/)) to encode the string *maxadmin:maxadmin. *The encode string is *bWF4YWRtaW46bWF4YWRtaW4=*

![](/images/0*BYSbVF8NZ2efCaOK.png)

We will use this authentication approach in our experiment on this blog.

(2) LDAP based authentication

This is where Maximo does not own the authentication credentials. Its owned by the application server (for example WebSphere), and hence the authentication is validated by the application server.

**2. Object Structure (OS)**

An object structure (OS) is the common data layer that the integration framework components use for sending and receiving data in Maximo. An object structure consists of one or more related Maximo business objects (MBO) that define the content of an integration message.

An object structure provides the message content for channels and services and enables application-based import and export. In addition, an object structure, on its own, can be called as a service, supporting the Create, Update, Delete, Sync, and Query operations.

In the next sections, assume you would like to explore Asset data along with Asset specifications information to the external system, we will build a new OS in Maximo to structure the asset and asset specification.

Login Maximo, open Object Structure Application, click “New Object Structure” Menu on the left panel or Create button on the tool bar. Fill out object structure name as MXASSETSPEC, and other required field.

![](/images/1*4bMpUfyXY9UgvBUMWjg8ew.png)

Under Source Object section, click “New Row” Button, select ASSET from the Object lookup list.

![](/images/1*hCsKAgTeGrBkzx_aW0R9FQ.png)

Click “New Row” button again, select ASSETSPEC from the Object lookup list. Pick Asset as Parent object, and select ASSETSPEC relationship from relationship lookup list.

![](/images/1*tBP_5UA51C6YPQxTNus0cw.png)

Save the above change. We built an OS with Asset and Asset Specifications structure, which we can leverage to query the asset with its specification information in next step.

**3. Query through NextGen Rest API**

Open Postman, create a new tab. Click headers, at the blank row, input “maxauth” at the key column, please refer to Maximo native authentication section to get your base64 encode string of your Maximo login account and password, and then paste it to the value column.

![](/images/1*y0W7oQmXcODp2IquoqM9gA.png)

Change the {[your maximo server url](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)} to your maximo server name and port, and put it to the postman url.

[http://{your maximo server url}/maximo/oslc/os/mxassetspec?lean=1](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)

Please Make sure the request action is GET, and click “Send” button after the url box. You should receive 200 return status code and the response payload in the response Body section. I will explain more detail on the url and response content in the next section.

![](/images/1*BFJEpqtgY7ju3v_SDb4nyg.png)

In the response content, it returns MXASSETSPEC resource (which is based on the ASSET Mbo in Maximo) — each member pointing to an asset record with a “href” that contains the link to get the details for that MXASSETSPEC. Any collection resource in this REST api will follow this same basic structure.

You might ask how to return all of the members’ attributes along with values instead of need to loop each url in the members to get detail of each Asset along with Asset Spec information. The following are the most important aspects this NextGen Rest APIs framework to query the data.

● Being able to select the list of attributes that we want the API to fetch. Being able to filter and sort Maximo business objects using a higher level query language which internally will map to the native sql for the corresponding relation db used by Maximo deployment.

● Being able to fetch data from related objects leveraging existing Maximo relationships — without needing any additional configuration.

● Being able to page data as needed.

Let’s get into more detail on each of them with the example OS MXASSETSPEC we built before.

(1) Select attributes, Filter and Sort

In NexGen Rest API, we can use the below query parameters to query the data.

**oslc.select**: this is used to specify the set of attributes to fetch from the Object structures as well as the related objects.

**oslc.where**: This is used to specify the where clause which follows the syntax detailed in the next section (Where clause syntax).

**oslc.orderBy**: this is used to specify the order by clause.

Now assume we have the following requirements to query the Asset and Asset Specifications data through NextGen Rest API:

● Return the fields of assetnum,location,description and status from Asset MBO, return the fields of assetattrid,description,alnvalue,numvalue abd tablevalue from Child AssetSpec MBO.

● Only show the assets with status equals to OPERATING.

● The return asset data need to be sorted by ASSETNUM field in ascending order.

For the first requirement, we can constructure oslc.select query paramenter as following. For the fields in child MBO AssetSpec, using the notation — <child objectname>{comma separated attribute names or * to get all properties}. So assetmeter{*} is going to fetch all properties for the assetmeter. This notation can easily be nested any levels deep. For example you can do obj1{prop1,prop2,obj2{prop21,prop22}} — where obj2 is defined as a child object of obj1.

> oslc.select=assetnum,location,description,status,assetspec{assetattrid,description,alnvalue,numvalue,tablevalue}

For the second requirement, we can use query parament oslc.where to implement it. This internally maps to the Maximo Qbe framework. We can filter data based on all persistent attributes — at the main MBO (for the os) or any related MBO. Please refer to the below example based on the second requirement:

*oslc.where=status=”OPERATING”*

Apart from “=” operator in *oslc.where, *we can use below operators to achieve more complicate query.

![](/images/1*3gTF8Qe1JJpcBpt8BLIwVQ.png)

For the 3rd requirement, to specify what attributes (fields) we would like to return from the API, we can identify it in *oslc.oslc.orderBy* query parameter. The format for this is

*oslc.orderBy=-attr1,+attr2*

The attributes prefixed with the minus sign (-) will be sorted in the descending order and the ones with the plus (+) sign will be sorted in the ascending order. All attributes listed here should be prefixed with a + or — sign. There is not default sort order here. Also related attributes are not supported here. Only persistent attributes from the main object are supported here. The following is the query parameter example to meet the requirement:

*oslc.orderby=+assetnum*

Now let’s consolidate what we discussed above into the url. Change the {[your maximo server url](http://stellarmaximolab.eastus.cloudapp.azure.com:9080/maximo/oslc/os/mxassetspec?lean=1)} to your maximo server name and port, and put it to the postman url.

*http://{your maximo server url}/maximo/oslc/os/mxassetspec?lean=1&oslc.select=assetnum,location,description,status,assetspec{assetattrid,description,alnvalue,numvalue,tablevalue}&oslc.orderby=+assetnum&oslc.where=status="OPERATING"*

After click Send button after the url box in Postman, you should receive response similar to this:

![](/images/1*iJZsL907rHzUMN8DWNt_4Q.png)

(2) Fetch data from related objects

In pervious *olsc.select* section, we discussed on how to identify return attributes for the child MBO in the OS with notation *<child object name>{comma separated attribute names or * to get all properties}*. While this notation works for child objects, you often will need to get more data from a related objects (say Location for the Asset) which is not defined in the Object structure. The notation below will help us do this.

*oslc.select=assetnum,status,description,location,location.description,location.status*

Let’s change url in postman as following to return Asset’s location information, such as location.description, location.status:

*http://{your maximo server url}/maximo/oslc/os/mxassetspec?lean=1&oslc.select=assetnum,location,description,status,location.description,location.status,assetspec{assetattrid,description,alnvalue,numvalue,tablevalue}&oslc.orderby=+assetnum&oslc.where=status="OPERATING"*

We will get location information at location section in the response content. Please keep in mind we don’t have Locations MBO in the MXASSETSPEC OS we built before.

![](/images/1*awBNXBnRav5Iz-sUPdzrVA.png)

(3) Page data

Paging of data is good fit to the scenario when your query return a big amount of the data. Query parameter *oslc.pageSize* is used to specify the max records to fetch for a page. This causes the responseInfo object to have the nextPage , previousPage , pagenum , totalCount (optional) and totalPages (optional) properties to describe page navigation information. If we don’t specify this pagesize query parameter in our query, the default page size setting will use the setting in OS. An attribute in the Object structure application to set the auto-paging threshold.

![](/images/1*q1uGMZjQbL_t419KpZHDdQ.png)

> Note:This field AUTOPAGINGTHRESHOLD was available on backend in Maximo 7.6.1.2.

Let’s modify our previous url to be the following to add pagesize query paraneter:

*http://{your maximo server url}/maximo/oslc/os/mxassetspec?lean=1&oslc.select=assetnum,location,description,status,location.description,location.status,assetspec{assetattrid,description,alnvalue,numvalue,tablevalue}&oslc.orderby=+assetnum&oslc.where=status="OPERATING"&oslc.pageSize=10*

We will find page information at the end of response.

![](/images/1*2Id2CXvTGZ155mC6JVnITA.png)

Now let’s recap what we discuss on this blog. I introduced authentication of Maximo NextGen Rest APIs, how to build Object Structure (OS) in Maximo, and how to leverage query parameters to query OS data through NextGen Rest APIs.

In [next blog](/posts/maximo-nextgen-rest-apis-part2-update-data/), I will introduce how to update Maximo data through NextGen Rest APIs.

If you think this blog helping you, welcome to buy me a coffee through below Green *Give a Tip* button :)

---
