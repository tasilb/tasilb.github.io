---
layout: post
title: Simulating Robotic Control with Processing (debrief)
excerpt: I designed robotic control workshops with simulations made from scratch in Processing.
---

Starting in September, I've been involved with the leadership team in [Robotics@Berkeley][RAB], a campus organization aiming to increase the accessibility of robotics and related fields at UC Berkeley. One of the ways we're approaching this task is through hosting workshops in robotics-related topics across all technical skill levels.

### Background

Our first workshop series is on robotic controls. It's a three-part series about the basics of controls in the context of a mobile robot with sensing capabilities. The [first workshop][cw1], intended for complete beginners, went over control abstractions in movement and sensor data, interpretations of those abstractions, and how to apply them using fundamental programming concepts. Attendees navigated a simulated robot in a tile-based plane around obstacles and to a goal. The [second workshop][cw2] discussed additional variables, sensors, and considerations such as speed and heading for controls in a continuous environment, as opposed to the discrete tiles in the first one. Like the previous workshop, attendees controlled a simulated robot, but this time on a free-handed map. The third workshop, which is still in the works, will move from our simulated robots to a physical version and apply what we've covered about controls to something tangible. Most of the work involved in putting together these first two workshops was in making the robot simulations, and since I was in charge of developing the software, I thought to share some insights on the development process. For this post, I'll be referring to the workshops by the abbreviations CW1 and CW2

### Processing

Our simulations were intended to encompass the minimum states required to illustrate concepts in the talk into a simple, interactive system for inputting robot instructions, executing them, and obtaining visual feedback. Given these parameters, as well as a close deadline, we decided to implement our simulation in [Processing][processing] for the following reasons:

- Processing uses Java syntax, so it was easy to understand and quick to whip up an OO design for our minimum state simulation engine.
- Processing is primarily a visualization tool, and so the graphical representation requirements were a cinch. A robust graphical library meant that most of the heavy lifting involved in displaying things on a screen was a search away through the Processing documentation. Being able to directly manipulate pixels with Processing also proved to be immensely helpful.
- The Processing IDE just about met our need for a simulation interface singlehandedly. The IDE is easy to navigate for beginners, and hooking up user code to the simulation was only a matter of adding a file with an empty function to fill in. We didn't need to spend time figuring out how to make a GUI or parse text, compile on the fly, or anything of that sort. It also made "installing" the workshop materials effortless.

### Design Overview

CW1 was straight-forward, so I designed it and coded it simultaneously. This gave the CW1 code some cruft, but nothing was broken. CW2 ended up being considerably rushed, and the design wasn't great. I thought it wouldn't be too complicated of an extension from CW1, but there were few opportunities to reuse CW1 code due to how different the two simulations actually were, as well as several issues I encountered while trying to put CW2 together. In the end, everything was okay and both of the workshops went off without a hitch.

#### Controls Workshop I

CW1 had an agent class, a tile class, and a map class, where tiles belonged to spaces on the map and interacted with an agent. The map determined whether or not a tile was a wall or a goal. An agent could perceive if the tile directly in front was a wall, as well as what its coordinates were and what direction it was facing via "sensor" methods that asked for agent, tile, and map attributes. Most of the issues in this part involved trying to render the environment and redrawing after each of the agent's moves. Problems at different points in development ranged from the agent's state changes not being reflected on-screen, to the agent's sprite teleporting to a goal tile without any intermediate steps. These issues were mainly Processing specifics that tripped me up as a first-time user, and were resolved with a careful rereading of the documentation.

#### Controls Workshop II

CW2 saw many things get stuffed into the agent class, since agents no longer moved along discrete tiles. To describe the environment, we drew maps in MS Paint and had Processing load up the PNG files so that we could access the pixel data. Our main motivation for the PNG environments was that for CW1, creating maps by manually changing the attributes of each tile was tedious, and PNG environments allowed us to easily construct more complicated scenarios. PNG environments also eliminated the need for any significant bookkeeping backend, as all environment information could be obtained by getting a pixel's color. In the agent class, I replaced the position sensing method for a distance sensing method that scanned pixels in a straight line until a wall was seen, and rewrote the agent's movement system to accommodate the continuous environment. Instead of moving a unit per clock tick in one of four directions, the agent moved each tick based on its throttle and heading. Collision detection in the PNG environment, was extremely simplistic. Before redrawing the agent after a position change, the code would check for a wall or a goal based on the color of the pixel beneath where the tip of the agent was supposed to move.

### Technical Remarks

The CW2 simulation was usable for the purposes of the workshop, but as a standalone piece of code, it's not very useful because of oversights and shortcuts in the software design that I let slide in the interest of time. The problems with the design were due to the muddling of the distinction between where on the screen the agent appeared, and where the trigonometry said it would be. The environment wasn't truly continuous and was indeed limited by being defined by the finite precision of a pixel array, but this should not have had any effect on the behavior of a completely correct agent class. Instead, the implementation I went with made some sacrifices as to how accurate the simulation would be in order to simplify the code and achieve a MVP faster. Notable omissions include a robust distance sensing method, collision detection for the entire agent sprite, and collision detection for the entire length moved in a tick. These compromises enabled some amusing agent behavior.

Say, for example, there was a map with a 45-degree wall that is 1 pixel wide, with an agent facing perpendicular to the wall with no space between the two. The agent's distance-sensing method would not detect the wall, since, technically, the space in front of the tip of the agent is free. And if the agent were told to move forward from that place, it would pass right through. The easy fix for these edge cases was to make all the walls at least several pixels thick. Agents could also pass through walls if they were moving enough units per tick, and so the power rating, or how many units per tick the agent can move at full throttle, was obscured for the workshop.

Technically, the sensing method is correct as-is, but it just isn't very useful in the scenario I described. Just because you can shine a laser through a small crack doesn't mean that you can pass through it. In the case of the above wall, the contact point at the corner of the two pixels would be considered the crack through which the scan line shines. The method could be made more useful in general by making the scan line fatter, or by starting additional scan lines from elsewhere on the agent's sprite. The latter would only be useful if collision detection were implemented for the entire sprite--otherwise a single scan line for the pixel-sized agent would suffice. Improving collision detection, which I consider the sloppiest and weakest part of the codebase, would involve refactoring the agent class to define its boundaries better and calculating additional loops to scan all pixels that might be occupied by the agent's movement instead of looking at a single pixel at the destination.

### Concluding Remarks

Besides the room for technical improvements, I thought that the overall development process, as well as myself, had a lot of growing to do. For starters, the code was rather messy in certain places, nothing was properly encapsulated, and there was lots of unused and scrapped code floating around. Time management on this project was also bad--there was a lot of time spent fiddling with minor details and worrying about small things instead of staying focused on the greater task at hand. I had fun with it, though!

My personal takeaways for this project:

- plan effectively
- write better code
- think carefully, don't panic

[RAB]: http://rab.berkeley.edu/
[cw1]: https://github.com/ucberkeleyrobotics/controls-workshop-1
[cw2]: https://github.com/ucberkeleyrobotics/controls-workshop-2
[processing]: https://processing.org
