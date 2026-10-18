---
title: "Maximo Asset/Location Hierarchy Query"
date: 2024-05-01
tags: [maximo, sql, database]
draft: false
---


In Maximo, Asset / Location hierarchy is a pretty standard data format for asset and location management. The following is an example of HVAC asset hierarchy in Maximo asset drilldown function.

![](https://cdn-images-1.medium.com/max/800/1*re4t8_dxEth34rhE2Kle9Q.png)

It would be very common for the business user asks to have asset / location hierarchy report along with other information. There is an out of box (OOO) Asset Hierarchy in Maximo with very limit information (only asset num and description fields). To query the asset / location hierarchy information along with other information, for example, level information, custom format description which can show the layer indent description make it looks like a tree instead of flat table.

In Maximo, Asset and location hierarchy are managed in a little different tables structure. The following would provide the sample sql query script to pull the asset or location hierarchy information.

(1) Asset Hierarchy

In Maximo, there are 2 major tables to maintain asset hierarchy information: ASSET and ASSETANCESTOR. Asset table contain asset detail information along with parent asset field, ASSETANCESTOR table contain asset hierarchy level and asset Ancestor (top level asset) information. The following sql script can help you pull HVAC asset 1359 hierarchy information in previous screenshot.

```sql
select a.ASSETNUM ,a.PARENT , s.HIERARCHYLEVELS AS level, a.DESCRIPTION, 
       lpad('.',(s.HIERARCHYLEVELS)*4) CONCAT a.DESCRIPTION as padded_asset
   from maximo.ASSET a INNER JOIN maximo.ASSETANCESTOR s
     ON a.ASSETNUM =s.ASSETNUM  
   where s.ANCESTOR  = '1359'
```

After execute the above script, we will retrieve the following data:

![](https://cdn-images-1.medium.com/max/800/1*H04YHZQMXInFPdS29MenHg.png)

It looks good except the order. If we would like to get the data with the tree structure order like below:

![](https://cdn-images-1.medium.com/max/800/1*fg8cd_V4cwRDFm46U-Mwsg.png)

we need to adjust our query to add a field for sorting purpose. The following is the updated script to pull the data with tree structure order.

```sql
with assetparent (assetnum,parent, level, DESCRIPTION,padded_asset,order_sequence  ) as (
   select a.ASSETNUM ,a.PARENT, 0 AS level, a.DESCRIPTION, 
      a.DESCRIPTION as padded_asset,CAST(a.ASSETNUM  AS nvarchar(1000)) AS order_sequence
   from maximo.asset a
   where a.ASSETNUM = '1359' 
   union all
   select a.ASSETNUM ,a.PARENT,p.LEVEL+1 AS level,a.DESCRIPTION, 
      lpad('.',(p.LEVEL+1)*4) CONCAT a.DESCRIPTION as padded_asset,
      CAST(p.order_sequence CONCAT '_' CONCAT a.ASSETNUM  AS nvarchar(1000)) AS order_sequence
   from maximo.ASSET a,  assetparent p where p.assetnum = a.PARENT      
)
select *
from assetparent ORDER BY order_sequence;
```

In above sql script, we used recursive query to generate the level and the field “order_sequence” for sorting as tree structure. After execute the above script, we can pull the Asset hierarchy from Maximo database as tree structure order.

![](https://cdn-images-1.medium.com/max/800/1*ICNTOx16bf-ssusiXxaZsg.png)

If your Maximo instance database is Oracle which supports “connect by” recursive query, you can simply the above query as below:

```sql
select
  assetnum,
  parent,
  level,
  lpad('.',level*4) || assetum as padded_asset
from
  maximo.asset
connect by 
  prior assetnum = parent and prior siteid = siteid
start with
  assetnum = '1359'
```

In Maximo, Locations table structure has a little difference with Asset table for hierarchy perspective. In locations table, the Parent field in Maximo is non-persistent field, that means this field doesn’t existing in Maximo Locations table. Maximo uses Lochierarch to maintain location parent-child relationship. The following script can pull location hierarchy data start from the parent location “HVACROOM1” from Maximo database.

```sql
with locparent (location,parent, level, DESCRIPTION, padded_loc,order_sequence  ) as (
   select l.location,h.PARENT, 0 AS level,l.DESCRIPTION, 
      l.DESCRIPTION as padded_loc,CAST(l.location AS nvarchar(100)) AS order_sequence
   from maximo.locations l INNER JOIN maximo.lochierarchy h
     ON l.LOCATION =h.LOCATION 
   where l.LOCATION  = 'HVACROOM1'
   union all
   select l.location,h.PARENT,p.LEVEL+1 AS level, l.DESCRIPTION, 
      lpad('.',(p.LEVEL+1)*4) CONCAT l.DESCRIPTION as padded_loc,
      CAST(p.order_sequence CONCAT '_' CONCAT l.location AS nvarchar(100)) AS order_sequence
   from maximo.locations l , maximo.lochierarchy h ,
      
           locparent p where p.location = h.PARENT   AND  l.LOCATION =h.LOCATION      
)
select *
from locparent ORDER BY order_sequence;
```

And if your Maximo database instance is Oracle, you can simplify the above query as following:

```sql
select
  l.location,level,l.description, lpad('.',level*4) || l.location as padded_loc
from
  maximo.lochierarchy h  INNER JOIN maximo.location l  ON h.location=l.location 
connect by 
  prior h.location = h.parent and prior h.siteid = h.siteid
start with
  h.location = 'HVACROOM1'
```

---
