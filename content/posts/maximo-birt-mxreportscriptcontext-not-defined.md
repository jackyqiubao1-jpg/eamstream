---
title: "Solution for Maximo BIRT Report Exception of “MXReportScriptContext” is not defined."
date: 2022-05-20
tags: [maximo, birt, troubleshooting]
draft: false
---


2 years ago was working on developing Maximo BIRT report in client laptop, which was installed firefox browser by default. There was not issue when I installed and configured BIRT report development environment at that moment. Recently in my project one of the my team member was on paternity leave, I had to roll up my sleeve again to pick up the a BIRT Report he was working which was required hard deadline (Firefighter is always my other role in the projects). Followed the instruction to install and configure my local BIRT report development environment, everything was good until tried to run the report in Eclipse, got the following error:

*org.mozilla.javascript.EcmaError: ReferenceError: “MXReportScriptContext” is not defined.
 at org.mozilla.javascript.ScriptRuntime.constructError(ScriptRuntime.java:3654)
 at org.mozilla.javascript.ScriptRuntime.constructError(ScriptRuntime.java:3632)
 at org.mozilla.javascript.ScriptRuntime.notFoundError(ScriptRuntime.java:3717)
 at org.mozilla.javascript.ScriptRuntime.nameOrFunction(ScriptRuntime.java:1758)
 at org.mozilla.javascript.ScriptRuntime.name(ScriptRuntime.java:1697)
 at org.mozilla.javascript.Interpreter.interpretLoop(Interpreter.java:3410)
 at org.mozilla.javascript.Interpreter.interpret(Interpreter.java:2487)
 at org.mozilla.javascript.InterpretedFunction.call(InterpretedFunction.java:164)
 at org.mozilla.javascript.ContextFactory.doTopCall(ContextFactory.java:398)
 at org.mozilla.javascript.ScriptRuntime.doTopCall(ScriptRuntime.java:3065)
 at org.mozilla.javascript.InterpretedFunction.exec(InterpretedFunction.java:175)
 at org.eclipse.birt.report.engine.javascript.JavascriptEngine.evaluate(JavascriptEngine.java:290)
 at org.eclipse.birt.core.script.ScriptContext.evaluate(ScriptContext.java:159)
 at org.eclipse.birt.report.engine.executor.ExecutionContext.evaluate(ExecutionContext.java:789)
 at org.eclipse.birt.report.engine.executor.ExecutionContext.evaluate(ExecutionContext.java:718)
 at org.eclipse.birt.report.engine.script.internal.ScriptExecutor.handleScriptInternal(ScriptExecutor.java:118)
 at org.eclipse.birt.report.engine.script.internal.ScriptExecutor.handleScript(ScriptExecutor.java:105)
 at org.eclipse.birt.report.engine.script.internal.ReportScriptExecutor.handleInitialize(ReportScriptExecutor.java:84)
 at org.eclipse.birt.report.engine.api.impl.EngineTask.loadDesign(EngineTask.java:1962)
 at org.eclipse.birt.report.engine.api.impl.RunAndRenderTask.doRun(RunAndRenderTask.java:99)
 at org.eclipse.birt.report.engine.api.impl.RunAndRenderTask.run(RunAndRenderTask.java:77)
 at org.eclipse.birt.report.debug.internal.core.launcher.ReportLauncher.createReport(ReportLauncher.java:620)
 at org.eclipse.birt.report.debug.internal.core.launcher.ReportLauncher.renderReport(ReportLauncher.java:566)
 at org.eclipse.birt.report.debug.internal.core.launcher.ReportLauncher.run(ReportLauncher.java:480)
 at org.eclipse.birt.report.debug.internal.core.launcher.ReportLauncher.main(ReportLauncher.java:124)
 at org.eclipse.birt.report.debug.internal.core.ReportDebugger.start(ReportDebugger.java:39)
 at org.eclipse.equinox.internal.app.EclipseAppHandle.run(EclipseAppHandle.java:196)
 at org.eclipse.core.runtime.internal.adaptor.EclipseAppLauncher.runApplication(EclipseAppLauncher.java:110)
 at org.eclipse.core.runtime.internal.adaptor.EclipseAppLauncher.start(EclipseAppLauncher.java:79)
 at org.eclipse.core.runtime.adaptor.EclipseStarter.run(EclipseStarter.java:354)
 at org.eclipse.core.runtime.adaptor.EclipseStarter.run(EclipseStarter.java:181)
 at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
 at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
 at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
 at java.lang.reflect.Method.invoke(Method.java:606)
 at org.eclipse.equinox.launcher.Main.invokeFramework(Main.java:636)
 at org.eclipse.equinox.launcher.Main.basicRun(Main.java:591)
 at org.eclipse.equinox.launcher.Main.run(Main.java:1450)
 at org.eclipse.equinox.launcher.Main.main(Main.java:1426)*

As usual tried to ask Mr. Google, but didn’t find any useful information on this error. There is the reason I wrote this blog tried to give hint to you to solve this issue if you happen to encounter the same issue.

After checked with the team who installed and configured BIRT report development environment, they mentioned need to use firefox browser to view report in Eclipse. That give me the hint after I reviewed the following error message:

org.***mozilla***.javascript.EcmaError: ReferenceError: “MXReportScriptContext” is not defined.

Firebox is also the product from Mozilla. Now it explain why it works fine on other team member’s environment which installed firefox browser.

The following steps is to fix the above issue:

(1) Download firefox from the below official site and install it.

(2) Open Eclipse, go to menu “Window -> Preference”, expand node “Report Design”, click on “Preview” node. Check “Always use external browsers”

![](https://cdn-images-1.medium.com/max/800/1*GeX9NMGwPc54J6F4g-VBEA.png)

(3) Click link “Web Browser” Link under the “Always use external browsers” checkbox. Select option “Use external web browser”, and check “Firebox” checkbox in the list. Click OK button to save the change.

![](https://cdn-images-1.medium.com/max/800/1*gsq09FgC0lmcqxhwYGeRTg.png)

(4) Restart Eclipse, and run your report through menu “Run -> View Report -> In Web Viewer”. Your report should be launched successfully now.

If you think this blog helping you, welcome to buy me a coffee through below Give a Tip button after this blog :)

---
