---
layout: post
title: "An intro to aviation - The gyroscopic flight instruments"
date: 2024-02-04
categories: aviation
---

The post follows on from [An intro to aviation - The pitot-static flight instruments](https://brydonleonard.github.io/aviation/2024/01/21/aviation-pitot-static-instruments.html). This time, I'll take a look at the last three instruments of the sixpack; the gyroscopic instruments.
## Gyroscopes

Gyroscopes exhibit two phenomena that we leverage for the gyroscopic flight instruments; rigidity and precession. ***Rigidity*** describes a spinning gyroscope's resistance to its orientation being changed. The video below shows a gyroscope inside a *gimbal*. The gimbal rings are able to rotate, but the spinning disk in the centre stays stationary due to rigidity.

![](/assets/images/2024-02-04-aviation-gyroscopic-instruments/gyroscope_operation.gif)

Slightly confusingly, when a force is applied to one of the edges of a gyroscope, instead of the gyroscope tilting towards the side that was pushed, it leans down 90 degrees ahead (in the direction of spin) of that spot. That's called ***precession***.

The ***spin axis*** of a gyroscope is the one that it spins *around*. A spinning top, for example, spins around the vertical axis, while a wheel spins around the horizonal axis. 

## Axes of rotation

Before we cover how gyroscopes are used to help pilots understand their plane's orientation, it'll be useful to cover the different ways that planes rotate and turn.
- When a plane leans to the side, it is called ***roll***. The angle that the plane is rolling is called the ***bank angle***.
- When a plane rotates like a car would, it's called ***yaw***.
- When a plane's nose moves up or down, it's called ***pitch***.
- Finally, rather than changing the plane's angles of rotation, ***turning*** is changing the direction that the plane is flying. Pilots use a combination of roll, pitch, and yaw to get their planes to turn.

| Roll | Yaw | Pitch |
| ---- | ---- | ---- |
| ![](/assets/images/2024-02-04-aviation-gyroscopic-instruments/roll.gif) | ![](/assets/images/2024-02-04-aviation-gyroscopic-instruments/yaw.gif) | ![](/assets/images/2024-02-04-aviation-gyroscopic-instruments/pitch.gif) |

**Turn:**
![](/assets/images/2024-02-04-aviation-gyroscopic-instruments/turn.png)

## The turn coordinator

A ***rate one*** or ***standard rate*** turn is one where the plane turns at 3 degrees per second, which would lead to a full 360 degrees in two minutes. When a plane turns, the turn coordinator shows the direction of turn by lowering the wing of the little plane on the instrument. When the lowered wing reaches the L/R mark, the plane is turning at the standard rate. The instrument can be confusing initially because despite having the little plane roll, the instrument *doesn't* show how much the plane itself is rolling; it's only showing the rate of turn.

![](/assets/images/2024-02-04-aviation-gyroscopic-instruments/turn_coordinator.png)[^1] 

When a pilot rolls a plane to start turning, the plane also naturally starts yawing[^2]. When that happens, the ball suspended in liquid in the turn coordinator moves to the opposite side of the yaw (the ball moves left when yawing right). The pilots watch that ball to balance their roll, pitch, and yaw inputs and *coordinate* their turns. Coordinated turns are much more comfortable for passengers because they don't feel like they're being pushed to the side. When done right, passengers that aren't looking outside may not even know that the plane is turning.

### The turn coordinator's gyro

The gyroscope in a turn indicator is fixed and always rotates in the same direction as the plane's wheels would, were it rolling forward on the ground. When the plane turns to the side, the gyroscope's housing forces it to stay aligned with the plane. That rotation creates forces that, through gyroscopic precession, push the top of bottom of the gyroscope to the side, causing it to lean. That lean can be measured and used to calculate the *rate* of turn, hence these gyroscopes being called ***rate gyros.***

## The attitude indicator

***Attitude*** is the name for the combination of both pitch and roll. As such, the attitude indicator shows both the plane's pitch and its roll. As the plane rolls, the yellow triangle indicating the plane's angle of bank moves to the side. The bank angles are marked along the top of the instrument, but become less frequent after 30 degrees partly, because a turn with more than 30 degrees of bank is known as a *steep* turn[^3] and wouldn't be performed under normal circumstances [^4].

![](/assets/images/2024-02-04-aviation-gyroscopic-instruments/attitude_indicator.png)

Similarly to roll, as the plane pitches, the yellow dot in the centre moves vertically to indicate the plane's pitch angle in degrees.

### The attitude indicator's gyro

The attitude indicator uses an ***earth gyro***, which spins around the vertical axis (like a spinning top) and always has that vertical axis pointed towards the earth (hence the name). Since the vertical axis always points down, as the plane pitches and rolls, the gyro's position *relative to the plane* changes and is measured to determine the plane's attitude.

## The heading indicator

The final gyro-driven instrument is the heading indicator. A plane's ***heading*** is the direction it's flying, measured in degrees from magnetic north. Planes also have compasses, which also show the plane's heading but, because they're magnetic, have some key weaknesses that make the gyro-driven heading indicator useful:
- Compasses are subject to *deviation*, which is an error caused by the magnetic fields of the components of the plane itself. Pilots have to correct for those when reading the compass.
- The compass balances vertically on a pivot point inside its housing. Near the poles, the magnetic field starts to become more vertical and causes the compass to tilt and drag along its housing.
- The centre of gravity of the compass itself is off-centre, so as the plane accelerates, the compass rotates slightly.
- As the plane turns, the compass' inertia will cause it to lag behind the turn and read wrong until it stabilises in the new heading.

Despite all of those downsides, compasses are still necessary since the heading indicator's gyro will drift over time and start showing incorrect readings. To account for that, pilots reset the heading indicator to match the compass every 15 minutes.

![](/assets/images/2024-02-04-aviation-gyroscopic-instruments/heading_indicator.png) [^5]

### The heading indicator's gyro

The heading indicator's gyro rotates around the horizonal axis; the same one as the turn coordinator's rate gyro. Unlike the rate gyro, however, the heading indicator's gyro can rotate around the vertical axis, which allow it to stay spinning in the same direction even as the plane turns. As a result, the position of the gyro changes relative to the plane, which can be used to determine the plane's heading.

## Next time

Now that we've covered the basic flight instruments, my next few posts will hop out of the plane and take a look at how pilots actually control them.

## Footnotes

[^1]: CC BY-SA 3.0 DEED image shared unmodified from [Mysid on Wikipedia](https://commons.wikimedia.org/wiki/File:Turn_coordinator_-_coordinated.svg)
[^2]: I'll talk about coordinated turns more in a future post about the dynamics of flight.
[^3]: During flight training, pilots practice steep turns at 45 degrees. 
[^4]: 30 degrees is also noteworthy because when in a holding pattern or flying orbits, pilots will either perform a standard rate turn or turn with 30 degrees of bank, whichever requires *less* bank angle.
[^5]: GFDL image shared unmodified from [Oona Räisänen on Wikipedia](https://en.m.wikipedia.org/wiki/File:Heading_indicator.svg)

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Faviation%2F2024%2F02%2F04%2Faviation-gyro-instruments.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)