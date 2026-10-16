---
title: "Modify Maximo data through Restful API"
date: 
tags: [maximo, rest-api, integration]
draft: false
---


In my previous blog I introduced [how to query Maximo data through Restful API](https://jackyqiubao.medium.com/how-to-query-data-in-maximo-through-restful-api-aef45c47c14e), in this blog, I will show you how to modify (Create, Update, Delete) Maximo data through Restful API.

You can modify Maximo resources by using the HTTP PUT, POST, and DELETE methods with Restful API. The PUT, POST, and DELETE methods can be used to modify business object resources and object structure resources. However, the business rules of an object might prevent it from being updated by a REST API request. For example, a DELETE request on a work order resource might fail if the business object validations prevent deletions because of the current state of the work order.

In reality, most of the integrations require using Object Structure (OS) instead of Maximo Business Object (MBO). The following practice will demonstrate on how to create  an OS in Maximo, and use this OS to create and update Maximo data.

Assume we have the business requirement that an external system will call Maximo restful API to create and update asset along with meters. We can create an object structure MXRAsset in Maximo to hold the structure as following:

As above picture, Asset is the primary MBO, Meter is child MBO. We can post data to Maximo restful API through postman. You can download postman through [https://www.postman.com/downloads/](https://www.postman.com/downloads/)

Open a new tab in postman
