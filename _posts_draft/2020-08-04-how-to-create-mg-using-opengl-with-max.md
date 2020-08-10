---
layout: post
date: 2020-08-04
title: "Creating Spectral Mean Images in Max/MSP/Jitter Using OpenGL"
image: /assets/img/2020_08_04_spectral_main2.jpg
categories: General
excerpt: "In this post, I conduct an experiment to determine what the most effective system is for recording and previewing spectral images of motion over time in Max/MSP/Jitter using an OpenGL framework."
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

If we produce a spectral mean image from a motion image, we have something called a motiongram. We create a motion image by extracting the difference between two sequential frames of a video. This will render only the movement of the video in color while leaving the rest of the image black. The idea and concept behind motiongrams have been thoroughly researched and explored by professor [**Alexander Refsum Jensenius**](http://people.uio.no/alexanje) at the University of Oslo. You can read much more about this research side of videograms and motiongrams [**in this article**](https://www.duo.uio.no/handle/10852/26907).  

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
1. The first system uses a readback method to process the mean calculation on the CPU via ```[xray.jit.mean]```. From now, we refer to this system as the **XRAY method**.  

2. The second system uses a ```[jit.gl.pix]``` (an OpenGL object in MaxMSP) and some GenExpr code to process the mean calculations on the GPU. From now, we refer to this system as the **GPU method**.  

3. The third system uses the same readback method as system one, but instaed uses a ```[jit.pix]``` (a regular jitter object in MaxMSP) with the same GenExpr code from system two to process the mean calculation on the CPU. From now, we refer to this system as the **PIX method**.

### Configurations

1. In the first configuration we test the systems on their own. In other words, in an isolated fashion. From now, we refer to this configuration as an **isolated configuration**.

2. In the second configuration we test the systems as part of a bigger system. Here, I modified three version of the AudioVideoAnalysis 2.0, each with its own dedicated system. From now, we refer to this system as the **AVA configuration**.

<figure style="float: left; margin-left: 10px;">
  <img src="/assets/img/2020_08_04_spectral_app.jpg"
  alt="motion average image" title="motion average image" width="320" />
  <figcaption>Each method (XRAY, GPU and PIX) had an <b>isolated configuration</b>. These "apps" only produced video/motiongrams.</figcaption>
</figure>

<figure style="float:none">
  <img src="/assets/img/2020_08_04_spectral_avamod.jpg"
  alt="average image" title="average image" width="360" />
  <figcaption>Each method also had an <b>AVA configuration</b>. These apps generated video/motiongrams, spectrograms of audio and more. This configuration let me test how the individual methods performed when implemented as part of a larger system.</figcaption>
</figure>

### Machines
Since one of my goal also was to find a system that would be "universally" stable, I decided to only use middle/low-grade machines with standard integrated Intel Graphics cards in testing. However, having the ability to switch to a NVIDIA Quadro T2000 GPU on my Lenovo Thinkpad will be valuable to get better perspectives on the performance of particular systems.

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
* **High and low resolution video** - To generate video-feed, I used each machines dedicated web-camera. For consistency, I conducted all tests with the same high resolution image (1280x720). Additionally, each system in each configuration had their own *downsample* trigger that downsampled the video-feed (1280x720) to 320x240. The purpose of this downsampling is to help generate a more holistic and better understanding of how each system performs overall.  

  * **Performance on low battery** - I checked how each system behaved on machines with low battery because laptops often behave and prioritize differently when in this condition. More so, the better a system performs on low battery the more stable it is. A high score on this condition means the system performed WELL on low battery.

  * **General GPU/CPU load** - An important (if not the most important) part of the testing was to monitor how each system stressed its dedicated processing unit. A high score in this category means the system did NOT stress the unit much.

  * **Various audio parameter settings** - I use audio (MSP) signals as system clocks. Therefore, MSP controls the printing rate of our systems. We should therefore test whether adjusting some parameters, like the I/O Vector Size and the Vector Size, affects performance in any way.

  * **FPS (Frames Per Second)** - The FPS value will always be closely linked to the *GPU/CPU load* and *audio parameter* categories. Basically, the higher FPS a system achieves, the better the system is. However, I designed each system to have a maximum limit of 60 FPS. It's important to have a max limit because we want to be able to design a clock source whose rate we can ensure will always exceed the current FPS value.

  * **Image Clarity** - A well-performing system in my experiment is a system that shows few or no lags, does not crash (often), and most importantly has a low occurrence of missing/blank frames. These aesthetical aspects is what evaluate and rate through  Image Clarity category.

### Clock
As briefly mentioned, the systems use MSP (audio) signals as their main clock source/counter mechanism. Therefore, it's essential to define a set of fixed audio parameter settings beforehand. The clock determines how fast the images are printed across the display as it provides printing coordinates for the incoming one-dimensional images. There are many ways to build counter mechanisms in Max/MSP/Jitter, but I decided to use MSP in this experiment because it's a fast(!), stable, highly controllable and quite reliable clock source.

<figure style="float: left; margin-right: 30px;">
  <img src="/assets/img/2020_08_04_spectral_audio_status.jpg"
  alt="motion average image" title="motion average image" width="250" />
  <figcaption></figcaption>
</figure>

The ```I/O Vector Size``` controls the number of samples that are transferred to and from the audio interface at any one time. Therefore, a smaller I/O vector size will consume a greater percentage of the computer's CPU resources. However, we don't want this value to be too big either as it can stall other computational processes. So to make sure the audio rate was stable but not too demanding, I set the I/O vector size to a reasonable 1024.

The ```Signal Vector Size``` in Max/MSP/Jitter determines the number of samples that are calculated by MSP objects at any one time. Although [**it's documented**](https://docs.cycling74.com/max5/tutorials/msp-tut/mspaudioio.html) that variations in signal vector size only have slight effects on CPU performance, a larger vector size will nevertheless reduce some load. Considering that the systems opt for a maximum framerate of 60 FPS (one frame every 16ms), I could theoretically bring my signal vector size up to 256 ("outputting" a new vector every 5,8ms) without problems, given a sampling rate of 44.1Khz.

In addition to having a fixed I/O vector size, signal vector size and sampling rate, each system used internal sound cards as main audio drivers. With these audio parameter settings I could be confident that the frame rates would never exceeded the counter signal rate. However, there is more to be said about how the counter mechanism is designed. More on this in the upcoming section.

# System Design

Creating a spectral mean image system requires much more than just a process which calculates the mean of an image. I will therefore start by presenting how some of the more general features are designed in the **isolated system configurations**, before I go in-depth on how each of the different mean calculating methods are constructed.

### Overview

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_system_overview.jpg"
  alt="System overview" title="System Overview" width="auto" />
  <figcaption>System overview. The objects in blue are custom abstractions which grab the video-feed (at.webcam), create a motion image (at.vg.mg.selector), provides a counting mechanism (at.imaginary.fft.counter) and finally processes the mean images (at.gl.mg.cpu/gpu). All systems are created in Max/MSP/Jitter version 8.1.5</figcaption>
</figure>

In the image above, notice the ```[jit.gl.layer]``` and ```[jit.world]``` objects. I've set the systems maximum framerate by feeding a value to the @fps attribute in the [jit.world] object. Alternatively, I could set the @interval attribute to 17 for the same effect, a value that represents how many milliseconds between consecutive frames. More so, It's important to disable sync by adding @sync 0. The sync attribute let's us sync our image rendering to the fresh rate of our monitors, a feature I don't want in my particular case. The [jit.gl.layer] is where the image is sent to the [jit.world] to be rendered on the screen. Here, I specify that I want to preserve the aspect ratio of the incoming image in the rendered display window (@perserve_aspect 1) and that I want the image to be automatically scaled to fit the entire display window at any given time (@transform_reset 2).

<!--*
overdrive off(!) because we are doing video processing.
-->

To access my machines web-camera, and to make it output video content in a OpenGL context, I use the ```[jit.qt.grab]```object with an @output_texture attribute set to 1. This object is located inside the blue [at.webcam] abstraction, shown in the image above. At this stage, I also enable the @unique attribute to makes sure no identical frames are outputted, a feature that is important for the next stage of creating a motion image.

### Motion image

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_motion_image.jpg"
  alt="Motion Image" title="Motion Image" width="auto" />
  <figcaption>To create a motion image we calculate the difference between two frames. This image is a representation of what's inside the [at.vg.mg.selector] abstraction, seen in blue in the overview image above.</figcaption>
</figure>

As mentioned, a motion image is the result of calculating the difference between two consecutive frames. To do this in an OpenGL context, we can split and flip the order of frames before passing them through a [absdiff] calculation inside a ```[jit.gl.pix]``` object, as the image above shows. Having the @unique attribute enabled in the [jit.qt.grab] is essential because we never want to calculate the difference between two equal frames. The would result in an undesired amount of black frames between our rendered image frames.

At this stage I also added 2 optional noise filters (binary and unary) that are used to optimize the motion image. Finally, I set the systems data type to float32 (storing 32-bit floating point values) to ensure greater processing precision down the line.

### Printing Rate

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_imaginary_fft.jpg"
  alt="Imaginary FFT" title="Imaginary FFT" width="auto" />
  <figcaption>By making the counter length/size the same as the horizontal dimension (X-dim) of the rendered image, I can control the printing rate by adjusting the X-dim size. This image is a representation of what's inside the [at.imaginary.fft.counter] abstraction, seen in blue in the overview image above.</figcaption>
</figure>

For simplicities' sake, I decided to adapt a technique I used in my [**AudioVideoAnalysis**](https://aleksandertidemann.github.io/projects/2020/07/24/audiovideoanalysis.html) application to manage the printing rate of my spectral images. This design is based on MSP (audio signals) and uses specific FFT parameters, originally used to control the printing rate of a spectrogram, as its points of reference.

In this method, I can adjust the rate of the printing by manipulating a dial that outputs normalized values ranging from 0 to 1. These values are then used to configure the size of the horizontal dimension (X-dim) of the rendered image, as well as to extract the printing time (in seconds). However, this requires us to have a fixed X-dim range, a range which I sat from 100 to 1024.

Furthermore, I set the fundamental MSP ```[count~]``` object to count between a range of 0 and 5291926,6 because this number divided by 5167.9 (an essentially arbitrary number) amounts to a counting range from 0 to 1024 (our max X-dim) where one revolution is made every 60 seconds, presuming the samplerate is 44.1Khz. Next I compensated for an FFT Hopsize of 4 by simply multiplying the entire signal by 2, before allowing the maximum value of the counting range to de decided by the newly configured X-dim size.

The overall take-away here is that I adjust the printing rate by effectively "zooming in" on the spectral images.

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_printing.jpg"
  alt="Printing" title="Printing" width="auto" />
  <figcaption></figcaption>
</figure>

The final counter values are then sent to another ```[jit.gl.pix]``` object where they decide the specific horizontal location of the incoming spectral mean images/frames. The outputted image is then looped back into the [jit.gl.pix], as well as to the display window, to create the desired effect of an images printed across a canvas over time.

## XRAY method

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_cpu_xray_method.jpg"
  alt="cpu xray method" title="cpu xray method" width="auto" />
  <figcaption>This method uses [jit.gl.asyncread] and [xray.jit.mean] in succession to readback a GPU image texture to the CPU to perform mean calculations. This image is a representation of what's inside the [at.gl.mg.cpu] abstraction, seen in blue in the overview image above</figcaption>
</figure>

As briefly mentioned in the Methods section, this system uses a ```[xray.jit.mean]``` object to process the mean calculations on the CPU. This object is not part of the standard Max/MSP/Jitter library so it has to downloaded as part of the "XRAY package" from Max' dedicated package manager. Additionally, I used an object called ```[jit.gl.asyncread]``` that is specifically design to handle readbacks from an OpenGL framebuffer and is therefore a considerable advantage to any video processing chain that seeks to migrate stuff from the GPU to the CPU.

Finally I send the video processing back to the GPU by feeding the CPU matrix into a ```[jit.gl.texture]``` harboring our the vertical dimension of my current web-camera resolution and the horizontal dimension controlled by my current printing rate.

## GPU method

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_gpu_method.jpg"
  alt="gpu method" title="gpu method" width="auto" />
  <figcaption>This method does pixel mean calculations on the GPU by using a for-loop inside a [jit.gl.pix]. This image is a representation of what's inside the [at.gl.mg.gpu] abstraction, seen in blue in the overview image above</figcaption>
</figure>

As you can tell from the image above, the **GPU method** is almost identical to the XRAY one only replacing the [jit.gl.asyncread] and [xray.jit.mean] with a ```[jit.gl.pix]``` object. Since there are no Gen objects in Max that does specifically what [xray.jit.mean] does, I figured that the best way of recreating its functionality was to hard-code something inside the [jit.gl.pix] using a codebox object.

We can code inside MaxMSP's Gen environment using the GenExpr programming language. Luckily, at this stage, I was fortunate enough to receive some aid from a great 3D graphics artist and avid Max/MSP/Jitter user named [**Federico Foderero**](https://www.federicofoderaro.com/). Federico helped me with the specific syntax needed for these lines of code and later expanded on my idea of creating an OpenGL equivalent of ```[xray.jit.mean]``` himself in [**this video**](https://www.youtube.com/watch?v=hcHyiSKLpUA&list=UUvDUaH2fbXP_Yc5Lc9UXfqA). To do the necessary mean calculations on OpenGL textures we can use this GenExpr code:

```
main = vec(0,0,0,0);

if (cell.x) {
  for (i=0; i<dim.x; i+=1) {
    main += nearest(in1, vec(i/dim.x, norm.y));
  }
  main /= dim.x;		
}
out1 = main;
```
The code iterates through every row of an incoming frame and gathers all its pixel information in a vector (main), before finally dividing the vector by the size of the horizontal dimension (dim.x).

## PIX method
The **PIX method** is somewhat of a hybrid between the XRAY and GPU methods. The idea here is to basically implement the same **GPU method**, with a GenExpr for-loop inside a codebox object, but run it on the CPU instead. We can easily do this in Max by removing the "gl" tag in the [jit.gl.pix] object to create a ```[jit.pix]```object. However, as mentioned, it's important we use a [jit.gl.asyncread] whenever you want to migrate operations from the GPU to the CPU. The general recipe of this mean calculating method looks like this:

```
[jit.gl.asyncread] //Readback from GPU to CPU
[jit.pix] //with the GenExpr for-loop inside
[jit.gl.texture] //back to the GPU
```

This method also gives me the opportunity to closer inspect how the GenExpr language performs and if there's any significant difference when executing it on the CPU vs. the GPU.

# Results

Tested each system in both configurations on two machines, rated the overall performance of each system based on its FPS reading, image clarity (missing frames), machine load and how well it does on low battery conditions.  







The downsampling of my test applications is a bit crude. If its worth investigating more how this should relate to the chosen resolution, it should downsample by a factor of the dimensions.
320 240. Its about a third.
16:9 =1280x720
