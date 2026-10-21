---
title: "How to solve the issue of start center cannot be edited for being modified by someone"
date: 2020-11-15
tags: [maximo, troubleshooting]
draft: false
---


On one of the Maximo production release, we need to modify one of the start center template in production. But when click the start center template would like to modify, but got the following error:

BMXAA7941E — You cannot modify this template because it is being modified by another user

The issue was caused by someone open this start center template to modify, but forget to save or cancel the modification. It will keep this start center being locked by this user, until he/she save or cancel the change. We need to identify which user is modifying this template, and ask him / her to unlock the template by save / cancel. Unfortunately Maximo doesn’t provide the way to check who is locking the start center template. The only way you can figure out is through query the table “scconfig” in Maximo database directory. Open sql tool, execute the following sql script:

*select * from maximo.scconfig where groupname is null*

You will be able figure who (USERID column) is modifying the template (DESCRIPTION column)

![](/images/1*oGxQSMzormi3l7z4DGJ0Ww.png)

Ask the user to login Maximo, on the locked start center, click on the Save Template or Cancel Changes hyperlinks on the right top tool bar on the screen, will unlock the start center template.

How about if the user who is locking the template already left the organization? Or cannot be reached during the production release? Login as maxadmin security group user, find the user who is locking the start center template in the user application, reset the password for this user, and then login with this user to unlock the template.

It is really interested IBM doesn’t provide the function to allow maxadmin to unlock the start center, hope this function will be included in next release.

---
