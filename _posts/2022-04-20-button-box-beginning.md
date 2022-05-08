---
layout: post
title:  "Button Box: The beginning"
date:   2022-04-20
categories: racing coding electronics
---

# The beginning

Back in the darkest depths of 2020's lockdowns, my partner and I watched Netflix's Drive To Survive series. It's a dramatised view into the world of Formula 1 that's driven a [huge uptick in viewership and engagement][f1 viewership increase] for a sport that had seen several years of decline. The downward trend was not least of all due to ex-Formula 1 CEO Bernie Ecclestone's [lack of enthusiasm in attracting a younger audience][bernie ecclestone interview], which seems like the opposite of the F1 group's new approach since its takeover by Liberty Media in 2016. The new owners have made a concerted effort to increase F1's social media presence, as well as the presence and visibility of the drivers themselves; one aspect of that push for increased visibility is Drive To Survive. Our household happily contributed two sets of eyes to the "viewership increase post Netflix series" statistic and we starting watching the 2020 formula 1 season just in time to see [Lewis Hamilton winning his seventh world title][lewis hamilton 7th title] live.

## The hobby on the horizon

I've played video games for most of my life. The intersection of that and my new motor racing hobby meant that sim racing was the natural next step. For a few months, YouTube's algorithms noticed my new interest, forgot entirely about my other hobbies, and presented my with almost exclusively sim racing content. I happily obliged and watched everything it served up. I tried racing with an Xbox controller for a while, but it's tough to race accurately with such the fiddly analog sticks. Finally, in late 2021, a new friend lent me his old sim racing wheel and pedals, kicking off my sim racing hobby in earnest.

## Sim racing?

Racing simulators are essentially really realistic video games. Real cars racing around affected by a huge number of factors such as the temperature of the car's tyres, turbulent air being thrown off of the cars ahead, and the weather. The developers of the simulators (sims) attempt to capture as many of these factors as possible. As an example, let's look at tyre and brake temperatures. 

As cars race around a circuit, all that connects them to the circuit are their tyres. A car can be as powerful as you like, but without some way to output that power effectively, the power is going to be wasted. As a car accelerates, the tyres are applying a force backwards and pushing against the road, when the car brakes, the opposite happens and the tyres apply a forward force against the road. If the car attempts to brake or accelerate too hard, the force being exerted horizontally by the tyres exceeds the tyres' maximum grip levels and they start to slide across the road instead of gripping and pushing. When accelerating, that would look like wheel spin, when braking, the wheels lock up and stop turning while the car continues to slide.

Things become even more interesting when we look at cornering. When you turn the steering wheel, the same the tyres turn the car using the same mechanics as when accelerating: it applies a force in the opposite direction that the car should move. When turning left, the tyres try to push the ground to the right. However, because that tyre surface that's pushing the car to the side is the *same* one that is used for braking; if the car brakes and turns at the same time, the two need to *share* the tyre's grip. Given that the tyre's grip represents the maximum possible energy that can be transferred to the ground, it follows that a racing driver would want it to be as high as possible. 

Among other things, a tyre's grip is a function of its temperature. The tyre has an ideal temperature range within which it offers maximum grip. Below that, and the tyre isn't sticky enough, has lower grip, and leads to slower acceleration, braking, and cornering. Above the ideal range, you can start to experience blistering and graining, where chunks of melted rubber break off of the tyre and occasionally get stuck back in the wrong place.

[f1 viewership increase]: https://www.cnbc.com/2022/03/22/formula-1-2022-bahrain-grand-prix-was-espns-most-viewed-since-1995.html
[bernie ecclestone interview]: https://www.campaignasia.com/article/exclusive-f1-boss-bernie-ecclestone-on-his-billion-dollar-brand/392088
[lewis hamilton 7th title]: https://twitter.com/i/events/1327965210523676673?lang=en



- Sim racing is latest hobby
- Many buttons to press - Explains the functionality
- - Brake bias - Explain how it's used in cornering
- - Engine maps
- - ABS/TC - Why GT3 has these at all
- Not enough buttons on my wheel, but using keys on keyboard is boring
- Want a button box
- 
- What I wanted
- Box with buttons
- Some knobs
- Some nice clicky switches
- Would be nice to be able to use for flight sims too
- 
- The end product
- - Picture
- - What the buttons do
- 
- Parts of the series
- Fusion 360 and 3D modelling
- - Show model
- - Parametric modelling
- - Link to channel
- Electronics
- - Types of switches (momentary, DPDT, etc)
- - Encoders
- - Rotary switch
- - Arduino
- - Pull-up/-down resistors
- - Diodes
- - Button matrix
- - Multiplexers
- - Shift registers
- - Laying things out (veroboard)
- - Prototyping (breadboard)
- Code
- - Debouncing
- - Momentary switches and actions in games
- - PlatformIO
- - Arduino libraries
- - Controlling pins
- - Shift register management
- Next steps
- - Test it out in a flight sim
