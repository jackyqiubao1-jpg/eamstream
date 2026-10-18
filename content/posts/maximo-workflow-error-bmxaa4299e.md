---
title: "Maximo Workflow Error: BMXAA4299E"
date: 2020-12-30
tags: [maximo, workflow, troubleshooting]
draft: false
---


Recently encountered a pretty strange issue when tried to release an simple Maximo workflow to production environment. The workflow is to allow business team (Maximo Support team) to review new service request, to decide to create related work order, or cancel service request. The following is the workflow diagram:

![](https://cdn-images-1.medium.com/max/800/1*pWNz4k3EJeLpNmhD-mxCaQ.png)

This workflow is working fine in testing environment. When we implemented the same workflow in production, doing the production check-out: created a new SR, attached it to this workflow, started the workflow, got the error message: BMXAA4299E

![](https://cdn-images-1.medium.com/max/800/1*euKwtR7LU3stMZTch0HbqA.jpeg)

Tried to google BMXAA4299E, but got little related result on this issue. After couple hours’ research, finally figure the issue caused by the current user is not in the person group which is assigned to the role on task node:

![](https://cdn-images-1.medium.com/max/800/1*DQ2h4iPqaao69k6mPnYRTA.png)

After assign the current user mapped person to the person group which assigned to the role in the task node, the issue was solved. Hope this article will give you the hint if you encounter the same error message id BMXAA4299E.

If you think this blog helping you, welcome to buy me a coffee through below Give a Tip button after this blog :)

---
