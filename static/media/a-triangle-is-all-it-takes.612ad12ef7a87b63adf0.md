# A Triangle is all it takes

Have you ever wondered how a game renders such cool graphics on the display? If so, you’re in the right place. I also wondered the same, and it took me a good amount of effort to figure it out. In this series, I’m going to try to explain it in the easiest way possible so you might not have to look that hard for the same answer.

Spoiler alert: the answer lies in the title of this blog. Yep! To a large extent, a triangle is all it takes to render those cool graphics on your display. You might be wondering how and the answer might blow your mind. It surely blew mine when I first got to know the same. I think it’s best explained with an example image.


![Mario built with triangles](/blogs/mario_1.jpg)


Yep, most real time 3D graphics are built from triangles like these. You might ask why triangles? why not just points aka vertices or lines? Well you would not be wrong to ask that but a triangle itself is made up of those things. The fact is, it's a primitive with least number of edges that you can fill with colors and every other shape in graphics can be made from multiple triangles. But this Mario model is clearly made up of a lot of triangles and it’s just one object. Honestly, in an actual frame there is a lot more than this: there might be a cart, a road, trees, and a lot more things.

Also, it’s just one frame. Imagine the game running at 60 FPS: that doesn’t increase the number of triangles in a single frame, but it does increase the total number of triangles that need to be rendered per second. But I said a triangle, right? Well, believe me in a graphics pipeline, if you are able to render a single triangle, then it is not that difficult to understand how to render an entire graphics object.
Spoiler alert, this is because the graphics APIs that make the graphics pipeline to render graphics objects are state based APIs. Let me simplify that, Imagine you have a camera. Unfortunately, a really complicated manual camera. Now you will need to set up a lot of settings like aperture, Shutter Speed, ISO, White Balance, etc before you even click your first picture but once you setup up the camera, it's not that difficult to click a lot of pictures with same settings! A graphics API kind of work in the similar way. I will explain what are these graphics APIs from super ground up in my next blog but for now remember this state based nature of graphics APIs. But still, why are they like that?

To understand the reason behind it, we need to look at the hardware that is responsible for this rendering: the Graphics Processing Unit (GPU). You might have always wondered why exactly we need a GPU for playing games (or more precisely, to render graphics). You might even argue why a CPU is not enough? After all, it’s also a processing unit. And honestly, a CPU is more general purpose than a GPU. A CPU is great at multi-tasking, it means it can do a lot of different kind of tasks.

So can a CPU render a triangle? Yes, it surely can. But it won’t do it as efficiently as a GPU. A GPU might not be great at multi-tasking, but nothing comes close to it at multi-processing. Think of it as a hardware that can do a relatively narrow set of tasks, but it can do the same task multiple number of times at once.

For example, if you had to add two numbers, both CPU and GPU are capable of doing it. But what if you have to add 2 numbers a million or billions of times? In this case, a GPU can do it much faster, infact in less than a second because of its parallel processing capabilities. Isn’t that perfect when you have to process so many triangles in a fraction of a second? Yes it is! Let’s look at what exactly a GPU is.

![what-is-a-gpu](/blogs/card_2.jpg)

You might think that’s a GPU but sorry no, that’s a graphics card. This card has a chip on it which is the GPU (and many systems also have “integrated GPUs” built into the CPU/SoC). On a graphics card, the GPU is the chip package mounted on the board. It looks like this:
![Graphics-Card-PCB-Assembly](/blogs/gpu_3.jpg)

I would love to deep dive into the architecture of a _similar_ GPU chip and explain how exactly a GPU works, but let’s do that another day. Believe me, it will make a lot more sense later when we have a better understanding of the other stages we need to go through to render a triangle on a display.

Before telling what all stages are required to render a triangle, let’s intuitively think what all we need for this.

Since a graphics object is displayed on a display, we will need a display and some sort of operating system/window system that will talk to the display controller to render our triangle.

We will also need to write a program to render a triangle. You can write a program from scratch for it, it’s not impossible (almost nothing is impossible) but it would be insanely difficult to do that alone because rendering millions of triangles so efficiently and so consistently from scratch is not going to be easy.

Luckily, as spoiled for you earlier, we have graphics APIs to help us do that. A graphics API helps you write a rendering pipeline program that has various pipeline stages, and at the end of the pipeline we have our triangle ready to be turned into display pixels.

So what exactly is a graphics API and how does this graphics API talk to the OS/window system and the GPU driver? Let's try to answer that in the next blog!

Thanks!
Happy reading :)