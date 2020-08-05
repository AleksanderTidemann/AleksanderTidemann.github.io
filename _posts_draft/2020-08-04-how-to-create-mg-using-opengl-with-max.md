---
layout: post
date: 2020-08-04
title: "The best way to create spectral mean images in Max/MSP/Jitter using OpenGL"
image: /assets/img/2020_08_04_spectral_main2.jpg
categories: General
excerpt: "In this post, I conduct an experiment to determine what the most optimal and effective system is for recording and previewing spectral images of motion over time in Max/MSP/Jitter using an OpenGL framework."
Keywords: MaxMSP, OpenGL, Jitter, spectral mean images, analysis, motiongram, videogram
---
<!--*
<figure style="float: none">
   <img src="/assets/img/2020_08_04_spectral_main2.jpg" alt="main"
   title="main" width="640" />
   <figcaption></figcaption>
</figure>
-->

As the title suggests, in this post we will investigate what the most optimal and effective system is for recording and previewing spectral mean images (motiongrams) in Max/MSP/Jitter using an OpenGL framework. I was motivated to explore this specific topic in-depth after running into some challenges while redeveloping an application that analyzes the spectral content of video and audio for the [**RITMO Centre for Interdisciplinary Studies in Rhythm, Time and Motion**](https://www.uio.no/ritmo/english/) at the University of Oslo, called AudioVideoAnalysis. I've written [**an entire post**](https://aleksandertidemann.github.io/projects/2020/07/24/audiovideoanalysis.html) about this application earlier so be sure to check it out if you're interested.

My investigation consists of an experiment where I designed and tested several of these OpenGL spectral mean image-producing systems in Max/MSP/Jitter to determine which was most effective. After briefly on elaborating on the background and method of this experiment, I will discuss the results before finally presenting the winning system(!)

# Background
Spectral mean images are representations of movement over time. These images are created by calculating the mean values of either every row or column of an incoming matrix, depending on if we want to represent time on the horizontal or vertical axis, and subsequently printing the outputted one-dimension matrices across a canvas or display.

<figure style="float: none">
   <img src="/assets/img/2020_08_04_spectral_motiongram_diagram.jpg" alt="motiongram diagram"
   title="motiongram diagram" width="640" />
   <figcaption>Calculating mean values for every row of a frame (matrix) can be used to create a spectral mean image with time on its horizontal axis.</figcaption>
</figure>

If we produce a spectral mean image from a motion image, we have something called a motiongram. We create a motion image by extracting the difference between two sequential frames of a video. This will render only the movement of the video in color while leaving the rest of the image black. The idea and concept behind motiongrams have been thoroughly researched and explored by professor [**Alexander Refsum Jensenius**](http://people.uio.no/alexanje) at the University of Oslo. You can read much more about the research side of motiongrams [**in this article**](https://www.duo.uio.no/handle/10852/26907).  

<figure style="float: none">
   <img src="/assets/img/2020_08_04_spectral_motiongram_preview.jpg" alt="motiongram preview"
   title="motiongram preview" width="auto" />
   <figcaption>Three horizontal motiongrams. Image retrieved from <a href="https://www.hf.uio.no/imv/english/research/projects/mg/subprojects/motiongrams/"><b>UiO</b></a></figcaption>
</figure>

In the outdated version of AudioVideoAnalysis, the application I was tasked to whip into shape, this mean calculating process was handled by a MaxMSP external object called ```[xray.jit.mean]```. The XRAY external package was developed by Wesley Smith ([**read more here**](https://cycling74.com/forums/get-the-xray-package-by-wesley-smith/)) and is a set of jitter objects that do video processing on the CPU, the same way every other jitter object in MaxMSP operates. However, when dealing with graphics and video processing in MaxMSP today, it's highly recommended you "wrap your code" in something called an OpenGL context that enables your GPU to take care of all the graphical processing. OpenGL (Open Graphics Library) is a cross-platform graphics interface for 3D and 2D graphics and is often considered an industry standard in digital graphics circles. Because of this, I wanted the new version of AudioVideoAnalysis to have OpenGL image rendering. However, this gave rise to an interesting problem:

> **Should an OpenGL spectral mean image system in Max/MSP/Jitter do pixel mean calculations on the CPU or the GPU?**

# The Problem Explained
When doing computationally expensive tasks like calculating thousands (if not millions) of mathematical operations many times per second, as our system demands, the general norm is that these operations should be executed on the CPU. This is because the CPU is specifically designed for these kinds of tasks while the GPU is more optimized to deal with graphics. [**There's a great video**](https://www.youtube.com/watch?v=V3_p9R7YG-g) on the pros and cons of processing video in Max/MSP/Jitter on the CPU vs. the GPU by Federico Foderero. Be sure to check it out if you're interested in learning more.

With these facts considered, it seems obvious that the mean calculations in question should be executed on the CPU. However, according to official documentation on the [**Best Practices in Jitter**](https://cycling74.com/tutorials/best-practices-in-jitter-part-1), there is reason to doubt that this is best practice in our particular case. The Cycling74 article states that if you transition from OpenGL jitter to regular jitter (from the GPU to the CPU) you should use specific readback and downsampling methods that minimize data copies between the processing units. What this essentially means is that you risk creating a bottleneck in your processing chain if you continually transition heavy loads between units.

Therefore, with this risk in mind, I wondered whether keeping all of the processing for my application on the GPU would be more effective than migrating some of the math heavy tasks to and from the CPU.

<!--*
That such a computationally expensive task, basically process 3 value for 921 600 pixels many (up to 60) times per second(!).  
-->
# Method
To explore this question, I designed an experiment with the explicit purpose of testing this assumption thoroughly. The experiment consisted of testing three different OpenGL spectral mean image systems in two different configurations on two different machines. Testing was conducted on one system at a time (on both machines) in two rounds. First round with a high resolution image (1280x720) and the second round with a much lower resolution (320x240).

### Systems
1. The first system uses a readback method to process the mean calculation on the CPU via ```[xray.jit.mean]```. From now, we refer to this system as the **XRAY CPU method**.  

2. The second system uses a ```[jit.gl.pix]``` (an OpenGL object in MaxMSP) and some GenExpr code to process the mean calculations on the GPU. From now, we refer to this system as the **GPU method**.  

3. The third system uses the same readback method as system one, but instaed uses a ```[jit.pix]``` (a regular jitter object in MaxMSP) with the same GenExpr code from system two to process the mean calculation on the CPU. From now, we refer to this system as the **PIX CPU method**.

### Configurations

1. In the first configuration we test the systems on their own. In other words, in an isolated fashion. From now, we refer to this configuration as an **isolated configuration**.

2. In the second configuration we test the systems as part of a bigger system. Here, I modified three version of the AudioVideoAnalysis 2.0, each with its own dedicated system. From now, we refer to this system as the **AVA configuration**.

<figure style="float: left; margin-left: 10px;">
  <img src="/assets/img/2020_08_04_spectral_app.jpg"
  alt="motion average image" title="motion average image" width="320" />
  <figcaption>Each system had an isolated configurations. These "apps" only produced video/motiongrams but were capable of altering several parameters on the go.</figcaption>
</figure>

<figure style="float:none">
  <img src="/assets/img/2020_08_04_spectral_avamod.jpg"
  alt="average image" title="average image" width="360" />
  <figcaption>Each system also had an AVA configuration. These apps generated video/motiongrams, spectrograms of audio and more. These configurations let me test how the systems performed as part of a larger context.</figcaption>
</figure>

### Machines
Since one of my goal also was to find a system that would be "universally" stable, I decided to only use middle/low-grade machines with standard integrated Intel Graphics cards in testing. However, having the ability to switch to a NVIDIA Quadro T2000 GPU on my Lenovo Thinkpad will be valuable to get better perspectives on the performance of a particular system.

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

### Test Conditions
* **Full and downsampled resolutions** - To generate the video-feed, I used each machines dedicated built-in webcamera. For consistency, I conducted all tests with the same high resolution image (1280x720). Additionally, each system in each configuration had their own *downsample* trigger that downsampled the video-feed (1280x720) to 320x240. The purpose of this downsampling is to help generate a more holistic and better understanding of how each system performs overall.  

  * **Performance on low battery** - I checked how each system behaved on machines with low battery because laptops often behave and prioritize differently when in this condition. More so, the better a system performs on low battery the more stable it is. A high score on this condition means the system performed WELL on low battery.

  * **General GPU/CPU load** - An important (if not the most important) part of the testing was to monitor how each system stressed its dedicated processing unit. A high score in this category means the system did NOT stress the unit much.

  * **Performance with various audio parameters** - I use audio (MSP) signals as system clocks. Therefore, MSP controls the printing rate of our systems. We should therefore test whether adjusting the I/O Vector Size and the Vector Size affects performance in any way.

  * **Monitor FPS** - The FPS is closely linked to the *GPU/CPU load* and *audio parameter* categories. Basically, the higher FPS a system achieves, the more stable the system is. It can also indicate how much slack we have.

It's worth mentioning that a *well performing system* is a system that has few or no lags, does not crash (often), has a high and stable FPS, reasonable load and low occurrence of missing/blank frames.

### Clock
Because the systems use MSP (audio) signals as their main clock source, it's essential to define some default audio parameters beforehand. The clock determines how fast the images are printed across the display as it provides printing coordinates for the incoming one-dimensional image. I decided to use MSP (audio) signals for this process because its a reliable, highly controllable and stable counter mechanism.

<figure style="float: left; margin-right: 30px;">
  <img src="/assets/img/2020_08_04_spectral_audio_status.jpg"
  alt="motion average image" title="motion average image" width="250" />
  <figcaption></figcaption>
</figure>

In addition to a fixed I/O vector size, signal vector size and sampling rate, each system used the current machines internal sound card as main audio driver. Again, this was for universality and democratic purposes.

The ```I/O Vector Size``` controls the number of samples that are transferred to and from the audio interface at any one time. Therefore, a smaller I/O vector size will consume a greater percentage of the computer's CPU resources. However, we don't want this value to be too big either as it can stall other computational processes. So to make sure the audio rate was stable but not too demanding, I set the I/O vector size to a reasonable 1024.

The ```Signal Vector Size``` in Max/MSP/Jitter determines the number of samples that are calculated by MSP objects at any one time. Although [**it's documented**](https://docs.cycling74.com/max5/tutorials/msp-tut/mspaudioio.html) that variations in signal vector size only have slight effects on CPU performance, a larger vector size will nevertheless reduce some load. Considering that the systems opt for a maximum framerate of 60 FPS (one frame every 16ms), I could theoretically bring my signal vector size up to 256 ("outputting" a new vector every 5,8ms) without problems, given a sampling rate of 44.1Khz.

With these audio configurations, we should have no clocking issues even at the highest possible framerate (60) for the systems in question. Steady printing all around!

# Construction
Discuss some general max parameters of the systems before we go more in-depth into how the motion images are created and how the printing process is managed. Finally, we will look closer at how the three spectral mean image systems are made.

### General

jit.world:
* black erase color
* sync 0 because we want the FPS to be indipendent of your monitors refresh rate.
* FPS 60 (or more), alternitavely set interval at 17 (or less). Will always strive to output 60 frame per second.

jit.gl.layer:
* Preseve Aspect ratio of the incoming image
* transform_reset to orthographic normalized (2) because we want our image to automatically be scaled to display window size.

overdrive off(!) because we are doing video processing.

Alwasy must have @unique 1 in jit.grab.
And use float32 matrix/texture type, storing our matrix/texture values as 32-bit floating point numbers.

### Motion image.

The downsampling of my test applications is a bit crude. If its worth investigating more how this should relate to the chosen resolution, it should downsample by a factor of the dimensions.
320 240. Its about a third.
16:9 =1280x720

### Printing Rate

Yeah

### XRAY CPU method.

Readback method
Sånn og sånn.
Xray MaxMSP external.

Also created one with the GPU methods for-loop GenExpr code in [jit.pix], which is processed on the CPU. This is to see whether the codebox coding may be the culprit. Also, if there is no differnce in the peformance of these Readback-methods, then we could at remove the xray.jit.mean dependancy for good.

### GPU method.

### PIX CPU method.

# Results
