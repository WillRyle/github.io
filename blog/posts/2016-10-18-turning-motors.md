---
layout: post
title: Turning Motors
date: 2016-10-18 22:30
author: willryle
comments: true
categories: [Irrigator]
---
This is a follow-on post from the previous <a href="https://willryle.wordpress.com/2016/10/16/autonomous-irrigation/" target="_blank">Autonomous Irrigation</a> and describes in detail the steps taken to get the development motors turning.

<!--more-->

<strong>Motor Controller</strong>

The Pololu motor controller I finally settled on is capable of controlling two DC motors with an operational voltage of up to 24volts and a current of 12amps. The device's motors are 24v 8 amp units, so opting for a higher rated controller allows for the initial higher surge of power drawn when the motors are first powered on.

The controller came as a bare board but with pins that matched the Arduino's that had to be soldered in place. Additionally, the connectors for power supply and the two motors were soldered into their positions on one end of the board. With that done, it was possible to attach the controller to the Arduino as a HAT, the controller's pins inserted into the matching Arduino sockets.

The next stage was wiring and connecting the power supply and motors to the controller. A simple 12v DC supply was obtained for this purpose and a series of adapters finished in positive and negative terminals which were connected to the power terminals on the controller. As a dual motor controller, the Pololu designates one as 'M1' and the other 'M2', each having two connections. Each motor has two terminals and these were simply wired to the two connections of a motor input.

The Arduino was connected to the Raspberry Pi via a USB cable. Opening the Arduino Sketch tool on the Raspberry Pi prompts for the serial port to use. The controller came with sample sketches demonstrating the use of the Pololu's API, one of which was loaded and deployed to the Arduino. The motors turned accordingly, confirming that the configuration was correct.

[caption id="attachment_2337" align="aligncenter" width="640"]<a href="https://willryle.files.wordpress.com/2016/10/wp_20161018_21_32_15_pro.jpg" target="_blank"><img class="wp-image-2337 size-large" src="https://willryle.files.wordpress.com/2016/10/wp_20161018_21_32_15_pro.jpg?w=640" alt="The desktop development system" width="640" height="361" /></a> The desktop development system - Raspberry Pi, breadboard, 12 volt DC motors, water valve, Arduino wearing a motor controller HAT and 4 channel relay board[/caption]

<strong>Arduino Sketch</strong>

For this application, each motor needs to run in a given direction at a given power setting for a specified length of time. The Pololu API distinguishes direction of rotation based on the sign of the power setting. The power setting is a value between 400 and -400 with 0 being off. I therefore needed to supply two numbers to the sketch, a power/direction integer value and a milliseconds duration.

The Arduino <a href="https://www.arduino.cc/en/Reference/HomePage" target="_blank">language</a> is based on C, but has additional features one of which is a serialEvent() method. This is called when data is available on the serial port and is key to setting the motor speed from the Raspberry Pi. The data is passed in as a pair of comma-separated numbers, where the first value is the motor speed and the second the duration in milliseconds.  The serialEvent() method parses the values from the serial data and calls a startMotor() method. A loop idles until the required milliseconds have elapsed when the motors are stopped by setting the power to zero. The complete sketch is shown below:
<blockquote>
<pre>#include "DualVNH5019MotorShield.h"

DualVNH5019MotorShield md;
const int MaxChars = 3;
char strValue[MaxChars + 1];
int motorSpeed = 0;
int index = 0;
unsigned long startTime;
unsigned long runTime = 10000;
boolean motorsRunning = false;

void stopIfFault()
{
  if (md.getM1Fault())
  {
     Serial.println("M1 fault");
     while(1);
  }
  if (md.getM2Fault())
  {
     Serial.println("M2 fault");
     while(1);
  }
}

void setup()
{
   Serial.begin(115200);
   Serial.println("Dual VNH5019 Motor Shield");
   md.init();
}

void loop()
{
   if (motorsRunning &amp;&amp; millis() &gt; (startTime + runTime))
   {
      stopMotors();
   }
}

void startMotors()
{
   startTime = millis();  // time motors started
   motorsRunning = true;
   setMotorSpeed();
}

void stopMotors()
{
   motorSpeed = 0;
   setMotorSpeed();
   motorsRunning = false;
}

void showSpeed()
{
   char strSpeed[5];
   sprintf(strSpeed, "%d", motorSpeed);
   Serial.println("motor speed: " || strSpeed);
}

void setMotorSpeed()
{
   //showSpeed();
   md.setM2Speed(motorSpeed);
   md.setM1Speed(motorSpeed);
   stopIfFault();
}

void serialEvent()
{
   while (Serial.available() &gt; 0)
   {
      // expecting motor speed and runtime in millisecs in csv format e.g.
      // 90,5000 = run forward at power setting 90 for 5 seconds
      motorSpeed = Serial.parseInt();
      runTime = Serial.parseInt();
      startMotors();
   }
}</pre>
</blockquote>
<strong>Python Script</strong>

The Raspberry Pi controls the motors via an 'Irrigation Sequence' written in Python. An irrigation sequence is simply moving the device to the first row of vegetables, turning the water on for a period of time then turning the water off and moving to the next row. The distance between each row is a measure of how long the motors run at a given power setting and will initially be determined by experimentation. The Python script below has demonstration values only, in practice the number of rows will be greater and the watering time around 10 minutes per row.

The moveIrrigator() method creates a command string using the speed and duration parameters then writes the command to the serial port. This triggers the serialEvent() method in the Arduino sketch thereby setting the motor speed. The water valve is turned on and off by setting the GPIO pin to a low or high value using appropriately named methods. A delay in moveIrrigator() of the same length as the travel duration prevents control returning to turn the water on while the device is still moving. Water on and off commands are separated by a delay equal to the watering time.

The final step is returning the device to the starting point ready for the next run. This is achieved by supplying a negative speed setting and a return duration that totals the motor time moving down the rows. Whether this dead reckoning method of movement proves to be practical remains to be seen. It is hoped to be adequate for the prototype, but will be unknown until an attempt is made to define an irrigation sequence on site rather than on the desktop. The demonstration Python script is shown below.
<blockquote>
<pre>import serial
import time
import RPi.GPIO as GPIO

ser = serial.Serial('/dev/ttyUSB0', 115200)
wateringTime = 10
irrigatorSpeed = 200
returnTime = 0
waterValveRelayPin = 23

# initialise GPIO
GPIO.setmode(GPIO.BCM)
GPIO.setwarnings(False)
GPIO.setup(waterValveRelayPin, GPIO.OUT)

def moveIrrigator(speed, duration):
    # arduino expects time in milliseconds
    command = str(speed) + ',' + str(duration * 1000)
    print('Moving: ' + command)
    ser.write(command)
    # sleep while the irrigator is moving so that the water
    # isn't turned on/off at the wrong times
    time.sleep(duration)

def waterOn() :
    time.sleep(1)
    print('Water on')
    GPIO.output(waterValveRelayPin, GPIO.LOW)

def waterOff() :
    print('Water off')
    GPIO.output(waterValveRelayPin, GPIO.HIGH)
    time.sleep(1)

# irrigation sequence
# advance to first row
moveIrrigator(speed=irrigatorSpeed, duration=5)
returnTime += 5
# turn water on
waterOn()
time.sleep(wateringTime)
waterOff()

# row 2
moveIrrigator(speed=irrigatorSpeed, duration=3)
returnTime += 3
# turn water on
waterOn()
time.sleep(wateringTime)
waterOff()

# row 3
moveIrrigator(speed=irrigatorSpeed, duration=3)
returnTime += 3
# turn water on
waterOn()
time.sleep(wateringTime)
waterOff()

# row 4
moveIrrigator(speed=irrigatorSpeed, duration=4)
returnTime += 4
# turn water on
waterOn()
time.sleep(wateringTime)
waterOff()

#return to start point - reverse
print('Returning to start point')
ser.write(str(-irrigatorSpeed) + ',' + str(returnTime))
moveIrrigator(speed= -irrigatorSpeed, duration=returnTime)
GPIO.cleanup()

</pre>
</blockquote>
The next post will describe creating the node-red flows on the Raspberry Pi.

&nbsp;
