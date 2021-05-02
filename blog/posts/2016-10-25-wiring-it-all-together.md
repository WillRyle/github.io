---
layout: post
title: Wiring it All Together
date: 2016-10-25 21:59
author: willryle
comments: true
categories: [Irrigator]
---
This post, the third in the irrigator series (<a href="https://willryle.wordpress.com/2016/10/16/autonomous-irrigation/" target="_blank">Part 1</a>, <a href="https://willryle.wordpress.com/2016/10/18/turning-motors/" target="_blank">Part 2</a>), covers how the device's hardware was wired into a coherent system. The key technology in this respect is the curiously named "<a href="http://nodered.org/" target="_blank">Node-RED</a>". It is described as a 'visual tool for wiring the Internet of Things', which I didn't find particularly helpful at first and had to dig a little deeper to gain some understanding.

<!--more-->

I decided to first get to grips with Node-RED on the Raspberry Pi and noticed that it helpfully came pre-installed. I'd got as far as a Python script to run an 'irrigation sequence', so looked at how Node-RED would let me invoke this from a remote process. The nodes in Node-RED are many and varied and I soon discovered the executable node which I could use to start the Python script. An 'inject' node can be used for testing and timer-based initiation, the idea being that the nodes are 'wired' into a sequence that forms a 'flow'. A message containing data is passed from one node to the next as each node completes its task. A debug node allows these messages to be viewed in the debug console. A function node allows JavaScript functions to be run to do bespoke tasks, like extracting parts of the message. I then discovered the Watson input node and wired this in as the entry point to the flow.

[caption id="attachment_2308" align="aligncenter" width="640"]<a href="https://willryle.files.wordpress.com/2016/10/irrigatorflow.jpg" target="_blank"><img class="wp-image-2308 size-large" src="https://willryle.files.wordpress.com/2016/10/irrigatorflow.jpg?w=640" alt="irrigatorflow" width="640" height="276" /></a> The Node-RED flow running on the Raspberry Pi with Twitter Out and Watson Out nodes added.[/caption]

Searching for means of invoking the flow from the Watson IoT Foundation (IoTF), I came across the <a href="https://developer.ibm.com/recipes/tutorials/sending-mqtt-commands-from-a-bluemix-app-to-a-device/" target="_blank">IoTSend application</a> and was soon able to use this to send messages manually to the device, via my IoTF service on Bluemix.

I still needed a means to invoke the irrigator automatically following an informed decision to water the garden and turned to Bluemix, guessing that a Node-RED application there may be key. A few minutes into browsing the available nodes and the solution was obvious. The two  Node-RED applications are effectively a device/server application in the manner of traditional client/server apps. Server and device each have their own flows and pass data between them using the input and output nodes of the appropriate type. The IoTF out node on the server allows the device parameters to be configured and a particular command given like 'waterGarden'. On the device, an IoTF input node is configured to respond to this command and the flow starts the irrigation sequence.

I added a Weather Data node to get the forecast and current conditions for our location by specifying the latitude and longitude, then mocked up the irrigation decision process - that's another job on the to-do list - and added a Twitter out node to tweet 'irrigation starting'. This last a substitute for the proposed mobile application's push notification.

[caption id="attachment_2313" align="aligncenter" width="640"]<a href="https://willryle.files.wordpress.com/2016/10/serverflow-irrigate.jpg" target="_blank"><img class="wp-image-2313 size-large" src="https://willryle.files.wordpress.com/2016/10/serverflow-irrigate.jpg?w=640" alt="serverflow-irrigate" width="640" height="274" /></a> Server-side Node-RED flow to initiate an irrigation sequence by sending a 'waterGarden' command to the device.[/caption]

Having got the device's hardware working, wiring the nodes - once I'd worked out what to do - was ridiculously easy. I added a second server-side flow to save data to the database in response to the device's IoTF out node on completion of the irrigation sequence. Adding an 'irrigation complete' tweet for example was just a matter of dropping a twitter out node onto the flow, allowing it to use the <a href="https://twitter.com/CognitiveALIS" target="_blank">CognitiveALIS</a> account and wiring it to get invoked on completion of the irrigation sequence, with just a bit of formatting to prevent it exceeding the maximum length of a tweet.

Overall I'm impressed with Node-RED. I like the visual way of working and the nodes are specific to a particular task to keep developers on the right path. I look forward to using it in the day job.
