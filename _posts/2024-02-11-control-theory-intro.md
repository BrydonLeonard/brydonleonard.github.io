---
layout: post
title: "Control theory - An introduction"
date: 2024-02-11
categories: controlTheory
---

Imagine a system that has some inputs that you control and outputs that are _influenced_ by that input, but may also depend on some other factors that are out of your control. There's a whole field of study around those problems called ***control theory***. This post takes a look at some of control theory's core concepts and the interesting ways that they can be applied.
## A toy example

Let's start by imagining a hypothetical system that has a queue of maths equations to be computed. Every minute, the processor pulls between 1 and 20 equations from the queue and computes them (the number varies because more complex equations take longer to process). Due to technical constraints, if the queue has more than 100 items at any time, the whole system stops working. 

![maths solver](/assets/images/2024-02-11-control-theory-intro/maths_solver_flowchart.PNG)[^1]

Given that the system processes an average of 10 items every minute, a naive approach would be to add 10 items to the queue every minute. Here's a plot of the size of the queue over 500 minutes in a simulation that uses that approach:

![avg input explosion](/assets/images/2024-02-11-control-theory-intro/AvgOutputController.gif)

Because we didn't take the system's actual output into account, a few slow minutes resulted in the queue exploding and the system falling over! Instead of just adding 10 items every minute, let's focus on keeping the queue at a consistent length instead; we'll pick 50 as our desired queue length. Every minute, we'll check the number of elements in the queue and, if we have fewer than 50, we'll add however many we need. 

![P Controller](/assets/images/2024-02-11-control-theory-intro/PController.gif)

Much better. The processor always has enough work to do and we aren't at any risk of the queue exploding on us.  
## Feedback

The software that controls the input to systems like the one above is called a ***controller***. ***Control theory*** is the field of study that focuses on designing those controllers, of which there are two broad categories:
- ***Feed-forward*** controllers use a plan formulated ahead of time that doesn't need any information about the system's current behaviour. The "always input the known average output" approach from above is an example of a feed-forward controller.
- ***Feedback*** controllers observe the system's output and use those observations to inform their behaviour. The "add back up to 50" approach above is a feedback controller.

While systems that use feedback controllers tend to be slightly more complex (given that they have to measure the system's output and pass it back to the beginning of the process) the controllers themselves are much simpler. Because feed-forward controllers can't adapt on the fly, they need to account for all of the system's behaviours ahead of time, which requires a much deeper understanding of the dynamics of the system than a feedback controller's adaptable approach.
## Systems

Before going any further, let's define some the other terms we use to describe control systems:
 - The ***input*** is some configurable value that the controller can change
 - The ***output*** is a value whose value is measurable and influenced by the input (though it may also be affected by things outside of your control)
 - The ***set point*** is the desired output value for the system (50 in our earlier example)
 - The ***error*** is the difference between the system's current output and the set point

It's useful to note that the input and output of the control system aren't necessarily the same as the inputs and outputs of the underlying system. We saw that in our maths equation solver; the input and output of the equation solver were maths equations and answers, while the inputs and outputs of our controller were the number of equations to add to the queue and the size of the queue itself. 

Here are a few more examples of different problems that we could solve with control systems, as well as their setpoints, inputs, and outputs:

| Control system | Setpoint | Input | Output |
|---|---|---|---|
| Heating a pot of water to a 75 degrees | 75 degrees | The stove temperature | The water temperature |
| Dynamically resizing a cache to achieve a 75% hit rate | 75% | The cache size | The cache hit-rate |
| Pitching a quadcopter forward to 30 degrees | 30 degrees | The speed of the four motors | The pitch angle of the quadcopter |
| Keeping a video game running at 60 FPS | 60 FPS | The game's render resolution | The frame rate |

### Some limitations

Notice how all of the examples above had exactly one value as their set point; that's always the case for control systems. As a result, there are some limitations to the types of problems that they can solve:
- A control system can't have a *range* of acceptable values as a setpoint
- Control systems don't attempt to *optimise* anything; they just try to hit the setpoint

## PID controllers

One of the most common types of controller is the Proportional-Integral-Derivative (PID) controller. The only information that PID controllers need is the magnitude and direction of the output's error, from which they determine the input that they'll provide to the system.
### Proportional controllers

Before we jump all the way to a full PID controller, let's look at the simpler proportional (P) controller, which you've already encountered; the "add back to 50" controller from our maths equation solver was a proportional controller! The number of items we added to the queue was *proportional* to the magnitude of the error, hence its name. In our toy example, adding back exactly 10 items when the error was 10 worked out, but in a more complex system, the relationship between error and input may not be 1:1. As such, we can multiply the error by some number to account for that relationship. That number is the coefficient of the proportional term, \\(k_p\\). 

The input, \\(u\\) to the system at time \\(t\\), given the latest error, \\(e_t\\) can be written as

$$
u_t = k_p e_t
$$

### PI controllers

In the P controller simulation, you'll have noticed that the output never _quite_ reaches the setpoint of 50. That's a phenomenon known as *proportional droop*. One way to solve it is to track the error over multiple steps; if it's consistently too low or high, we can nudge the output towards the setpoint over time.

We achieve that by adding up the error at each step and using that error sum in our controller: \\(\sum_{\tau=0}^{t}e_\tau\\) [^2]. That error sum is called the ***integral term***. If the error is consistently positive or negative, the integral term's influence will increase and move the output towards the setpoint. As with the proportional term, we want to be able to control the weighting of the integral term, so we give it its own coefficient, \\(k_i\\). Here's the full PI controller equation: 

$$
u_t = k_pe_t + k_i\sum_{\tau = 0}^{t}e_\tau
$$

With the addition of the integral term, our controller is no longer droopy!

![PI controller](/assets/images/2024-02-11-control-theory-intro/PIController.gif)

### PID controllers

In many cases, a PI controller is perfectly sufficient. If, however, you find that your controller's a little too sluggish and takes too long to react to changes in the setpoint, you can add the ***derivative*** term. Where P tracks the current error and I tracks the historical error, D tracks the _change_ in error since the previous step: \\(e_t - e_{t-1}\\)[^3]. If the error suddenly changes, the derivative term will account for that and make the controller react quickly to counteract that change. 

$$
u_t = k_pe_t + k_i\sum_{\tau = 0}^{t}e_\tau + k_d(e_t - e_{t-1})
$$

If you are going to include a derivative term in your controller, it's important to find coefficients for all three terms that balance speed and stability; without that, derivative terms can introduce significant instability. A small change in the error can cause the derivative term to kick in and overcorrect in the other direction. In the next time step, the derivative term could react to *its own* overcorrection and overcorrect in *the other direction*. 

Our queue-managing controller from earlier is pretty quick even without a derivative term, so the simulation doesn't show much of a change from the PI controller, but here it is anyway!

![PID controller](/assets/images/2024-02-11-control-theory-intro/PIDController.gif)

## What's next?

Control theory is an interesting field with much more depth than I've had time to cover here. In a future post, I'll revisit the topic to look at more applications of the approach, how we optimise those coefficients, and to play with more fun simulations!


## Footnotes

[^1]: Are the animations entirely unnecessary? Yes. Did I just start playing with [Manim](https://docs.manim.community/) and find any excuse to use it? Also yes.

[^2]: The integral term is usually represented as an integral (so \\(\int_{0}^{t}e_\tau d(\tau))\\), but given that when implementing it, you'll be finding the sum of discrete elements, I prefer this notation.

[^3]: This is usually an actual derivative (\\(de/dt\\)), but again, I prefer this notation, which more closely mirrors the how this is implemented in code.

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Fcontroltheory%2F2024%2F02%2F11%2Fcontrol-theory-intro.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)