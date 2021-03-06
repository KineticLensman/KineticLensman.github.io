---
layout: post
title: "Generative art [1]"
date: 2018-12-24
---

# Computer-generated art

<img srcset="/assets/images_0001_generative/2018-12-01-140138-500.jpg 500w,
             /assets/images_0001_generative/2018-12-01-140138-1200.jpg 1200w"
     sizes="(max-width: 500px) 500px,
	    (max-width: 1200px) 1200px,
            1200px"
     src="/assets/images_0001_generative/2018-12-01-140138-1200.jpg" alt="Generative art">

I didn't use Photoshop or a camera to create this picture - instead I coded it using ideas from the world of generative art: the creation of art by autonomous systems (typically software) following procedures defined by the artist.

Computer-based generative art first appeared in the 1960s and has been used to create imagery, music, architectural designs and even stories. Nowadays, one of the main applications is creating landscapes, buildings and other content for computer games. There are some really good instructional resources available, and [one of these](https://inconvergent.net/generative/) by Anders Hoff inspired me to have a go myself.

## How it works

For my first try, I developed a really simple algorithm (inspired by one of Hoff's). First, draw some points in a grid on the screen. Then, repeat the following process many, many times:
* slightly move all of the points
* draw dotted lines between their new positions

The computer moves the points so that over time, each point gradually traces out a circle. Because the circles created by the moving points vary in size, the lines drawn between the points create fascinating spirograph-like shapes. The colour of the dots that form each line depend on the length of the line.

The resultant image illustrates a very common outcome in generative art: extremely complex patterns emerge from the repeated application of quite simple rules. It's no surprise that generative art can create images that are biological in their complexity, since many forms in nature are themselves created by pattern-based developmental process. I'm planning to create more images like this, gradually increasing their complexity by developing and exploring more sophisticated algorithms.

## Technical details

I programmed this pattern generator in the [C#](https://en.wikipedia.org/wiki/C_Sharp_(programming_language)) language using Microsoft [Visual Studio](https://docs.microsoft.com/en-us/visualstudio/get-started/visual-studio-ide?view=vs-2017) (which is free).

The pattern is controlled by the following parameters:
* Number of nodes
* Initial distance between the nodes
* Number of iterations of the move-draw loop
* Number of points drawn on each node-to-node line

The code maintains a 2D array of the nodes, each of which has:
* Current screen position (in x,y) coordinates (updated after each movement)
* Speed (assigned a random value when the node is created, left unchanged thereafter)
* Heading (assigned a random value when the node is created)
* Heading delta (assigned a random value when the node is created)
* A list of neighbours to which dotted lines will be drawn (set to be it's initial nearest neighbours). 

Coding the algorithm was quite straightforward, and was actually faster than learning how to create the basic program framework in Visual Studio. Some additional complexity arose because I wanted the graphics window to be responsive while the pattern-generating algorithm was running: this involved making the code multi-threaded.
