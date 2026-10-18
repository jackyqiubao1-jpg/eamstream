---
title: "Maximo Login Error “loginerror.jsp?group=system&key=NoLogin”"
date: 2023-04-14
tags: [maximo, troubleshooting]
draft: false
---


Recently I tried to install Maximo Spatial package to a Maximo instance. After the Spatial package was installed, I tried to login Maximo to verify the result of the installation, got the following error:

*Login Error You cannot login at this time. Contact the system administrator*

And the error page parameters are loginerror.jsp?group=system&key=NoLogin

![](https://cdn-images-1.medium.com/max/800/1*J9WxoRVsW5cBwctA27x-VQ.png)

Investigated on the log, cannot find any useful information since no exception happened for this case. After research through Mr. Google, looks like it was caused by useAppServerSecurity was updated issue. To verify my assumption, opened the file *maximo.properties* (default location in Windows *C:\IBM\SMP\maximo\applications\maximo\properties*), the value mxe.useAppServerSecurity=1, means it will use LDAP authentification. But in my Maximo instance I was configured it used Maximo application authentication. It might be updated during the Maximo Spatial package installation.

![](https://cdn-images-1.medium.com/max/800/1*4BbEnspVkw7vUKM3mRBdjg.png)

And checking the *useAppServerSecurity *property value in the *web.xml* file under the following folders:

*- IBM\SMP\maximo\applications\maximo\maximouiweb\webmodule\WEB-INF
- IBM\SMP\maximo\applications\maximo\mboweb\webmodule\WEB-INF
- IBM\SMP\maximo\applications\maximo\meaweb\webmodule\WEB-INF
- IBM\SMP\maximo\applications\maximo\maxrestweb\webmodule\WEB-INF*

I found the *useAppServerSecurity *value was set to 1 as well.

![](https://cdn-images-1.medium.com/max/800/1*9jt9ZGRv-Ds2Sm0iTooWfA.png)

After performed the below steps, the issue was solved.

**Step-1: Update *maximo.properties *File under the folder *C:\IBM\SMP\maximo\applications\maximo\properties***

We need to change *mxe.useAppServerSecurity=1* to *mxe.useAppServerSecurity=0* on this file. You cannot modify this file directly, since it was encrypt. Follow the below sub steps to modifiy this file.

(1) Delete or rename the existing maximo.properties

(2) Copy* *the file c:*\ibm\smp\etc\maximo.properties_orig* to folder c:*\ibm\smp\maximo\applications\maximo\properties*\

(3) Rename *maximo.properties_orig *to be* maximo.properties.*

(4) Open this file,

update *mxe.useAppServerSecurity=1 *to be* mxe.useAppServerSecurity=0,*

Add *mxe.db.password={Replace with your DB password} *line under below session:

![](https://cdn-images-1.medium.com/max/800/1*uffd0OMw8P9niwDdRJcXwQ.png)

Save the changes.

(5) From an Administrative command prompt, navigate to *c:\ibm\smp\maximo\tools\maximo* directory

(6) Run encryptproperties.bat
This command encrypts the database user password in maximo.properties.

**Step-2: Update *web.xml* files**

Open *web.xml* file under folder c:\*IBM\SMP\maximo\applications\maximo\maximouiweb\webmodule\WEB-INF*

Search *useAppServerSecurity in web.xml, and u*pdate *useAppServerSecurity *value to be 0.

![](https://cdn-images-1.medium.com/max/800/1*NfJQQwuqnh-KuXrkBGq9Yg.png)

Search *security-constraint* and comment all of the* security-constraint *sections.

![](https://cdn-images-1.medium.com/max/800/1*WOX0Q7JPEMhmDmuziBJ6rA.png)

![](https://cdn-images-1.medium.com/max/800/1*oGtg4qC3LUHpFdT0VQ4cAQ.png)

Search *login-config* and comment *login-config* section for ***BASIC ***auth-method

![](https://cdn-images-1.medium.com/max/800/1*mh7m7Y3TXbsDCbVtA_OMyw.png)

Uncomment *login-config* and comment *login-config* section for ***Form ***auth-method

![](https://cdn-images-1.medium.com/max/800/1*QEK4PjYnBYNSS45oCJDReg.png)

Save the above changes.

And make the change to *u*pdate *useAppServerSecurity *value to be 0 in *web.xml *file under the following folders
- *IBM\SMP\maximo\applications\maximo\mboweb\webmodule\WEB-INF
- IBM\SMP\maximo\applications\maximo\meaweb\webmodule\WEB-INF
- IBM\SMP\maximo\applications\maximo\maxrestweb\webmodule\WEB-INF*

**Step-3: Rebuild and redeploy the Maximo application.**

You can use Configuration Tool *ConfigUI.exe* under the folder *C:\IBM\SMP_2\ConfigTool,* use the option “Update Database and Build and Deploy Application EAR files” to rebuild and redeploy the Maximo application.

![](https://cdn-images-1.medium.com/max/800/1*MhF8HdY0sbKd9eaeBLqnzQ.png)

Or you can follow the step on this page to rebuild and redeploy it manually.

[https://www.ibm.com/support/pages/node/416993](https://www.ibm.com/support/pages/node/416993)

After Maximo application deploy complete, I can get my Maximo login page back. And after login, I can verify Maximo Spatial package is installed.

![](https://cdn-images-1.medium.com/max/800/1*XWcoS5RkxZPQzSskk4ik4A.png)

If you think this blog helping you, welcome to buy me a coffee through below Green *Give a Tip* button :)

---
