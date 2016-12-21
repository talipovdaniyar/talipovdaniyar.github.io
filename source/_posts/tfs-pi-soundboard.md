---
title: Hear your TFS Builds fail with a Raspberry Pi Soundboard
date: 2016-12-19 21:35:54
thumbnailImage: http://res.cloudinary.com/dcgoisyp0/image/upload/v1482276781/pi-tfs-thumb.png
autoThumbnailImage: yes
thumbnailImagePosition: left
clearReading: true
tags:
  - Windows IoT Core
  - TFS
  - RaspberryPi
  - C#


keywords:
  - IoT
  - Windows IoT Core
  - Internet of Things
  - Team Foundation Server
  - TFS 2017
  - RaspberryPi
  - C#
---
Breaking the build is not the end of the world. What's really important is noticing it on time and fixing it as quick as possible. Therefore you need a notifier that warns you when a build has failed.
<!-- excerpt -->

Breaking the build can happen to the best of us. Usually when a build fails the concerned developer or the whole dev team receives an email with the stacktrace. But all too often, there is a latency between breaking the build and noticing it being broken. In the worst case scenario this can lead to other developers retrieving broken code and building on it.

## Fail-Fast
It's crucial to have a [fail-fast](https://en.wikipedia.org/wiki/Fail-fast) development flow. There are alternatives out there that use different types of communication platforms, that are more team oriented. You can e.g. link build events to a Slack channel or even [shoot missiles](https://github.com/codedance/Retaliation) at the culprit. However I decided to create one of my own. A webservice that plays the sound of Septa Unella's bell if a build fails.
<br>
<iframe src="//giphy.com/embed/Ob7p7lDT99cd2" width="480" height="268" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

## 1. Setup
The webservice will be hosted on a RaspberryPi 2 and will be triggered by Team Foundation Server 2017. We will start with an already configured repository on TFS and a Pi that runs Windows IoT Core. In case you still need to install Windows IoT Core on your Pi you can follow the [Get Started](https://developer.microsoft.com/en-us/windows/iot/GetStarted) instructions.  

![pi-tfs](http://res.cloudinary.com/dcgoisyp0/image/upload/b_rgb:fff,bo_0px_solid_rgb:000,c_scale,q_100,r_0,w_348/v1482185390/pi-tfs.png)

By browsing to the IP address of your Pi (Port 8080) you will get the Pi's Dashboard. Under App File Explorer you can add a mp3 file that you wish to play when a build fails. Remember the name and location of the mp3 file.

## 2. Code
### 2.1. Project Architecture

To my knowledge there (still) isn’t a native Web Api template for IoT devices. But luckily for us there is a library called [Restup](https://github.com/tomkuijsten/restup) that enables us to create a webservice and host it on a RaspberryPI.

The Solution will contain 2 projects:

1. The **Soundboard.Api** project (hosting the Soundboard API)

2. The **Soundboard.Startup** project (initialising the API).

<script src="https://gist.github.com/talipovdaniyar/128a4556101207d12401f186125ea2a4.js"></script>


### 2.2. Soundboard.Api

#### 2.2.1. Data Transfer Object
The DTO that TFS will include in the POST call has really a lot of information. You could not only play a sound but even ligth up a red light bulb if a specific developer broke the build.


#### 2.2.2. Controller

First we will start of with the Soundboard.Api project, it’s a Windows Portable Library project. After creating the project, include the Restup nuget in your project. Now you should be able to start building the SoundboardController. We have to add some references to Media libraries for playing the sound located on the Raspberry Pi. The rest of the code syntax looks a lot like the ASP.net controllers syntax. Most of the code logic is currently located in the controller but should be moved to a service layer if the logic grows.

Snippet CONTROLLER



#### 2.1.2. Soundboard.Startup
Now we will create the IoT Service project with the following code in the main class.

As you can see the start method initialises the Api. Now try to deploy it to your Pi by pressing F5 but keep in mind that the running setting should be set on “Remote”. Once the deploy is finished you can test the Web Service by sending a POST with a HTTP client like Postman. Don’t forget to include Basic Authentication. By now you should hear the sound that you put on your Pi.

## 3. TFS
Now that the Raspberry Pi listens to Post requests we only need to configure TFS. Open your TFS dashboard and head to settings. Under setting you will have to create a WebHook a Build event on TFS. Select to which build event
