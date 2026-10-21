---
title: "How to shrink docker volume after delete containers / images in Windows 10"
date: 2022-07-01
tags: [docker, windows, devops]
draft: false
---


Docker becomes more and more popular in the development process, especially for the organization adopting DevOps process. I have couple projects require to use docker to build local development environment. My local hard disk space started to getting red after 2–3 rounds of rebuilding local docker environment for different branches of the code change. I did run rm command to remove containers, images and volumes. Since Docker Desktop for Windows uses WSL to manage all the images and container files and keeps them in a private virtual hard drive (VHDX) called ext4.vhdx. This vhdx will not automatically shrink when you remove docker, images or volumes. You need to follow the below step to shrink it.

(1) Quick from your docker desktop application

(2) Run Windows Powershell as Administrator

(3) Run the following command to shut down wsl

*wsl --shutdown*

(4) Run the follow command to mouth VHD file. Please modify yourusername to your current windows’ user

*Mount-VHD -Path C:\Users\yourusername\AppData\Local\Docker\wsl\data\ext4.vhdx -ReadOnly*

(5) Run the following command to optimize the VHD file.

*Optimize-VHD -Path C:\Users\jacky\AppData\Local\Docker\wsl\data\ext4.vhdx -Mode Full*

The Progress bar will be shown as following.

![](/images/1*t6mmh9rbAxQhmpEkYtkkjg.png)

After this step, you should be able to see your hard drive space coming back.

(6) Run the following command to dismount VHD file

*Dismount-VHD C:\Users\jacky\AppData\Local\Docker\wsl\data\ext4.vhdx*

(7) Restart your docker desktop. You all set.

If you think this blog helping you, welcome to buy me a coffee through below Give a Tip button after this blog :)

---
