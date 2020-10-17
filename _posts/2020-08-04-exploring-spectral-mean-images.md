---
layout: post
date: 2020-08-04
title: "Exploring Spectral Mean Images in Max/MSP/Jitter using OpenGL"
image: /assets/img/2020_08_04_spectral_main_gif.gif
categories: General
excerpt: "What is the most effective system for real-time spectral video processing in MaxMSP?"
Keywords: MaxMSP, OpenGL, Jitter, spectral mean images, analysis, motiongram, videogram
---
<!--*
-->
<figure style="float: none">
   <img src="/assets/img/2020_08_04_spectral_main_gif.gif" alt="main"
   title="main" width="640" />
   <figcaption></figcaption>
</figure>

As the title might suggest, in this post I investigate what the most effective system is for real-time spectral video processing in Max/MSP/Jitter using an OpenGL framework. I was motivated to explore this specific topic in-depth after running into some challenges while redeveloping an application that analyzes the spectral content of video and audio for the [**RITMO Centre for Interdisciplinary Studies in Rhythm, Time and Motion**](https://www.uio.no/ritmo/english/) at the University of Oslo, called AudioVideoAnalysis. I've written [**an entire post**](https://aleksandertidemann.github.io/projects/2020/07/24/audiovideoanalysis.html) about this application earlier so be sure to check it out if you're interested.

My investigation consists of an experiment where I designed and tested several of these OpenGL spectral mean image-producing systems in Max/MSP/Jitter to determine which was most effective. After briefly elaborating on the background and method of this experiment, I will discuss the results before finally presenting the winning system(!)

# Background
Spectral mean images are representations of movement over time. These images are created by calculating vector mean values of either every row or column of an incoming matrix (depending on if we want to represent time on the horizontal or vertical axis) before printing the outputted one-dimensional matrices across a canvas.

<figure style="float: none">
   <img src="/assets/img/2020_08_04_spectral_motiongram_diagram.jpg" alt="motiongram diagram"
   title="motiongram diagram" width="640" />
   <figcaption>Simplified(!) image rendition of the vector mean process in question. When we process mean values along every row of a frame, we can use to create a spectral motion image with time on the horizontal axis. Since the alpha plane is a constant (1) its not included in this image.</figcaption>
</figure>

If we produce a spectral mean image from a motion image, we have something called a motiongram. We create a motion image by extracting the difference between two sequential frames of a video. This will render only the movement of the video in color while leaving the rest of the image black. The idea and concept behind motiongrams have been thoroughly researched and explored by professor [**Alexander Refsum Jensenius**](http://people.uio.no/alexanje) at the University of Oslo. You can read much more about this research side of videograms and motiongrams [**in this article**](https://www.duo.uio.no/handle/10852/26907).  

<figure style="float: none">
   <img src="/assets/img/2020_08_04_spectral_motiongram_preview.jpg" alt="motiongram preview"
   title="motiongram preview" width="auto" />
   <figcaption>Three horizontal motiongrams. Image retrieved from <a href="https://www.hf.uio.no/imv/english/research/projects/mg/subprojects/motiongrams/"><b>UiO</b></a></figcaption>
</figure>

In the outdated version of AudioVideoAnalysis, the application I was tasked to whip into shape, this mean-calculating process was handled by a MaxMSP external object called ```[xray.jit.mean]```. The XRAY external package was developed by Wesley Smith ([**read more here**](https://cycling74.com/forums/get-the-xray-package-by-wesley-smith/)) and is a set of jitter objects that do video processing on the CPU, the same way every other jitter object in MaxMSP operates. However, when dealing with graphics and video processing in MaxMSP today, it's highly recommended you "wrap your code" in something called an OpenGL context that enables your GPU to take care of all the graphical processing. OpenGL (Open Graphics Library) is a cross-platform graphics interface for 3D and 2D graphics and is often considered an industry-standard in digital graphics circles. Because of this, I wanted the new version of AudioVideoAnalysis to have OpenGL image rendering. However, this gave rise to an interesting problem:

> **Should an OpenGL system that generates real-time spectral mean images (or motiongrams) in Max/MSP/Jitter do vector mean calculations on the CPU or the GPU?**

# The Problem Explained
When doing computationally expensive tasks like calculating thousands (if not millions) of mathematical operations many times per second, as our system demands, the general norm is that these operations should be executed on the CPU. This is because the CPU is specifically designed for these kinds of tasks while the GPU is more optimized to deal with graphics. [**There's a great video**](https://www.youtube.com/watch?v=V3_p9R7YG-g) on the pros and cons of processing video in Max/MSP/Jitter on the CPU vs. the GPU by Federico Foderero. Be sure to check it out if you're interested in learning more.

With these facts considered, it seems obvious that the mean calculations in question should be executed on the CPU. However, according to the official documentation on the [**Best Practices in Jitter**](https://cycling74.com/tutorials/best-practices-in-jitter-part-1), there is reason to doubt that this is best practice in our particular case. The Cycling74 article states that if you transition from OpenGL jitter to regular jitter (from the GPU to the CPU) you should use specific readback and downsampling methods that minimize data copies between the processing units. What this essentially means is that you risk creating a bottleneck in your processing chain if you continually transition heavy loads between units.

Therefore, with this risk in mind, I wondered whether keeping all of the processing for my application on the GPU would be more effective than migrating some of the math-heavy tasks to and from the CPU.

# Method
To explore this question, I designed an experiment with the explicit purpose of testing this assumption further. The experiment consisted of testing three different OpenGL spectral mean image systems in two different configurations on two different machines. Testing was conducted on one system at a time (on both machines) in two rounds. First-round with a high-resolution image (1280x720) and the second round with a much lower resolution (320x240).

### Systems
1. The first system uses a readback method to process the mean calculation on the CPU via ```[xray.jit.mean]```. From now, we refer to this system as the **XRAY method**.  

2. The second system uses a ```[jit.gl.pix]``` (an OpenGL object in MaxMSP) and some GenExpr code to process the mean calculations on the GPU. From now, we refer to this system as the **GPU method**.  

3. The third system uses the same readback method as system one, but instead uses a ```[jit.pix]``` (a regular jitter object in MaxMSP) with the same GenExpr code from system two to process the mean calculation on the CPU. From now, we refer to this system as the **PIX method**.

### Configurations

1. In the first configuration, we test the systems on their own. In other words, in an isolated fashion. From now, we refer to this configuration as an **isolated configuration**.

2. In the second configuration, we test the systems as part of a bigger system. Here, I modified three versions of the AudioVideoAnalysis 2.0, each with a dedicated system. From now, we refer to this system as the **AVA configuration**.

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
Since one of my goals was to find a system that would be "universally" stable, I decided to only use middle/low-grade machines with standard integrated Intel Graphics cards in testing. However, having the ability to switch to an NVIDIA Quadro T2000 GPU on my Lenovo Thinkpad will be valuable to get better perspectives on the performance of particular systems.

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
* **High and low-resolution video** - To generate video-feed, I used the dedicated web-camera on each machine. For consistency, I conducted all tests with the same high-resolution image (1280x720). Additionally, each system in each configuration had a *downsample* trigger that downsampled the video-feed (1280x720) to 320x240. The purpose of this downsampling is to help generate a more holistic and better understanding of how each method performs overall.  

  * **Performance on low battery** - I checked how each system behaved on machines with low-battery because laptops often behave and prioritize differently when in this condition. More so, the better a system performs on low battery the more stable it is. A high score on this condition means the system performed WELL on low-battery.

  * **General GPU/CPU load** - An important (if not the most important) part of the testing was to monitor how each system stressed its dedicated processing unit. A high score in this category means the system did NOT stress the unit much.

  * **Various audio parameter settings** - I use audio (MSP) signals as system clocks. Therefore, MSP controls the printing rate of our systems. We should therefore test whether adjusting some parameters, like the I/O Vector Size and the Vector Size, affects performance in any way.

  * **FPS (Frames Per Second)** - The FPS value will always be closely linked to the *GPU/CPU load* and *audio parameter* categories. The higher the FPS, the better the system is. However, I designed each system to have a maximum limit of 60 FPS. It's important to have a max limit because we want to be able to design a clock source whose rate we can ensure will always exceed the current FPS value.

  * **Image Clarity** - A well-performing system in my experiment is a system that shows few or no lags, does not crash (often), and most importantly has a low occurrence of missing/blank frames. These aesthetical aspects are what I evaluate and rate through the Image Clarity category.

### Clock
As briefly mentioned, the systems use MSP (audio) signals as their main clock source/counter mechanism. Therefore, it's essential to define a set of fixed audio parameter settings beforehand. The clock determines how fast the images are printed across the display as it provides printing coordinates for the incoming one-dimensional images. There are many ways to build counter mechanisms in Max/MSP/Jitter, but I decided to use MSP in this experiment because it's a fast(!), stable, highly controllable and quite reliable clock source.

<figure style="float: left; margin-right: 30px;">
  <img src="/assets/img/2020_08_04_spectral_audio_status.jpg"
  alt="motion average image" title="motion average image" width="250" />
  <figcaption></figcaption>
</figure>

The ```I/O Vector Size``` controls the number of samples that are transferred to and from the audio interface at any one time. Therefore, smaller I/O vector sizes consume more CPU resources. However, we don't want this value to be too big either as it can stall other computational processes. So to make sure the audio rate was stable but not too demanding, I set the I/O vector size to a reasonable 1024.

The ```Signal Vector Size``` in Max/MSP/Jitter determines the number of samples that are calculated by MSP objects at any one time. Although [**it's documented**](https://docs.cycling74.com/max5/tutorials/msp-tut/mspaudioio.html) that variations in signal vector size only have slight effects on CPU performance, a larger vector size will nevertheless reduce some load. Considering that the systems opt for a maximum framerate of 60 FPS (one frame every 16ms), I could theoretically bring my signal vector size up to 256 ("outputting" a new vector every 5,8ms) without problems, given a sampling rate of 44.1Khz.

In addition to having a fixed I/O vector size, signal vector size and sampling rate, each system used internal sound cards as main audio drivers. With these audio parameter settings, I could be confident that the frame rates would never exceed the counter signal rate. However, there is more to be said about how the counter mechanism is designed. More on this in the upcoming section.

# System Design

Creating a system that explores spectral mean images requires more than just a method for vector mean-calculation. In this section, I will explain which key processes are necessary to build these systems, with the **isolated system configuration** as my focal point, before I go in-depth on how each of the different mean-calculating methods are constructed.

### Overview

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_system_overview.jpg"
  alt="System overview" title="System Overview" width="auto" />
  <figcaption>System overview. The objects in blue are abstractions which grab the video-feed (at.webcam), create a motion image (at.vg.mg.selector), provide a counting mechanism (at.imaginary.fft.counter) and finally calculates the vector means (at.gl.mg.cpu/gpu). All systems are created in Max/MSP/Jitter version 8.1.5</figcaption>
</figure>

In the image above, notice the ```[jit.gl.layer]``` and ```[jit.world]``` objects. By setting the @fps attribute to 60 in the [jit.world] object, I can determine the systems' maximum framerate. Alternatively, I could set the @interval attribute to 17 that would determine the maximum time interval (in milliseconds) between consecutive frames. The [jit.gl.layer] object sends content that is to be rendered on the screen to the [jit.world]. Here, It's important to make explicit that I want to preserve the aspect ratio of the incoming image (@perserve_aspect 1) and that I want the image to be automatically scaled to fit the entire display window at any given time (@transform_reset 2).

<!--*
overdrive off(!) because we are doing video processing.
-->

To access the web-cameras, and to make it output video information in an OpenGL context, I use the ```[jit.qt.grab]```object with the @output_texture attribute set to 1. This object is located inside the blue [at.webcam] abstraction, shown in the image above. At this stage, I also enable the @unique attribute to makes sure no identical frames are outputted, a feature that's important for the next step which is creating a motion image.

### Motion image

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_motion_image.jpg"
  alt="Motion Image" title="Motion Image" width="auto" />
  <figcaption>To create a motion image we calculate the difference between two frames. This image is a representation of what's inside the [at.vg.mg.selector] abstraction, seen in blue in the overview image above.</figcaption>
</figure>

As mentioned, a motion image is a result of calculating the difference between two consecutive frames. To do this in an OpenGL context, we can split and flip the order of frames before passing them through a [absdiff] object inside a ```[jit.gl.pix]``` object. Having the @unique attribute enabled in the [jit.qt.grab] is essential because we never want to calculate the difference between two equal frames. This would result in an undesired amount of black/missing frames in our rendered spectral image.

At this stage, I also added 2 optional noise filters (binary and unary) that can be used to optimize the quality of the motion image. Finally, I set the systems data type to float32 (storing 32-bit floating-point values) to ensure greater processing precision down the line.

### Printing Rate

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_imaginary_fft.jpg"
  alt="Imaginary FFT" title="Imaginary FFT" width="auto" />
  <figcaption>By making the counter length/size the same as the horizontal dimension (X-dim) of the rendered image, I can control the printing rate by adjusting the X-dim size. This image is a representation of what's inside the [at.imaginary.fft.counter] abstraction, seen in blue in the overview image above.</figcaption>
</figure>

For simplicities' sake, I decided to adopt a technique I used in the [**AudioVideoAnalysis application**](https://aleksandertidemann.github.io/projects/2020/07/24/audiovideoanalysis.html) to manage the printing rate of my spectral images. This design is based on MSP (audio signals) and uses specific FFT parameters, originally used to control the printing rate of a spectrogram, as its points of reference.

In this method, I can adjust the rate of the printing by manipulating a dial that outputs normalized values ranging from 0 to 1. These values are then used to configure the size of the horizontal dimension (X-dim) of the rendered image, as well as to extract the printing time (in seconds). However, this requires us to have a fixed X-dim range, a range which I determined to be from 100 to 1024.

Furthermore, I set the fundamental MSP ```[count~]``` object to count between a range of 0 and 5291926,6 because this number divided by 5167.9 (an essentially arbitrary number) amounts to a range from 0 to 1024 (our max X-dim) where one cycle is made every 60 seconds, presuming the sample rate is 44.1Khz. Next, I compensated for an FFT Hopsize of 4 by simply multiplying the entire signal by 2 before allowing the maximum value of the counting range to de decided by the newly configured X-dim size.

The overall takeaway here is that I adjust the printing rate by effectively "zooming in" on the spectral images. Not a particularly elegant method, but an effective one.

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_printing.jpg"
  alt="Printing" title="Printing" width="auto" />
  <figcaption></figcaption>
</figure>

The final counter values are then sent to another ```[jit.gl.pix]``` object where they decide the specific horizontal locations for the incoming spectral mean images/frames and produce the desired effect of an image printed across a canvas over time.

## XRAY method

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_cpu_xray_method.jpg"
  alt="cpu xray method" title="cpu xray method" width="auto" />
  <figcaption>This method uses [jit.gl.asyncread] and [xray.jit.mean] in succession to readback an openGL texture to the CPU before performing the necessary mean calculations there. This image is a representation of what's inside the [at.gl.mg.cpu] abstraction, seen in blue in the overview image above</figcaption>
</figure>

As briefly mentioned in the Methods section, the **XRAY method** system uses a ```[xray.jit.mean]``` object to process vector mean-calculations on the CPU*. Also, I used an object called ```[jit.gl.asyncread]``` that is specifically design to handle readbacks from an OpenGL framebuffer and is therefore a considerable advantage to any video processing chain that seeks to migrate stuff from the GPU to the CPU. Finally, I send the video processing back to the GPU by feeding the CPU matrix into a ```[jit.gl.texture]``` harboring the vertical dimension of my current web-camera resolution and the horizontal dimension controlled by my current printing rate.

*
<sub>This object is not part of the standard Max/MSP/Jitter library so it has to be downloaded as part of the "XRAY package" from Max' dedicated package manager.</sub>

## GPU method

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_gpu_method.jpg"
  alt="gpu method" title="gpu method" width="auto" />
  <figcaption>This method does vector mean calculations on the GPU by using a for-loop inside a [jit.gl.pix]. This image is a representation of what's inside the [at.gl.mg.gpu] abstraction, seen in blue in the overview image above</figcaption>
</figure>

As you can tell from the image above, the **GPU method** is almost identical to the **XRAY method**, except the [jit.gl.asyncread] and [xray.jit.mean] objects are replaced with a ```[jit.gl.pix]``` object. Since there are no Gen objects in Max that specifically does what [xray.jit.mean] does, I figured that the best way of recreating its functionality was to hard-code something inside the [jit.gl.pix] using a codebox object.

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

## PIX method
The **PIX method** is somewhat of a hybrid between the XRAY and GPU methods. The idea here is to implement the same **GPU method**, with a GenExpr for-loop inside a codebox, but run it on the CPU instead. We can easily do this in Max by removing the "gl" tag in the [jit.gl.pix] object to create a ```[jit.pix]```object. However, as mentioned, we must use a [jit.gl.asyncread] whenever we want to migrate operations from the GPU to the CPU. The general recipe of this mean calculating method looks like this:

```
[jit.gl.asyncread] //Readback from GPU to CPU
[jit.pix] //with the GenExpr for-loop inside
[jit.gl.texture] //back to the GPU
```

This method also allows me to closer inspect how the GenExpr language performs and if there's any significant difference when executing it on the CPU vs. the GPU.

# Results
When testing each method, I rated them on a scale from 0-10 in each test conditions except for the audio parameter category. In this category, a rating of 1 indicates *"yes, changing the audio settings did affect performance"* while 0 indicates *"no, changing the audio settings did not affect performance"*.

## XRAY Method

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_xray_results.png"
  alt="pix results" title="pix results" width="auto" />
  <figcaption>Test results for using the XRAY method to calculate mean values when processing spectral images. The higher the score, the better the performance. </figcaption>
</figure>

Not surprisingly, the overall performance of the **XRAY method** was good in both configurations. However, when displaying high-resolution images (1280x720) in the AVA configuration, its performance did depended on which machine I was using. On my early 2011 Macbook pro, processing high-resolution images in this configuration resulted in a steady accumulation of blank/missing frames here and there. However, on my more powerful Thinkpad, both resolutions created usable and consistent motion images overall.  

But regardless of some performance discrepancies when pushing the system quite hard, the overall load of the **XRAY method** applications was surprisingly low with only a 5-10% increase in CPU utilization from processing downsampled (320x240) to high-resolution images (1280x720). Similarly, running the applications on machines with low battery had little or no effect on their performance.  

<figure style="float: left; margin-left: 10px;">
  <img src="/assets/img/2020_08_04_spectral_xray_ava_downsample_load.jpg"
  alt="xray-ava-downsampled-load" title="xray-ava-downsampled-load" width="320" />
  <figcaption>Processing 320x240 images with the XRAY method in the AVA configuration used 25% of my Thinkpad P53's CPU resources and about 70% of its GPU resources. Similar results were also documented on my Macbook pro.</figcaption>
</figure>

<figure style="float:none">
  <img src="/assets/img/2020_08_04_spectral_xray_ava_highres_load.jpg"
  alt="xray-ava-highres-load" title="xray-ava-highres-load" width="320" />
  <figcaption>When processing high resolution images (1280x720) with the XRAY method in the same system configuration, there was only a 5% increase in CPU usage. Similar results were documented on my Macbook pro. </figcaption>
</figure>

But despite these relatively low CPU loads, the systems did not show very promising framerates. When processing low-resolution images the lowest recorded framerate was around 14/15, which is very much usable, but on higher resolution images the lowest FPS reading was down at almost 6.

An interesting discovery was that changing the ```I/O Vector Size``` and ```Signal Vector Size``` had a noticeable impact on performance. This was visible and consistent when processing high-resolution images in the AVA configuration. The behavior was that increasing the values of these audio settings effectively reduced the number of lost frames (blank spots), which makes perfect sense considering that higher vector sizes should reduce CPU load.

In terms of image quality, there was no advantage of rendering spectral images from a high-resolution video (1280x720), other than the occasional missing frame, as opposed to a downsampled (320x240) video-feed. The only factor that had any impact on the displayed image resolution was increasing the temporal resolution, or printing rate, of the images. An expected feature and side-effect of my system design.

<figure style="float: left; margin-left: 10px;">
  <img src="/assets/img/2020_08_04_spectral_xray_display_downsampled.jpg"
  alt="xray-ava-downsampled-display" title="xray-ava-downsampled-display" width="350" />
  <figcaption>Downsampled motiongram (320x240) using the XRAY method in the AVA configuration.</figcaption>
</figure>

<figure style="float:none">
  <img src="/assets/img/2020_08_04_spectral_xray_display_highres.jpg"
  alt="xray-ava-highres-display" title="xray-ava-highres-display" width="320" />
  <figcaption>High resolution motiongram (1280x720) using the XRAY method in the AVA configuration.</figcaption>
</figure>

## GPU Method

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_gpu_results.png"
  alt="gpu-results" title="gpu-results" width="auto" />
  <figcaption>Test results for using the GPU method to calculate mean values when processing spectral images. The higher the score, the better the performance.</figcaption>
</figure>

Overall, the **GPU Method** showed some promising results when processing downsampled video-feeds. When processing high-resolution images the system struggled to produce usable images, regardless of configuration or machine.

Interestingly, the GPU load of processing a downsampled video-feed with the **GPU Method** corresponded to the GPU load of processing both downsampled and high-resolution video-feeds with the **XRAY method**, except it used *less* of the CPU's resources in the process. However, increasing the image resolution with the **XRAY method** did not amount to any significant increase in GPU utilization. In the **GPU method**, this increase in image resolution vastly increased the GPU utilization, effectively maxing out its resources pretty quickly.

<figure style="float: left; margin-left: 10px;">
  <img src="/assets/img/2020_08_04_spectral_gpu_ava_downsample_load.jpg"
  alt="gpu-ava-downsampled-load" title="gpu-ava-downsampled-load" width="320" />
  <figcaption> Processing 320x240 images with the GPU method in the AVA configuration used about 70% of my Thinkpad P53's GPU resources, and 19% of its CPU resources. Similar results were also documented on my Macbook pro.</figcaption>
</figure>

<figure style="float:none">
  <img src="/assets/img/2020_08_04_spectral_gpu_ava_highres_load.jpg"
  alt="gpu-ava-highres-load" title="gpu-ava-highres-load" width="320" />
  <figcaption>When processing high resolution images (1280x720) with the GPU method in the same configuration, the GPU resources maxed out and used consumed 11% of the CPU resources. Similar results were documented on my Macbook pro. </figcaption>
</figure>

<br>

Similar to the **XRAY method**, the FPS readings of the **GPU method** correlated strongly with the overall system load. That is, the higher the resolution of the video-feed the lower the frame rates. The lowest recorded FPS of the **GPU method** in "HD mode" was as low as 3, while it remained pretty consistently at 20 when processing lower resolution image feeds.

In terms of image quality, I noticed the same disadvantage of rendering higher resolution images as experienced in the **XRAY Method**. However, it was clear that this method produced a higher frequency of missing frames, regardless of video resolution.

<figure style="float: left; margin-left: 10px;">
  <img src="/assets/img/2020_08_04_spectral_gpu_display_highres.jpg"
  alt="GPU-isolated-highres-display" title="GPU-isolated-highres-display" width="320" />
  <figcaption>High resolution motiongram (1280x720) using the GPU method in the isolated configuration.</figcaption>
</figure>

<figure style="float:none">
  <img src="/assets/img/2020_08_04_spectral_gpu_display_downsampled.jpg"
  alt="GPU-isolated-downsampled-display" title="GPU-isolated-downsampled-display" width="320" />
  <figcaption>Downsampled motiongram (320x240) using the GPU method in the isolated configuration.</figcaption>
</figure>

<br>
<br>
<br>

## PIX Method

<figure style="float: none">
  <img src="/assets/img/2020_08_04_spectral_pix_results.png"
  alt="gpu-results" title="gpu-results" width="auto" />
  <figcaption>Test results for using the PIX method to calculate mean values when processing spectral images. The higher the score, the better the performance.</figcaption>
</figure>

Another surprising finding was how awful the **PIX method** performed considering it used the same code as the **GPU method** only running on the CPU instead. The systems instantly crashed when processing high-resolution video-feeds with this method. More so, testing showed that downsampling the video-feed had little or no positive effect on the overall load.

<figure style="float: left; margin-right: 20px;">
  <img src="/assets/img/2020_08_04_spectral_pix_downsampled_load.jpg"
  alt="pix-downsampled-load" title="pix-downsampled-load" width="320" />
  <figcaption> Processing 320x240 images with the PIX method in the isolated configuration used close to 90% of my Thinkpad P53's CPU resources. Similar results were also documented on my Macbook pro. </figcaption>
</figure>

This method was so stressful on my machines that it was challenging to get any data at all. When processing a downsampled video-feed on my Thinkpad I was able to register a stable (and whopping) 88% CPU utilization, which effectively explains the awful behavior. What's interesting here is that this might shed some light on how the GenExpr environment works and what kinds of processes it's optimized for. On the other hand, it might suggest that my code was bad...   

<br>
<br>
<br>
<br>
<br>
<br>

# Conclusion
First of all, these findings should not be interpreted as being statistically significant in any way due to the small size of the experiment. More so, it's worth mentioning that the experiment does not heed to a particularly high scientific standard as it does not factor in or control a vast range of variables that might have contributed to its results. This being said, the findings do provide some valuable insights into how we should design a system that generates spectral mean images in Max/MSP/Jitter.

To briefly recap, in this experiment I sought to determine whether an OpenGL spectral mean image system in Max/MSP/Jitter should do vector mean calculations on the CPU or the GPU by testing three different such methods (XRAY, GPU and PIX method), in two different configurations (isolated and AVA), on two different machines (Thinkpad P53 and Macbook Pro 15" early 2011 model). The findings indicate that the **XRAY method** was the most stable and reliable system. However, the **GPU method** can be a better alternative in some cases as it performs equally well on low-resolution images while putting less stress on the machine's CPU. So if you want a stable and highly controllable system that can accommodate various resolutions you should opt the for **XRAY method**. On the other hand, if you're only going to process relatively low-resolution images and want to utilize most of your CPU power elsewhere, you should opt for the **GPU method**.

The findings also revealed some interesting limitations with the code used to generate the mean images in the GPU and PIX methods. These limitations might be attributed to the nature of the GenExpr language itself, it's speed and so on, or might simply be a side-effect of a sub-optimal code structure (basically bad coding on my part). Unfortunately, I found too little technical documentation to be able to rule out or confirm any of these assumptions.  

## Algorithmic Downsampling
The findings also suggest that we should always seek to downsample our video-feed before doing mean calculations, regardless of whether we use an XRAY or GPU method to generate our spectral images. This downsampling process is also a [**well-documented practice**](https://cycling74.com/tutorials/best-practices-in-jitter-part-1). The downsampling method I used in this experiment was pretty crude as it always downsampled the incoming image to 320x240. This is not very elegant because we might run into scenarios where we want to process video with even smaller dimensions, or where we want to have the opportunity to process high-resolution images for various reasons. In this case, it's worth briefly looking at how we can design a more dynamic and permanent **downsampling method**.

To illustrate how we could take a more algorithmic approach to this downsampling, I designed a simple program in GenExpr that downsamples image dimensions based on a few conditions:

* **If** the horizontal dimension is larger than the vertical dimension, **and** 1/3 of the horizontal dimension is *greater* than 320, **then** we scale the horizontal dimension to 1/3 of its original size and scale the vertical dimension to the value that ensures we preserve the aspect ratio of the incoming image.
* However, **if** the horizontal dimension is larger than the vertical dimension, **and** 1/3 of the horizontal dimension is *smaller* than 320, we do nothing.

The process is the same if the vertical dimension is larger than the horizontal dimension, except we use the vertical dimension as "the basis" for the aspect ratio and take 240 as the standard as opposed to 320. Here's what this translates to in GenExpr code:

```
xdim, ydim = 0; //in1 is the X-dim, and in2 is the Y-dim

if (in1 > in2) { //if X-dim is larger than Y-dim.
  if (in1 > 320) {
    if (in1/3 > 320) {
      ydim = (in2 / in1) * (in1/3); //preserving aspect ratio from X-dim
      xdim = in1/3;
      } else {
        ydim = (in2/in1) * 320;
        xdim = 320;
       }
    } else {
      ydim = in2;
      xdim = in1;
    }
  } else { //if Y-dim is larger than X-dim.
    if (in2 > 240) {
      if (in2/3 > 240) {
        ydim = in2/3;
        xdim = (in1/in2) * (in2/3); //preserving aspect ratio from Y-dim
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

## Download

The source code for these spectral mean image systems are readily available at my [**GitHub page**](https://github.com/AleksanderTidemann/opengl-spectral-image-processing), so feel free to also enhance, change and/or explore!
