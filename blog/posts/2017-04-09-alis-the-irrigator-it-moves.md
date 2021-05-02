---
layout: post
title: ALIS the Irrigator - It Moves
date: 2017-04-09 20:28
author: willryle
comments: true
categories: [Irrigator]
---
Something of a project milestone today for <a href="https://willryle.wordpress.com/2016/10/09/alis-the-irrigator/" target="_blank">ALIS</a>, our home-made irrigation system. After much research, the components required to take ALIS from bench top to scooter were assembled and today saw the necessary wiring completed.

<!--more-->

[caption id="attachment_2418" align="alignleft" width="300"]<a href="https://willryle.files.wordpress.com/2017/04/wp_20170409_13_30_40_pro.jpg"><img class="size-medium wp-image-2418" src="https://willryle.files.wordpress.com/2017/04/wp_20170409_13_30_40_pro.jpg?w=300" alt="" width="300" height="169" /></a> First motor trial of the irrigator[/caption]

The approach taken was to bypass the scooter's electrical system altogether, just using the batteries and motors.

The two 12 volt batteries were wired in series through a fuse box to give 24 volt outputs, each protected from overloading by standard car fuses. The scooter's motors were wired into the motor controller in the same way as the bench top proof of concept, but the controller itself used the 24 volt supply. The water valve is 12 volt, so that will be wired from a single battery.

The Raspberry Pi was powered through a DC step-down board which took 24 volts input and was adjusted to give the 5 volts the Pi uses. This aspect of the system is critical as too great a voltage could damage the Pi - possibly permanently - and too low a voltage has the Pi complaining about a lack of power. Oddly, with the step-down board indicating 5 volts output, the Pi still showed a 'low power' status. The solution was to increase the voltage slightly to 5.2 - still within the Pi's power range - which reduced the low power complaint somewhat.

With the wiring complete and the scooter on blocks so that the wheels were clear of the ground, I ran a tentative test script to turn the motors and check that they each turned in the right direction. This first run used a low power setting which I gradually increased until I found a power level that wouldn't have the scooter accelerating too quickly - I had no idea how the riderless scooter would track or how fast and far the test irrigation sequence would take it and had visions of it hurtling off and careering into something solid. As it happens all went well. I took the device outside and set it on a safe heading then used the server-side irrigation interface to initiate an irrigation run of four 'rows'. A second later it set off with a little jolt and motored for a few seconds before stopping at the first row, then carried on following the irrigation sequence precisely. The return to start point had it come to a rest about a metre short, likely a consequence of the scooter rolling slightly after each of the four stops. There is a relatively simple fix for this. Currently the motor power is applied at full immediately and stopped equally abruptly. The change is to ramp the power up and down incrementally, which will likely give an 'engine braking' effect when stopping and so give better control.

The next tasks will involve 'hardening' the electronic installation - boxing the components in something waterproof rather than having them balanced on a piece of wood. Then there's the new irrigation boom to fabricate and some bodywork to design and make. The next test will be after the new boom is fitted as I want to see how dragging the retractable hose and carrying the weight of the boom affects the scooter's tracking and motoring.

The video below shows the motor test, with the scooter pausing at each row before returning to the start point.
[youtube https://www.youtube.com/watch?v=tYKOTiYgPJg?rel=0]
