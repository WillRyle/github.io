---
layout: post
title: Autonomous Irrigation
date: 2016-10-16 22:05
author: willryle
comments: true
categories: [Irrigator, New Bolton]
---
A post detailing the progress so far on ALIS - warning: may contain technical content...

<strong>Update</strong> 17th Oct 11:50 am - assorted edits and additions to Possible Future Enhancements section.

<strong>Update</strong> 25th October - this is the first of three posts describing the irrigator system. The second post is <a href="https://willryle.wordpress.com/2016/10/18/turning-motors/" target="_blank">here</a> and the third <a href="https://willryle.wordpress.com/2016/10/25/wiring-it-all-together/" target="_blank">here</a>.

<!--more-->

<strong>Problem</strong>

A plot measuring 40 metres in length and 3.5 metres wide, planted with some 35-45 rows of assorted vegetables has a considerable maintenance overhead, especially at peak growing time where the Summer heat requires daily watering to prevent plant dehydration. Hand-held hose watering is time consuming while sprinklers often fail to apply water where required and are prone to strong winds dispersing the water indiscriminately. Fixed irrigation methods such as soaker hoses and drip waterers afford some measure of timer-based watering, but are prone to damage during weeding and harvesting and require reconfiguring each season where crop rotation varies the row spacing and position.

<strong>Requirements</strong>

<a href="https://willryle.wordpress.com/2016/10/09/alis-the-irrigator/" target="_blank">ALIS (Autonomous Linear Irrigation System)</a> is a response to the issues raised above and is intended to allow a flexible pattern of irrigation based on plant need rather than the availability of labour. ALIS consists of a number of cloud applications hosted on Bluemix and an irrigator device. The device will be configured with an irrigation sequence of instructions to move to the first row of plants and initiate watering for a pre-determined amount of time, after which it will cease watering and move to the next row and repeat for all rows in the irrigation sequence. It will return to the start point when not in use and will be reconfigurable season  after season to accommodate varying planting layouts. It will not rely on any fixed infrastructure beyond a retractable hose reel to supply water. It will use cloud technologies to analyse local weather data and determine the need for irrigation automatically. It will issue commands to the device to instigate irrigation. The device will report its runtime duration and water usage to the cloud application at the end of each irrigation sequence, this data being saved to a database for analysis. ALIS will work without any human intervention beyond defining the irrigation sequence and an opportunity to override the decision to irrigate, having been notified by mobile phone beforehand. The device will optionally submit photographs of each row watered in the irrigation sequence to allow verification of the sequence.

[caption id="attachment_2212" align="aligncenter" width="640"]<a href="https://willryle.files.wordpress.com/2015/11/irrigation-004.jpg" target="_blank"><img class="wp-image-2212 size-large" src="https://willryle.files.wordpress.com/2015/11/irrigation-004.jpg?w=640" alt="Irrigation 004" width="640" height="480" /></a> The Mark2 manually controlled irrigator[/caption]

<strong>Implementation</strong>

On the server-side, a  <a href="http://www.nodered.org" target="_blank">Node-Red</a> application is under development and consists of flows to do the following:
<ol>
 	<li>Periodically retrieve current and forecast weather information for the specified location via the Watson Weather Data service. Analyse this data to determine if irrigation is required based on current temperature, wind speed and humidity; and forecast rainfall and temperature. Where irrigation is determined necessary, optionally notify the operator by Push Notification to the mobile application, or initiate directly by means of a command to the device. Log the event and weather data to a Cloudant database.</li>
 	<li>Provide a means for the device to report data and save to a Cloudant database. A Watson IoT Foundation (IoTF) input node will accept data from the device.</li>
 	<li>A sub flow to examine the forecast and current conditions for the location to determine if irrigation is necessary.</li>
</ol>
<strong>Device Hardware</strong>

The device will consist of a wheeled platform, the two rear wheels being powered by 24v 8amp DC motors. A central pillar will allow the mounting of a counter-balanced, height adjustable irrigation boom equipped with water hose and spray nozzles. The boom will be asymmetric and of sufficient length to give coverage over the 3.5 metre width of the plot. Water supply will be via a 20 metre retractable hose, the reel fixed centrally along one side of the plot and the outlet secured to the top of the central pillar and connected to the boom-mounted water hose.  Device control and connectivity will be provided by a Wi-Fi equipped Raspberry Pi 3 Model B with camera, Arduino Uno and Pololu VNH5019 Dual Motor Controller. 5v to 12v relays will allow control of a water control valve and flow meter.
<ul>
 	<li>Raspberry Pi - running a variant of the Linux operating system. It is a registered device with an IoTF service on Bluemix. A Node-Red flow will run on the Raspberry Pi and accept commands from the IoTF service. On receipt of a command to irrigate, a Python script Irrigation Sequence will operate, sending motor commands to the Arduino and water on/off commands to the water valve.</li>
 	<li>Arduino Uno - connected by USB to the Raspberry Pi and acting as a high-level controller for the motor controller. The Arduino has a sketch deployed that listens for commands on the serial port. Commands consist of a motor direction, motor speed and duration. On receipt of a command, Arduino operates the motors via the motor controller's API commands, in the given direction and power for the given duration. The device will thus be driven from one row to the next.</li>
 	<li>Pololu VNH5019 - is a dual motor controller mounted as a HAT on the Arduino. This controller can control two motors of up to 24v DC at 12amps.</li>
 	<li>Water Control Valve - a 12v DC normally closed valve. Opens when a 12v current is applied.</li>
 	<li>Liquid Flow Meter - a pin-wheel sensor rotates when water is flowing. Counting the sensor pulses allows the water usage to be calculated.</li>
 	<li>5v-12v Relays - A board with an array of four relays allows the 5v signal from the Raspberry Pi to control larger voltage devices. One channel is currently used to allow switching of the water control valve. The remaining three are available for additional features.</li>
</ul>
<strong>Device Flow</strong>

[caption id="attachment_2308" align="aligncenter" width="640"]<a href="https://willryle.files.wordpress.com/2016/10/irrigatorflow.jpg" target="_blank"><img class="wp-image-2308 size-large" src="https://willryle.files.wordpress.com/2016/10/irrigatorflow.jpg?w=640" alt="irrigatorflow" width="640" height="276" /></a> Node-Red flow running on the Raspberry Pi. On receipt of a 'waterGarden' command, the flow invokes a Python script - 'runIrrigator' node - which controls the motors and water valve. At the end of the sequence the Watson IoTF Out node sends data to the SaveToDB flow on the server and a tweet is sent using the Twitter Out node.[/caption]

<strong>Server Flow</strong>

[caption id="attachment_2313" align="aligncenter" width="640"]<a href="https://willryle.files.wordpress.com/2016/10/serverflow-irrigate.jpg" target="_blank"><img class="wp-image-2313 size-large" src="https://willryle.files.wordpress.com/2016/10/serverflow-irrigate.jpg?w=640" alt="serverflow-irrigate" width="640" height="274" /></a> Server Node-Red Flow. This flow periodically polls the Watson Weather Data service, analyses the forecast and current conditions to decide if irrigation is required. If Yes, a 'waterGarden' command is sent to the device and a tweet issued.[/caption]

[caption id="attachment_2314" align="aligncenter" width="640"]<a href="https://willryle.files.wordpress.com/2016/10/serverflow-savetodb.png" target="_blank"><img class="wp-image-2314 size-large" src="https://willryle.files.wordpress.com/2016/10/serverflow-savetodb.png?w=640" alt="serverflow-savetodb" width="640" height="359" /></a> This flow receives data from the device using a Watson IoTF input node and saves it to the Cloudant database.[/caption]

<strong>To Do</strong>

Current development has a bench top proof of concept in operation. The cycle of command-irrigate-report is complete and two small development motors turn and the water valve opens and closes as the irrigation sequence runs. As an interim measure, ALIS uses the Twitter node (<a href="https://twitter.com/CognitiveALIS" target="_blank">CognitiveALIS</a>) to send 'irrigation starting' notifications from the server and 'irrigation complete' messages from the device The following remains to be implemented:
<ol>
 	<li>Construction and fitment of an irrigation boom to the device's central pillar</li>
 	<li>Re-wiring of the device's current motors and power supply to the motor controller such that they take commands from the irrigation sequence.</li>
 	<li>Construction of a suitable weatherproof housing for the Raspberry Pi and associated hardware, making provision for the Raspberry Pi's camera.</li>
 	<li>Provision and installation of a solar-powered power supply for the Raspberry Pi.</li>
 	<li>Installation and calibration of the water flow meter.</li>
 	<li>Populate the irrigation sequence complete report with live data.</li>
 	<li>Create mobile application to allow notifications to be received and control of irrigation sequence.</li>
 	<li>Complete the 'Irrigation Decider' server-side sub flow.</li>
</ol>
<strong>Possible Future Enhancements</strong>
<ol>
 	<li>Steering - the prototype will travel in a straight line only by fixing the steering wheels in the straight ahead position. A line-following mechanism, if implemented, would allow navigation around more complex shaped plots. This pre-supposes that an appropriate system can be found suitable for outdoor use.</li>
 	<li>Row Positioning - the prototype will use dead reckoning to determine the position of a row. (i.e. the length of time the motors run) A more accurate and easily configurable alternative would use marker pegs at the end of each row and a sonar distance measuring sensor. The device's arrival at a row would be determined when the sensor detected a marker peg within a given distance.</li>
 	<li>Local Weather Station - install a weather station on site to report local conditions. Use this data in conjunction with forecasts in the irrigation decider flow.</li>
 	<li>Add soil moisture sensor(s) - these will give accurate site-specific readings which can be included in the irrigation decider flow. This could take the form of a single moisture sensor attached to a probe on the device which is inserted into the soil to take readings at each row, or a static network of moisture sensors to give an overall indication of soil moisture at any one time.</li>
 	<li>Read water tank contents - The garden water supply is a 25,000 litre tank filled by catching rain water from the roof. Reading the available water supply by some means could also factor in to the irrigation decision.</li>
</ol>
