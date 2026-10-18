---
title: "What is Maximo Recurrence of inbound transactions and how to use it?"
date: 2020-11-08
tags: [maximo, integration, mif]
draft: false
---


IBM Maximo is very powerful and flexible EAM Platform, especially its integration framework. In Maximo Integration Framework (MIF), there are a lot of options you can configure or develop inbound and outbound interfaces to integrate Maximo with other systems. I will try to introduce more details on MIF in other topics, this topic will introduce recursion of inbound interface in Maximo and how to use it in real business case.

What is Maximo inbound recursion? Simplify the explanation: An inbound transaction from external system to Maximo will trigger an outbound transaction from Maximo to external system. If we don’t have mechanism or event filter configuration setup correctly, might cause a dead loop for external system send inbound transaction again to Maximo which is triggered by the previous outbound transaction from Maximo. To avoid this issue happen, Maximo default to set recursion to be false, that means outbound transaction will not be triggered by inbound transaction. But for some special business cases, we do need to open this recursion. For example, when an service ticket from ServiceNow system, is sent in Maximo using the Integration Framework. In Maximo, it will create service request from this inbound transaction, after service request is created, need to send services ID (ticket ID) back to ServiceNow. But since Maximo default to disable inbound recursion, this will prevent Maximo outbound interface to send out ticket ID to ServiceNow. To enable outbound interface trigger by the inbound transaction, you just need to extend event trigger class *MaximoEventFilter *and overwrite the function *stopRecurrence *to return false (Maximo default to return true). Set this new customized event class in your publish channel which will allow your outbound interface trigger by inbound transaction. The following is the java code for the customized event class:

```
package psdi.iface.app.common;

import java.rmi.RemoteException;
import psdi.iface.mic.MaximoEventFilter;
import psdi.iface.mic.MicUtil;
import psdi.iface.mic.PublishInfo;
import psdi.mbo.MboRemote;
import psdi.server.MXServer;
import psdi.util.MXException;

public class StopRecurEventFilterextends MaximoEventFilter
{
    
    /**
     * This method stops recurrence, i.e it checks if this is incoming 
     * transaction and if the MetaDataProperties.SOURCETYPE is specified stops the event
     * from going out.
     * This method can be overriden by individual event filter classes to allow
     * recurrence.
     * 
     * @return - true for stopping the event and false otherwise.
     * @throws MXException
     *             MAXIMO exception
     * @throws RemoteException
     *             Remote exception
     */
    protected boolean stopRecurrence(MboRemote mbo) throws MXException, RemoteException
    {
    	
        return false;
    	
    }


}
```

On your publish channel for your outbound, set this customized event class psdi.iface.app.common.StopRecurEventFilter on in the Event Filter Class field.

![](https://cdn-images-1.medium.com/max/800/1*h5vdcinIB_imDtN8dqca0w.png)

Publish your java class to ear file and restart your Maximo applicaiton. Test inbound to see if it will trigger the outbound publish channel with this customized event class.

---
