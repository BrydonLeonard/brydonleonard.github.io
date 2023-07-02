---
layout: post
title:  "Button Box: The rubber hits the road"
date:   2022-04-20
categories: racing coding electronics tyres
---

# The beginning

Back in the darkest depths of 2020's lockdowns, my partner and I watched Netflix's Drive To Survive series. It's a dramatised view into the world of Formula 1 that's driven a [huge uptick in viewership and engagement][f1 viewership increase] for a sport that had seen several years of decline. The downward trend was not least of all due to ex-Formula 1 CEO Bernie Ecclestone's [lack of enthusiasm in attracting a younger audience][bernie ecclestone interview], which seems like the opposite of the F1 group's new approach since its takeover by Liberty Media in 2016. The new owners have made a concerted effort to increase F1's social media presence, as well as the presence and visibility of the drivers themselves; one aspect of that push for increased visibility is Drive To Survive. Our household happily contributed two sets of eyes to the "viewership increase post Netflix series" statistic and we starting watching the 2020 formula 1 season just in time to see [Lewis Hamilton winning his seventh world title][lewis hamilton 7th title] live.

## The hobby on the horizon

I've played video games for most of my life. The intersection of that and my new motor racing hobby meant that sim racing was the natural next step. For a few months, YouTube's algorithms noticed my new interest, forgot entirely about my other hobbies, and presented my with almost exclusively sim racing content. I happily obliged and watched everything it served up. I tried racing with an Xbox controller for a while, but it's tough to race accurately with such the fiddly analog sticks. Finally, in late 2021, a new friend lent me his old sim racing wheel and pedals, kicking off my sim racing hobby in earnest.

## Tyres?

Racing simulators are essentially really realistic video games. To the casual observer, it may not be clear how much more there could *reall* be to capture that isn't present in games in (for example) the Need For Speed series. Real cars racing around affected by a huge number of factors such as the temperature of the car's tyres, turbulent air being thrown off of the cars ahead, and the weather. The developers of the simulators (sims) attempt to capture as many of these factors as possible.

One of the most important factors of a racing simulation is its *tyre model*; the component of the sim responsible for making the cars' tyres behave as closely to real tyres as possible. The reason it's as important as it is is because in real life, a cars tyres are all that connect it to the road. The car can be as powerful as you like, but without a set of tyres that's able to transfer that power from the engine to the road, all that power is being wasted.

### How is grip different to friction?

When discussing cars and their tyres, you'll often hear about *grip* and *traction*, but it might not be immediately obvious how those concepts related to *friction*, which also has *something* to do with surfaces sliding over each other. Let's start with a quick dive into friction.

#### What friction do?

When to objects are touching each other and slide in different directions, they will experience some reistance to that sliding motion. You can feel this right now by rubbing your hands together. Friction is the reason you have to keep pushing to keep your hands moving. If you stop applying a force, the frictional force will stop your hands from sliding past each other. 

Interestingly, that frictional force is the same reason your can rub your hands together to warm them up. The [law of conservation of energy](conservation of energy) tells us that energy can't just *disappear*, but we can change it from one type of energy to another. Friction turns movement (called kinetic energy) into heat (called thermal energy). The idea that friction turns kinetic to thermal energy is central to how car brakes and tyres work at all.

#### How friction do?

If you took physical science as a subject in school, you'll remember that [friction $$f$$ is the product of the normal force ($$N$$) and the coefficient of friction ($$ \mu $$)](friction wikipedia):

$$ f = \mu N $$

The normal force, $$N$$ is the amount of force pushing the two surfaces together. When you rub your hands together, this is how hard you push them into each other. From the equation above, we can see that increasing the normal force increases the friction. When rubbing your hands together, if you push them together harder, you'd notice that it gets more difficult to rub them; the friction has increased. The coefficient of friction, $$\mu$$ is a number that essentially represents how the stickiness of the connection between two surfaces. Two smooth pieces of glass sliding past each other would have a small value for $$\mu$$, while two pieces of sandpaper would have a much higher $$\mu$$. 

Something really interesting to note is that the friction equation contains no information about the size of the area that's making contact between the two surfaces. A really small piece of sandpaper sliding with a 5kg weight on it will experience exactly the same amount of friction as a much larger piece with 5kg spread over the whole area. 

#### Kinetic and static friction 

The last bit of friction information that'll be useful for a discussion of car tyres is the difference between *static* and *kinetic* friction. When two surfaces aren't yet moving, they can experience *more* frictional force than when they start moving. The graph below shows an example of the result. The horizontal axis shows the amount of force being applied to an object, while the vertical shows the amount of friction it experiences.

{:refdef: style="text-align: center;"}
![Friction diagram](/assets/images/button-box-beginning/friction_graph.svg)
{: refdef}

You'll have experienced thes phenomenon when trying to push something gently, but instead of it starting to slide slowly, it suddenly jolts forward as it starts moving. That jolt is because you have to push harder to get it moving than to keep it moving. 

### Why are tyres important?

#### Friction and tyres

Now that we've got a basic understanding of friction, let's talk about how a car's tyre's use friction to move the car forward. 

As I mentioned before, the car's tyres are all that connect it to the road, but it's not even the entire tyre that's making that connection at all times. At any moment, there's only a small piece of the tyre (at the bottom) that's actually resting on the road. That small area is called the *contact patch*. 

![Contact patch diagram](/assets/images/button-box-beginning/wheel_diagram.svg)

Cars move around and their tyres use friction to make it happen; given those two pieces of information, it would be understandable to assume that the tyres are using *kinetic* friction.  In reality, the car's tyres are using *static* friction to move the car forward. 

When the engine rotates the wheel, what it's effectively trying to do is *slide the contact patch backwards*. When it does that, the contact patch experiences friction, so instead of the contact patch just sliding around, it actually stays in the same place on the road and the wheel rotates away from it. As the wheel rotates, *something* has to move, and there are two choices: the ground or the car. It's almost always going to be easier to move the car than the ground itself, so the car starts to move forward. 

If the car accelerates or brakes too hard, the force pushing the contact patch to the side can exceed the maximum static friction and cause the patch to start sliding. Somewhat counterintuitively, when the car's wheels *stop* rotating, they start experiencing *kinetic* friction. The reason is that the bit of the tyre we really care about is the contact patch. When driving with the wheels rotating along the road as normal, the contact patch is stationary and pushes against the road. When the tyres start to slide, however, the contact patch itself is no longer stationary relative to the group; it is sliding over it. The switch from static to kinetic friction looks different depending on whether the driver is accelerating or braking. If a driver tries to accelerate too fast, the driven wheels (the ones with power from the engine) start rotating too fast for them to grip the group and they (and their contact patch) start skidding, which means that the car doesn't accelerate as fast as it could. When braking, on the other hand, the wheels are already rotating. If the driver tries to *slow down* too fast, the wheels will *stop* rotating while the car is still moving forward and the car won't be able to slow down quite as fast as it should.

#### Tyres in the real world

The friction equation we saw earlier is nice and tidy, but leaves out a lot of the unpredictability of the real world. Traction (or grip) is the actual friction between the tyre and road surface. One way that this plays out is that traction can be increased by increasing the size of a tyre's contact patch, despite friction not depending on its size at all.

One of the primary reasons for this is that road surfaces aren't perfectly smooth, so the *actual* coefficient of friction experienced by a tyre fluctuates as a car drives around. The tyre has some theoretical *maximum* coefficient of friction based on its own composite and the surface of the road, so by increasing the width of a tyre (and therefore the size of its contact patch), the tyre has more chance to make good contact with the road and achieve maximum friction.

#### Sharing the grip

We know now that accelerating or braking too hard can cause the tyres to stop gripping the road and start sliding. Things become even more interesting when we look at cornering. When you turn the steering wheel, the tyres are still doing the same thing: rotating and pushing the road backwards. When you're turning they just happen to be pushing in a direction that's slightly to the side of the car. Because it is the same set of tyres doing the same thing, if the car brakes and turns at the same time, those tyres then need to apply forces in two different directions (backwards to brake and sideways to turn) at the same time. Those two forces need to *share* the tyre's grip. 

If the forces required to both slow the driver down and turn the car around the corner are more than the tyre's maximum grip, then they will lose traction. Because kinetic friction is less than static, once that skidding starts, not only is the car not able to slow down as effectively, but the tyres aren't able to rotate the car so that it can get around the corner. In such a situation, the car is steering *less* than the driver wants it to, so it's called *understeering*. 

*Oversteer* is a similar problem that's most often experienced during acceleration instead of braking and results in the car rotating *more* than the driver wanted. As discussed previously, when a driver accelerates, if they attempt to put too much power through the wheels too fast, they'll start to skid. In rear-wheel-drive cars, where the driven wheels are at the back of the car, that skidding means that the back of the car has less grip than the front. If the car's momentum is pushing it to the side and there's less grip at the back, it means that the back of the car starts to swing *around* the front. That extra rotation means that the car is steering more than the driver wants, hence *over*steering.

![Drifting!](https://images.pexels.com/photos/274974/pexels-photo-274974.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1) 

### How do we get more grip?

Among other things, a tyre's grip is a function of its temperature. The tyre has an ideal temperature range within which it offers maximum grip. Below that, and the tyre isn't sticky enough, has lower grip, and leads to slower acceleration, braking, and cornering. Speeding up, slowing down, and going around corners are arguably three of the most important things in racing, so it stands to reason you'd want to do them effectively.

Above the ideal range, you can start to experience *blistering* and *graining*, where chunks of melted rubber break off of the tyre and occasionally get stuck back in the wrong place. Problems can also arise when there are large differences in temperature between the core and surface of a the tyres. 

- Racing slicks vs road tyres
- Brake temperatures

### Tyre wear
- Weight shift

- Weight shift and more wear on certain tyres
- Brakes and how they heat tyres
- Grip and how it differs from friction


### Types 
- 



[f1 viewership increase]: https://www.cnbc.com/2022/03/22/formula-1-2022-bahrain-grand-prix-was-espns-most-viewed-since-1995.html
[bernie ecclestone interview]: https://www.campaignasia.com/article/exclusive-f1-boss-bernie-ecclestone-on-his-billion-dollar-brand/392088
[lewis hamilton 7th title]: https://twitter.com/i/events/1327965210523676673?lang=en
[conservation of energy]: https://en.wikipedia.org/wiki/Conservation_of_energy
[friction wikipedia]: https://en.wikipedia.org/wiki/Friction#Dry_friction

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
