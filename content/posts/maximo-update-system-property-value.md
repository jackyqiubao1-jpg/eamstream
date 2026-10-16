---
title: "Maximo: How to update system property value"
date: 2022-01-25
tags: [maximo, administration]
draft: false
---


In Maximo, there are a lot of system default property for different purposes. You can define your own property in System Properties Application, which can be used in your customized script code, configuration, or Maximo Integration Framework (MIF).

As default setting, you cannot change the value for system default property. For some special scenarios, you might have to change the system property default value. To be able to modify system default property, the following instruction will provide an example on how to change the property: mxe.int.binarytext

Connect to Maximo database, run the following script against Maximo database (You need to have update permission to the maximo database):

> update maximo.maxprop set userdefined=1 where propname = ‘mxe.int.binarytext’

Go to Maximo -> System Configuration -> Platform Configuration -> System Properties. Filter by mxe.int.binarytext in the column property name to locate the record:

![](https://cdn-images-1.medium.com/max/800/1*Kg4oA3DEbIr-k7EikXj8ig.png)

Check Online Changes Allowed? checkbox, and Save the change.

![](https://cdn-images-1.medium.com/max/800/1*JDN16fzzcIBmljc6bhujwg.png)

Now you can modify Global Value field. After modify the Global Value, Save the change, and click Live Refresh from action list to apply the global value to current value.

![](https://cdn-images-1.medium.com/max/800/1*571mxWbLYUsCxzooKxstrg.png)

***Warning: Please make sure your fully understanding the impact of the system property you want to update before you make the change.***

---
*Originally published on [Medium](https://medium.com/@jackyqiubao/maximo-how-to-update-system-property-value-8d7061196b36).*
