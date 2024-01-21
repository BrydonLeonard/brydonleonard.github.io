---
layout: post
title:  "What the Kotlin? Again!"
date:   2024-01-14
categories: learning kotlin
---

# An intro to aviation - The pitot-static flight instruments

## Intro
I got my private pilot's licence a few years ago. A big part of the fun for me was getting an insight into what's going on when I fly commercially. This series of blog posts will take a little look behind the curtain and hopefully make aviation a little more approachable. The first few posts are going to focus on aeroplanes themselves, as well as the instruments that pilots use to fly.

## The types of instruments

There's a set of six basic flight instruments in most planes known as a "six pack" [^1]. They're like the speedometer in a car and help the pilot understand what it is that their flying machine is doing.

![Sixpack.png](/assets/images/2024-01-21-aviation-pitot-static-instruments/Sixpack.png)

### The pitot-static system

The pitot-static system works by comparing the static and pitot *air pressures*, each measured in hectopascals (hPa) (though the images in this blogpost are from American planes, which use inches of mercury).

***Static*** air pressure is the pressure you feel when not moving at all. It's caused by all the air in the atmosphere above you pushing downwards and it decreases when you're in places with higher elevation. It's measured via little holes called static ports on the outside of the plane.

***Dynamic*** air pressure is the *extra* force you feel when you hold your hand out the window of a moving car. When you're feeling that force, however, there's static pressure being applied to your hand as well. As such, if we try measure dynamic pressure by seeing how hard your hand is pushed back, we end up measuring the *sum* of dynamic and static pressure. In aviation, that's called the ***pitot pressure*** because it's measured via a ***pitot tube***, which sticks out from the plane and into the air stream.

![PitotTube.png](/assets/images/2024-01-21-aviation-pitot-static-instruments/PitotTube.png)[^5]

## The airspeed indicator

![ASI.png](/assets/images/2024-01-21-aviation-pitot-static-instruments/ASI.png)[^6]

Dynamic pressure and airspeed are closely related; if you know one, you can calculate the other. The airspeed indicator (ASI) uses that fact to tell pilots how fast the plane is moving through the air [^2] (measured in [knots](https://en.wikipedia.org/wiki/Knot_(unit))). The ASI finds the dynamic pressure effectively by subtracting static pressure from pitot pressure.

It's worth noting that a plane's airspeed isn't necessarily the same as the speed it's moving over the ground. If a plane flies at 100 knots in the same direction as the wind, which is blowing at 10 knots, its speed over the ground would be 110 knots.

## The altimeter

![Altimeter.png](/assets/images/2024-01-21-aviation-pitot-static-instruments/Altimeter.png)

The altimeter tells pilots how high a plane is (in feet) by measuring the static pressure outside the plane. As altitude increases, the static air pressure [decreases at a predictable rate](https://en.wikipedia.org/w/index.php?title=Atmospheric_pressure#Altitude_variation). The altimeter uses that rate and the known pressure at sea level to calculate the plane's altitude. Because air pressure can change, the altimeter has a knob that pilots turn to set the *reference* pressure (the pressure where the altimeter would read zero) by adjusting a setting called the altimeter's ***subscale***.

Before taking off from an airport, pilots speak to air traffic control to get the reference pressure that will make the altimeter show their ***altitude*** (height above mean sea level). That number is called the _QNH_[^3]. Setting the reference pressure to QNH while on the ground at [Morningstar airfield in Cape Town, will cause the altimeter to show 200 feet](https://morningstarflyingclub.co.za/airfield-info/), whereas at OR Tambo International (and with ORT's QNH), it would show 5,558 feet[^4].

### Flight levels

Air pressure is a weather phenomenon that, just like the wind or temperature, can vary even between places at the same altitude. As a result, when a plane flies long distances, it might fly past planes that took off from different airfields with different QNHs. That's a dangerous situation, since their altimeters would disagree and knowing nearby planes' altitudes is pretty useful in not flying into them.

To solve that issue, once a plane climbs through an altitude called the ***transition layer*** (which varies based on airport and weather conditions) the pilots set their altimeter's subscale to the ***standard pressure*** of 1013 hPa. Because everyone does this, all planes agree about their altitudes and can avoid each other.

Because altimeter readings aren't *really* showing the plane's altitude once the sub-scale is set to standard pressure, the altimeter readings are called ***flight levels*** instead. Flight levels are 1/100th of the altimeter reading, so when the altimeter reads 18000 feet, the plane is at flight level 180 (FL180).

## The vertical speed indicator

![VSI.png](/assets/images/2024-01-21-aviation-pitot-static-instruments/VSI.png)[^7]

Like the altimeter, the vertical speed indicator (VSI) uses static pressure. This time, however, it uses measures how fast the static pressure is *changing* by find the difference between the current and delayed pressure. The display of the VSI uses that rate of change to show the plane's rate of climb or descent in feet per minute. 

The VSI may not seem useful, given that the altimeter's needle moving already shows that you're changing altitude. It is, however a much easier way to ensure that you're maintaining your altitude during manoeuvre than watching the altimeter needle for tiny movements.

## Footnotes

[^1] Today, physical six packs are actually largely being replaced by electronic flight instrument systems (EFIS) like [the Garmin G3X](https://www.garmin.com/en-US/p/166058). The information they display is, however, largely the same and is gathered in the same ways.

[^2] Because of variation in temperature, static pressure, and the position of the pitot tube (which might not be pointing straight into the airflow), the indicated airspeed (IAS) can be wrong. The true airspeed (TAS) is the actual speed of the plane through the air after the errors are corrected.

[^3] QNH (and other Q-codes) aren't abbreviations. They're a standard set of codes that were invented in the days of morse codehttps://en.wikipedia.org/wiki/Q_code. They've seen continued use and expansion to today as useful shorthand in radio communication.

[^4] Another interesting reference pressure is an airfield's ***QFE***, which is the reference pressure that would make the altimeter show zero on the ground. The reference pressure that a pilot sets in the altimeter is essentially the "bottom" of the altimeter's scale (since when the static pressure increases all the way to the reference pressure the altimeter shows zero). As such, QFE is actually the same thing as the ambient pressure at the airfield. QNH is almost always higher than QFE since setting QNH makes the "bottom" of the altimeter's scale mean sea level, where the pressure is higher. It can be a confusing concept when thinking about airports at high altitude, since there isn't actually a sea below them, but one way to think of it is that if the ground disappeared and the aeroplane started falling, its altimeter would hit zero as it reached the sea below.

[^5] CC BY-SA 4.0 DEED image shared unmodified from [Cjp24 on wikimedia commons](https://commons.wikimedia.org/wiki/File:Cessna_172_Skyhawk_II_-_Pitot_tube.jpg)

[^6] CC BY-SA 3.0 DEED Image shared unmodified from [Mysid on wikipedia](https://en.wikipedia.org/wiki/File:Airspeed_indicator.svg)

[^7] CC BY-SA 3.0 DEED image shared unmodified from [The High Fin Sperm Whale on wikipedia](https://en.wikipedia.org/wiki/File:Vertical_speed_indicator.PNG)