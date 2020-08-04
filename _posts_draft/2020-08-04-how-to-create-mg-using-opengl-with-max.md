---
layout: post
date: 2020-08-04
title: "How To Create Motiongrams Using OpenGL With Max/MSP/Jitter"
image: /assets/img/2020_08_04_spectral_motiongram_preview.jpg
categories: General
excerpt: "In this post, I conduct an experiment to determine what the most optimal and effective system is for recording and previewing spectral images of motion over time using OpenGL with Max/MSP/Jitter."
Keywords: MaxMSP, OpenGL, Jitter, spectral, image, analysis, realtime, motiongram, videogram
---

As the title suggests, in this post we will investigate what the most optimal and effective system is for recording and previewing motiongrams (spectral images of movement in video over time) in OpenGL with Max/MSP/Jitter. Now, if there were more than 2 unfamiliar words in the previous sentence, I highly suggest you read through my previous post on the [**AudioVideoAnalysis**](https://aleksandertidemann.github.io/projects/2020/07/24/audiovideoanalysis.html) application, where I elaborate more on the subject and present my motivation for doing a longer post on this specific topic.

This "investigation" consists of an experiment I designed to test several motiongram-producing systems in various conditions to determine which of these methods was most effective at doing this kind of spectral image processing in an OpenGL context. After elaborating a little on the background and method of this experiment, I will discuss the results before finally presenting the system that proved most effective for this task.


# Background
Recently, I redeveloped a standalone application for realtime spectral analysis of audio and video called AudioVideoAnalysis for the RITMO Center for Interdisciplinary Studies in Rhythm, Time and Motion at the University of Oslo. The application analyzes the spectral content of video by producing something called motiongrams, a concept and research topic originally explored by professor [**Alexander Refsum Jensenius**](http://people.uio.no/alexanje) as a part of a [**Musical Gestures Toolbox for Max**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/musicalgesturestoolbox/mgt-max/). Read more about this research [**in this article**](https://www.duo.uio.no/handle/10852/26907).   

<figure style="float: none">
   <img src="/assets/img/2020_08_04_spectral_motiongram_preview.jpg" alt="motiongram preview"
   title="motiongram preview" width="auto" />
   <figcaption>3 motiongrams from a dancer video. Here, the mean of the motion image (Y-axis) is printed over time (X-axis). Image retrieved from <a href="https://www.hf.uio.no/imv/english/research/projects/mg/subprojects/motiongrams/"><b>UiO</b></a></figcaption>
</figure>

A motiongram is an image representation of movement over time. We create these images by first extracting the difference between the current frame and the previous frame. This will generate a motion image where only the movement in the video is represented as color while the rest is black. Then, we create one-dimensional matrices of these frames by calculating the mean values of either every row or column depending on if we want to represent time on the X or Y-axis, as demonstrated in Figure 1 below. Finally, we print the frames sequentially across a canvas/display window as a motiongram.

<figure style="float: none">
   <img src="/assets/img/2020_08_04_spectral_motiongram_diagram.jpg" alt="motiongram diagram"
   title="motiongram diagram" width="640" />
   <figcaption>Figure 1. Calculating mean values for every row will produce a motiongram where time is represented in the X-axis.</figcaption>
</figure>

In the Musical Gestures Toolbox for Max, this mean calculating process was handled by a MaxMSP external object called ```[xray.jit.mean]```, developed by Wesley Smith ([**read more here**](https://cycling74.com/forums/get-the-xray-package-by-wesley-smith/)). The XRAY external package is a set of jitter objects that do video processing on the CPU, similar to how every jitter object in MaxMSP operates. When dealing with graphics and video processing in MaxMSP today, it's highly recommended you "wrap your code" in something called an OpenGL context. OpenGL (Open Graphics Library) is a cross-platform graphics interface for 3D and 2D graphics. It's often considered a kind of "industry standard" for when dealing with digital graphics related work. Anyhow, a key feature of OpenGL is that it does all of its processing on the GPU. Because of this, I wanted the 2.0 version of AudioVideoAnalysis to have OpenGL image rendering. However, this gave rise to an interesting problem:

> **Should an OpenGL motiongram system calculate its running mean values on the CPU or the GPU?**

# The Problem Explained
When doing computationally expensive tasks like calculating thousands (if not millions) of mathematical operations many times per second, as our system demands, the general norm is that these operations should be executed on the CPU. This is because the CPU is specifically designed for these kinds of tasks while the GPU is more optimized to deal with graphics. [**There's a great video**](https://www.youtube.com/watch?v=V3_p9R7YG-g) on the pros and cons of processing video information on the CPU vs. the GPU in a MaxMSP related context by Federico Foderero on his YouTube channel. Be sure to check it out if you're interested in learning more.

With this in mind, the solution seems obvious. The mean calculation should be on the CPU. However, according to Cycling74's own documentation on the [**Best Practices in Jitter**](https://cycling74.com/tutorials/best-practices-in-jitter-part-1), there is reason to doubt this in our particular case. As explained by in the Cycling74 article, if you transition from OpenGL jitter to regular jitter (from the GPU to the CPU) you should use specific readback and downsampling methods that minimize data copies between the processing units. What this essentially means is that these transitions can increase computational load, creating a creating bottleneck-like effect. I therefore wondered, what would be the optimal solution for my realtime application? What would perform best, keeping everything on the GPU for consistency, or migrating some of the tasks to and from the CPU?

<!--*
That such a computationally expensive task, basically process 3 value for 921 600 pixels many (up to 60) times per second(!).  
-->
# Method
To explore this question, I designed an experiment with the explicit purpose of testing whether a CPU-or GPU-oriented method of mean calculations was best practice in a realtime application like AudioVideoAnaysis. The experiment consisted of testing three different OpenGL motiongram-producing systems in two different configurations on two different computers. The tests had specific pre-determined conditions which each system was subjected to and subsequently rated based on certain criterion.   

To generate video, I used the dedicated built-in webcameras of my Macbook pro And Lenovo ThinkPad. For consistency, I conducted all tests with the highest possible camera resolution (being 1280x720). Additionally, each system in each configuration had their own *downsample* trigger that downsampled the video-feed from 1280x720 to 320x240. We could therefore.

on integrated graphics card. Because ....
So only use laptops with middle/low-grade GPUs and CPUs to test.

Make an application that's supported on as many systems and machines as possible.

### Systems
1. The first system uses a readback method to process the mean calculation on the CPU via ```[xray.jit.mean]```. From now, we refer to this system as the **XRAY CPU method**.  

2. The second system uses a ```[jit.gl.pix]``` (an OpenGL object in MaxMSP) and some GenExpr code to process the mean calculations on the GPU. From now, we refer to this system as the **GPU method**.  

3. The third system uses the same readback method as system one, but instaed uses a ```[jit.pix]``` (a regular jitter object in MaxMSP) with the same GenExpr code from system two to process the mean calculation on the CPU. From now, we refer to this system as the **PIX CPU method**.

### Configurations
1. In the first configuration we test the systems on their own. In other words, in an isolated fashion. From now, we refer to this configuration as an **isolated configuration**.

2. In the second configuration we test the systems as part of a bigger system. Here, I modified three version of the AudioVideoAnalysis 2.0, each with its own dedicated system. From now, we refer to this system as the **AVA configuration**.


TWO PICTURES HERE...

### Test Conditions
* **Low battery** - The idea of testing the systems on a laptop with low battery is because your laptop most likely behaves and prioritises differently in this condition. A high rating means the application performs WELL on low battery.

* **Unit load/stress** - How much does the method stress the CPU or GPU (depending on the method). A high rating means the unit does NOT stress the system much.

* **FPS** - Test whether it behaves eqully well with a high printing rate.

* **Audio Config** - We use audio (MSP) signals to control the printing rate of our systems. We should therefore test whether adjusting the ```I/O vector size``` or the ```vector size``` affects performance in any way.

### Test Criterion
* Missing frames - A drop in frame rate corresponds to heavy load on the machine.
  * FPS
* Crashes
* Pixelated resolution.
* Extensive GPU/CPU load (from task manager or activity monitor)

### Devices
* **Lenovo Thinkpad P53**
  * 2.6 GHz Intel Core i7 9th gen six-cores
  * GPU
    * Intel UHD Graphics 630 (in use)
    * NVIDIA Quadro T2000
  * Windows 10 Home
  * 16GB RAM
  * 500GB SSD
  * Integraded Webcamea with up to 1280x720 resolution.

* **Macbook Pro 15" early 2011**
  * 2 GHz Intel Core i7 processor quad-cores
  * Intel HD Graphics 3000 512MB
  * OSX HighSierra v.10.13.6
  * 8GB RAM
  * 256GB SSD
  * Integraded Webcamera with up to 1280x720 resolution.

### Default Settings
* **Audio**
  * Sampling rate = 44100
  * I/O vector size = 1024
  * Signal vector size = 256
* **Video**
  * Camera = integrated laptop webcam.
  * Resolution = 1280x720
  * Downsampled resolution = 320x240
  * FPS = 60, if possible

The necessity of defining some audio/MSP parameters will become clear later on in the text, specifically under the Clock header in the Development section.   


# Construction

HOW are the two methods built explicitly in max.
Lets call them *readback method* and the *GPU method*
show images etc.

## Motion image.
Alwasy must have @unique 1 in jit.grab.

And use float32 matrix/texture type, storing our matrix/texture values as 32-bit floating point numbers.

jit.world:
* black erase color
* sync 0 because we want the FPS to be indipendent of your monitors refresh rate.
* FPS 60 (or more), alternitavely set interval at 17 (or less). Will always strive to output 60 frame per second.

jit.gl.layer:
* Preseve Aspect ratio of the incoming image
* transform_reset to orthographic normalized (2) because we want our image to automatically be scaled to display window size.

overdrive off(!)

The downsampling of my test applications is a bit crude. If its worth investigating more how this should relate to the chosen resolution, it should downsample by a factor of the dimensions.
320 240. Its about a third.
16:9 =1280x720

## CPU Method

Readback method
Sånn og sånn.
Xray MaxMSP external.

Also created one with the GPU methods for-loop GenExpr code in [jit.pix], which is processed on the CPU. This is to see whether the codebox coding may be the culprit. Also, if there is no differnce in the peformance of these Readback-methods, then we could at remove the xray.jit.mean dependancy for good.

## Clock
Use audio signal as counter because of AVA. Its reliable, highly controllable and stable counter method across platforms and systems.
Explain why the default is whats in use.

Only use internal sound driver. Again.. for democratic puposes.


The ```Signal Vector Size``` in MaxMSP sets the number of samples that are calculated by MSP objects at any one time.

In this case, we want a resonable vector size to ensure a as smooth interpolation of numbers as possible that provide the printing coordinates for incoming motion image matrix/texture.

The signal vector size only as a slight effect of the overall performance.

smaller ```I/O Vector Size``` consume a greater percentage of the computer's resources. *kanskje prøve en høyere??*
kanskje prøve 1024 I/O vector size.
NOT TOO LARGE, which can also be bad.

SINCE
With a maximum of 60FPS which is one frame every 16ms.

We can safely
bring our signal vector size up to 256, which caulculate a new vector every 5,8ms, and snapshot the float values every 10th millisecond without problems, given a sampling of 44.1Khz.

In theory, this ensures that our system . smooth interpolated stream of numbers.

### Printing Rate


# Results

## CPU Method

Tested on Thinkpad, Yoga and Macbook Pro.
Based on the conditions above.

* Overall well on clean(!), no troubles. Not as great in a big application, occasional hick-up here and there. But vector size an I/O had considerable effect when trying out on the AVA mod (bigger application). Suprising!.

However, on the Macbook pro (a less powerful machine, by far), using webcam in full re on a bigger application (AVA mod) is unusable. Also recorded the frame rate drop down to almost 6 FPS. sometimes.

Multiple crashes also..

* Performance got better with downsampled, as expected. But not as much as You'd might think. About 10% increase from 1280x720 to a 320x240 resoulution image. This speaks very well of the xray.jit.mean.

On downsample I experienced reasonably good images (with only one or two missing frames on an entire recording session) with a FPS as low as 14/15.

* Higher temporal resolution made lower res images, as expected, but no performance difference from high temp res.

* How about low battery? Performed as well in the clean version. There were noticable dips in the AVA when using full res. However, downsampled version worked suprinsingly well on both apps on very low battery.

* No real (percievable) difference in image clarity from high res to downsampled res. Which is good

jada jada jada

<table>
  <tr>
    <td> <b>Method</b>
    <td colspan="8">Readback CPU Method with <b>[xray.jit.mean]
  <tr>
    <td> <b>Resolution
    <td colspan="4">Full texture/video resolution
    <td colspan="4">Downsampled texture/video resolution
  <tr>
    <td> <b>Test Conditions
    <td> Low Battery
    <td> System Load/Stress
    <td> FPS
    <td> Audio Config
    <td> Low Battery
    <td> System Load/Stress
    <td> FPS
    <td> Audio Config
  <tr>
    <td> <b>Performance
    <td> 5/10
    <td> 6/10
    <td> From 6 to 15
    <td> Changing "I/O vector size" and "vector size" had good impact.
    <td> 8/10
    <td> 8/10
    <td> From 15 to 30
    <td> Changing "I/O vector size" and "vector size" had good impact.
  <tr>
    <td> <b>Image Clarity
    <td colspan="4"> 8/10
    <td colspan="4"> 8/10
  </tr>
  <tr>
    <td> <b>As part of larger system (AVA 2.0)</td>
      <td colspan="4"> Performance got worse, with lags and occasional hick-ups, but was well corrugated by increasing vector I/O size to 1024 and vector size to 256. </td>
      <td colspan="4"> No real difference from system on its own. </td>
</table>


Okay here er no greier.

## GPU method
Tested on Thinkpad, Yoga and Macbook Pro.

* overall experience of high res.
Consistently lagging in the clean version. More drastic in the big version. Changing audio parameters has no impact.. it seems. However, Changing the GPU to a more powerful one generated smooth images, as expected, but maxed out the gpu in both instances when viewing a 1280x720 feed.
At full res:
FPS sometimes as low as 3, when using the internal graphics on macbook pro. Usually around 8.
Around 10-15 when using the NVIDIA.

**Whats very interesting is the CPU load in relation to the readback method. The load is the same, meaning the xray.jit.mean uses a tiny portion of cpu power.. etc...**

* Overall downsampled. Difference in performance (load)
Downsampling shows good results aesthetically (no lags or hickups) when its by itself. However, in a bigger application there is consistent occasional hick-ups.
FPS at a steady 30FPS throughout.

* Low battery
often caused the app to glitch and be slightly unresponsive, but didn't create any more distorted images. This is a good thing, as we want various conditions to effect our image printing as little as possible.

* high temporal res
Steady, no change. Lower image res as expected.

* Image clarity from high res to downsampled.
There is no real difference here. Either downsampled or not, CPU or GPU.


<table>
  <tr>
    <td> <b>Method </td>
    <td colspan="8">GPU Method with for-loop in <b>[jit.gl.pix]</td>
  </tr>
  <tr>
    <td> <b>Resolution </td>
    <td colspan="4">Full texture/video resolution</td>
    <td colspan="4">Downsampled texture/video resolution</td>
  </tr>
  <tr>
    <td> <b>Test Conditions</td>
    <td> Low Battery </td>
    <td> System Load/Stress </td>
    <td> FPS </td>
    <td> Audio Config </td>
    <td> Low Battery </td>
    <td> System Load/Stress </td>
    <td> FPS </td>
    <td> Audio Config </td>
  </tr>
  <tr>
    <td> <b>Performance</td>
    <td> 5/10 </td>
    <td> 2/10 </td>
    <td> From 2 to 6 </td>
    <td> No impact </td>
    <td> 7/10 </td>
    <td> 5/10 </td>
    <td> From 20 to 30 </td>
    <td> No impact </td>
  </tr>
  <tr>
    <td> <b>Image Clarity</td>
    <td colspan="4"> 8/10 </td>
    <td colspan="4"> 8/10 </td>
  </tr>
  <tr>
    <td> <b>As part of larger system (AVA 2.0)</td>
      <td colspan="4"> Performance got slightly worse, but GPU load was already maxed out when running system by itself. </td>
      <td colspan="4"> Performance stayed the same. </td>
</table>


## Jit.pix with CPU if it works so well.

Tested on Thinkpad, Yoga and Macbook Pro.

How did it perform? Awful... So maybe the GenExpr lagunage is slow or unfit for these complex purposes?
Hopefully, have some documentation here.


Full res was an immediate crash on my Thinkpad, with the best CPU, so obviously this was not gonna fly on other machines.

However, I did test on my Macbook Pro to verify its uselessness.

Rather interesting discovery that might shed some light on the usefulness of for-loops in GenExpr for these types of task.


<table>
  <tr>
    <td> <b>Method </td>
    <td colspan="8">Readback CPU Method with for-loop in <b>[jit.pix]</td>
  </tr>
  <tr>
    <td> <b>Resolution </td>
    <td colspan="4">Full texture/video resolution</td>
    <td colspan="4">Downsampled texture/video resolution</td>
  </tr>
  <tr>
    <td> <b>Test Conditions</td>
    <td> Low Battery </td>
    <td> System Load/Stress </td>
    <td> FPS </td>
    <td> Audio Config </td>
    <td> Low Battery </td>
    <td> System Load/Stress </td>
    <td> FPS </td>
    <td> Audio Config </td>
  </tr>
  <tr>
    <td> <b>Performance</td>
    <td> 0/10 </td>
    <td> 0/10 </td>
    <td> 0 </td>
    <td> No noticable impact. </td>
    <td> 0/10 </td>
    <td> 1/10 </td>
    <td> 11 </td>
    <td> No noticable impact. </td>
  </tr>
  <tr>
    <td> <b>Image Clarity</td>
    <td colspan="4"> 0/10 </td>
    <td colspan="4"> 0/10 </td>
  </tr>
  <tr>
    <td> <b>As part of larger system (AVA 2.0)</td>
      <td colspan="4">  </td>
      <td colspan="4">  </td>
</table>

# CONCLUSION:
results suggest that ...

Downsampled version.
Without a doubt. There seems to be no indication of percievably reduced image quality. This is because of the size of the window and the printing rate decides much of what resolution we want. In any case, there's no reason to stress our machines to such a degree for such a small aesthetic effect.

CPU readback method.

Biggest shock was the difference between the gen code in jit.pix and the xray.jit.mean, both CPU oriented tasks.

So some of the troubles experienced with the GPU method might be attributed certain factors and traits with the GenExpr code environment in MaxMSP.
In other words, there might be a significant performance difference if the code as built and compiled in C++ as an external, for instance. This remains to be tested.

As reference, when we process audio we often sample between 44 and 48 thousand times per second.
https://docs.cycling74.com/max7/tutorials/03_msphowmspworks

General speed of control signals in MaxMSP is 0.001, so a message per millisecond. I suspect the issue lies here. That such a computationally expensive task, basically process 3 value for 921 600 pixels many times per second(!).  


Seems that its good to seperate the load on GPU and CPU.

So we should always downsample.

To illustrate how we could take a more algorithmic approach to downsampling our matricies/textures, I made a simple program in GenExpr. It works as follows:
* If the X-dim is larger than the Y-dim, and a third of the X-dim is greater than 320, then we scale the X-dim down to this value while preserving the aspect for the Y-dim.
* However, if a third of the X-dim is smaller than 320 we simply let the X-dim be 320 and scale the Y-dim to fit the original aspect ratio.
* The processes is a little different if the Y-dim is larger than the X-dim, but its essentially the same. In these cases we take 240 as a "standard" and scale the X-dim accordingly.


```
xdim, ydim = 0;

if (in1 > in2) { // if X-dim is larger than Y-dim.
	if (in1 > 320) {
		if (in1/3 > 320) {
			ydim = (in2 / in1) * (in1/3); // Perserving aspect ratio.
			xdim = in1/3;
		} else {
			ydim = (in2/in1) * 320;
			xdim = 320;
		}
	} else {
		ydim = in2;
		xdim = in1;
	}
} else { // if Y-dim is larger than X-dim.
	if (in2 > 240) {
		if (in2/3 > 240) {
			ydim = in2/3;
			xdim = (in1/in2) * (in2/3); // Perserving aspect ratio.
		} else {
			ydim = 240;
			xdim = (in1/in2) * 240;
		}
	} else {
		ydim = in2;
		xdim = in1;
	}
}

out2 = ydim;
out1 = xdim;
```
hello
