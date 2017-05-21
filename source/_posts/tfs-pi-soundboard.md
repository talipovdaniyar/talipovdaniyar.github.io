---
title: Hear your TFS Builds fail with a Raspberry Pi Soundboard
date: 2016-12-24 21:35:54
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
Breaking the build is not the end of the world. What's really important is noticing it on time and fixing it as quickly as possible. Therefore you need a notifier that warns you when a build has failed.
<!-- excerpt -->

Breaking the build, it can happen to the best of us. Usually when a build fails the concerned developer or the whole dev team receives an email with the stacktrace. But all too often, there is a latency between breaking the build and noticing it being broken. In the worst case scenario this can lead to other developers retrieving broken code and continue building on it.

It's crucial to have a [fail-fast](https://en.wikipedia.org/wiki/Fail-fast) development flow. There are good alternatives out there, that use more team oriented communication platforms. Like e.g. linking build events to a Slack channel or even [shooting missiles](https://github.com/codedance/Retaliation) at the culprit.

However I decided to build one of my own. A webservice that plays the sound of Septa Unella's bell if a build fails.
<br>
<iframe src="//giphy.com/embed/Ob7p7lDT99cd2" width="480" height="268" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

## 1. Setup
The webservice will be hosted on a RaspberryPi 2 and will be triggered by Team Foundation Server 2017. We will start with an already configured repository on TFS and a Pi that runs on Windows IoT Core. In case you still need to install Windows IoT Core on your Pi you can follow these [Get Started](https://developer.microsoft.com/en-us/windows/iot/GetStarted) instructions.  

![pi-tfs](http://res.cloudinary.com/dcgoisyp0/image/upload/b_rgb:fff,bo_0px_solid_rgb:000,c_scale,q_100,r_0,w_348/v1482185390/pi-tfs.png)

By browsing to the IP address of your Pi (Port 8080) you will get the Pi's Dashboard. Add the mp3 file under the CameraRoll directory in the App File Explorer tab. This is the sound that will play when a build fails.

## 2. Code
### 2.1. Project Architecture

To my knowledge there (still) isn’t a native Web Api template for IoT devices. But luckily for us there is a library called [Restup](https://github.com/tomkuijsten/restup) that enables us to create a webservice and host it on a IoT device.

The Solution will contain 2 projects:

1. The **PiSoundboard.Api** project
2. The **PiSoundboard.Daemon** project

<script src="https://gist.github.com/talipovdaniyar/128a4556101207d12401f186125ea2a4.js"></script>

### 2.2. PiSoundboard.Api
The PiSoundboard.Api project is a basic Class Library (Universal Windows). The only package you will have to include is [Restup](https://www.nuget.org/packages/Restup/).

#### Data Transfer Object
The Api will be triggered by a POST call from a TFS webhook. I extracted The DTO structure from the POST call and this is what it looks like:
<br/>
<script src="https://gist.github.com/talipovdaniyar/958183939a53829b7ac7ffc356538269.js"></script>

With all that data you can let your imagination go wild and build some really cool features. Like e.g. light up a red lightbulb if the build is broken or even assign different lightbulbs to different developers.


#### Controller

Now we can start with building the SoundboardController. We need the Windows.Media references for playing the sound, located in the CameraRoll directory. The rest of the code syntax should be familiar to anyone who has some Web Api experience.
<br/>
<script src="https://gist.github.com/talipovdaniyar/0b09f8f1069f5a29b9cb0da8370ca85e.js"></script>

Most of the sound playing code is currently located in the controller but should be moved to a service layer if you plan on expanding the features.

#### ApiStartup

The ApiStartup is the entrypoint for the IoT Background Service and it will be responsible for registering the controllers and starting the webservice on port 8800.
<br/>
<script src="https://gist.github.com/talipovdaniyar/96f54ee1e769de3d3fd017083265f7ff.js"></script>

### 2.3. PiSoundboard.Daemon

#### StartupTask
Now we will create the PiSoundboard.Daemon project, it's a Background Application (IoT). After creating the project you will notice that it already has the StartupTask.cs file. This will be the starting point of the whole Application. This StartupTask will initialise the Api.
<br/>
<script src="https://gist.github.com/talipovdaniyar/dcac9c936126a7a4dc464a25d99a4ca8.js"></script>

#### Appxmanifest
You will have to include some extra configurations to use features of your Pi. Make sure the following capabilities are included in the Package.appxmanifest.
<br/>
<script src="https://gist.github.com/talipovdaniyar/e5ee6d54ceac23dc3d68b5ca53b12dc8.js"></script>

### 2.4. Deploy

You can deploy your app to your Pi by running the app in Visual Studio. Make sure you run the app with the Remote Machine configuration and the platform is set to ARM. Once the app is running on your Pi it will stay on the Pi until you remove it manually. If you want to add it to the startup of your Pi you can do it by opening the Apps tab on the Pi Dashboard screen.

Once your app is deployed, I recommend you to first test your webservice with a Rest Client, I used [Postman](https://www.getpostman.com/)

## 3. TFS
Now that the Raspberry Pi can handle the POST requests, we only need to configure TFS. You can follow this [explanation](https://www.visualstudio.com/en-us/docs/integrate/get-started/service-hooks/services/webhooks) for configuring webhooks on your TFS repository.

## 4. Conclusion
Initially this small project started as an inside joke. However, I do think it can be useful in the development cycle. Just make sure that everyone agrees to the new soundboard notifier. Inform the less tech-savvy colleagues about what a RaspberryPi is and reassure them you aren't sniffing the network traffic.

<a class="post-action-btn btn btn--success" href="https://github.com/talipovdaniyar/PiSoundBoard"><i class="fa fa-github"></i> Get Code</a>
</br>
