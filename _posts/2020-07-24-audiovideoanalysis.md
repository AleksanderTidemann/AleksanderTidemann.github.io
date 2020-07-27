---
layout: post
date: 2020-07-24
title: "AudioVideoAnalysis"
image: /assets/img/2020_07_24_ava_main.jpg
categories: Projects
excerpt: "A standalone application for realtime spectral analysis of audio and video, developed in collaboration with RITMO (Center for Interdisciplinary Studies in Rhythm, Time and Motion) and the FourMS lab at the University of Oslo."
---

Welcome to the second installment of a three-part series on my adventures of redeveloping the [**Musical Gestures Toolbox for Max**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/musicalgesturestoolbox/mgt-max/) for the University of Oslo*. This post presents the AudioVideoAnalysis 2.0, a MaxMSP standalone application for realtime spectral analysis of audio and video I developed in collaboration with the [**RITMO center for interdisciplinary studies in rhythm, time and motion)**](https://www.uio.no/ritmo/english/) and their [**FourMS lab**](https://www.uio.no/ritmo/english/research/labs/fourms/) at the University of Oslo, spring 2020.

<figure style="float: left; margin-right: 10px;">
   <img src="/assets/img/2020_07_24_ava_1.jpg" alt="AudioVideoAnalysis 0.1"
   title="AudioVideoAnalysis 0.1" width="auto" />
   <figcaption>AudioVideoAnalysis 0.1</figcaption>
</figure>

AudioVideoAnalysis 0.1 was an application that recorded video/motiongrams and spectrograms from any available media source, developed by [**Alexander Refsum Jensenius**](http://people.uio.no/alexanje) in the early 2000's as a part of the [**Musical Gestures Toolbox for Max**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/musicalgesturestoolbox/mgt-max/). Its intended purpose was to provide a quick, reliable and easy way of analyzing spectral content of both video and audio together on one platform.

My task was to update the application and replace any old dependencies with more long-term and stable solutions. In this process I stayed true to the original design aspects and tried to improved them as much as I could hoping that it would contribute to its main purpose and inspire more people to play with spectral analysis.

*
<sup>Please [**visit my previous post on the VideoAnalysis**](https://aleksandertidemann.github.io/projects/2020/07/23/videoanalysis.html) to learn more about the context and history of these particular development projects.</sup>

# AudioVideoAnalysis 2.0

<figure style="float: none;">
   <img src="/assets/img/2020_07_24_ava_main.jpg" alt="AudioVideoAnalysis 2.0"
   title="AudioVideoAnalysis 2.0" width="auto" />
   <figcaption></figcaption>
</figure>

While (me and) [**Balint Lackzo**](https://github.com/balintlaczko) pushed the capabilities of MaxMSP Jitter with the VideoAnalysis software, we also effectively set the standard for the remaining two MaxMSP-based analysis applications still scheduled for updating, namely AudioVideoAnalysis and AudioAnalysis. Nevermind that the sole objective of these projects was to simply remove any dependencies on "old" MaxMSP externals, once the VideoAnalysis turned out the way it did there was no way the AudioVideoAnalysis could be of any lesser standard. At least in my mind it could not. A decision which resulted in a tiresome, challenging and extremely valuable experience all together.

## Features
Primarily, The AudioVideoAnalysis 2.0 differs from the 0.1 version by having more complex previewing functionalities, both before and after recording, and the ability to export the images. Among these display functionalities, I feel that some are worth elaborating on in more detail before giving a more general list of the applications features.

### Rate of Play
An inevitable issue that arises when trying to make an application that records spectral information from one place to another is how long the printing process should take from start to finish. In other words, what should the temporal resolution (or printing rate) of our images be?

<figure style="float: none">
   <img src="/assets/img/2020_07_24_rate_window.jpg" alt="AVA rate"
   title="rate" width="800" />
   <figcaption>The rate function controls the printing rate and temporal resolution of our images</figcaption>
</figure>

The AudioVideoAnalysis features a ```rate``` dial that enables any user to decide their preferred printing rate and thus the printing time from start (x-axis min) until it returns to start (x-axis max). A size which is further expressed under ```Total Display Time``` as seconds just below. It's worth mentioning that this rate functionality operates by scaling down or up the resolution of our display window from a fixed maximum which is an ```1024 x "user-specified y-dim"``` image with a printing rate of 60 seconds. Therefore, it does not compensate for loss of frames. What this means is while increasing the rate might give you a much higher temporal resolution of your spectral recording, it will effectively decrease the image resolution.

This decrease in image resolution is unfortunate but necessarily arises when we want to ensure a smooth printing process while simultaneously accommodating for various frame-rates on various cameras. However, to compensate for this you can alternatively set a higher video device resolution.

### Analysis Markers
Another mentionable display feature in AudioVideoAnalysis is its various analysis markers. These consists of optional frequency and time markers along the xy-axis, grid view with customizable size and clickpoint data retrieval.

As I introduced features like ```printing rate``` and a ```logarithmic``` dial that gives users the ability to adjust the spectrograms frequency distribution (from linear to logarithmic), the need for a set of analysis markers grew stronger. More so, it's essential to have a frame of reference when doing analysis on spectral images. The result was an implementation of automatically re-scaling frequency and time markers in the display window that users can add or remove from the menubar. Changing the ```Grid Size``` parameter in the UI adds and subtracts the amount of analysis markers visible. Furthermore, by enabling ```View Display Grid``` in the menubar, a grid matrix is added on-top of the images which can further improve our frame of reference.

<figure style="float: none;">
   <img src="/assets/img/2020_07_24_ava_analysis_markers.gif" alt="Grid"
   title="Grid" width="800" />
   <figcaption> spectrogram (audio) in green and motiongram (video) in grayscale. Adding a grid to the display window can further assist analysis by providing a better frame of reference.</figcaption>
</figure>

Finally, I decided to implement something called Clickpoint Data Retrieval, allowing for an even more detailed inspection of the spectral content produced. I first came over this feature in a paper by [**Jean-Francois Charles**](https://www.jeanfrancoischarles.com/) called [**A Tutorial on Spectral Sound Processing Using Max/MSP and Jitter**](https://www.mitpressjournals.org/doi/pdf/10.1162/comj.2008.32.3.87). In this fantastically detailed and informative paper on various ways to generate and view spectral audio content in MaxMSP and Jitter, he introduces a method for retrieving Time (ms), Hz (frequency) and Amplitude (volume) from anywhere in a recorded spectrogram via physically engaging the image with our mouse.

<figure style="float: left; margin-right: 10px;">
   <img src="/assets/img/2020_07_24_ava_clickpoint.jpg" alt="Clickpoint"
   title="Clickpoint" width="300" />
   <figcaption></figcaption>
</figure>

I used Jean-Francois' algorithms as the basis my own implementation of this feature in AudioVideoAnaysis, only reconfiguring the actual text to appear in the display window as opposed to in the main UI, to display seconds instead of milli-seconds, and finally to change color based on the display mode and spectrogram color. The displayed information (specific values in the text) also had to account for the dynamic printing rate and logarithmic feature previously mentioned.

### Image Layering
Being able to record and preview spectral content of both video and audio simultaneously is one of AudioVideoAnaysis' greatest strengths. With this in mind, I thought it would serve well to incorporate multiple display option that enable users to view the images in different configurations. Originally, the application presented the images layered in a cake-like fashion with video at the bottom and the audio directly above it. In the 2.0 version, we can view the images individually (only audio or only video), layered cake-wise (as before) or layered on-top of each other. The latter being the most recent and interesting for exploring relationships between movement and sound.

### General
If you want a more detailed run-through of these features, and how to use the application in general, you can access a in-depth *How To Use* guide online [**from this website**](https://github.com/fourMs/AudioVideoAnalysis/wiki) complete with lots of yummy illustrations and step-by-step instructions. And finally, a more general list of AudioVideoAnaysis' (2.0) most prominent features:

* Realtime preview of visualizations
  * Detached and scalable Display Window
* Create various types of visualizations from any connected sound and video source
  * Motiongrams and Videograms
  * Spectrograms
* Pre-processing
  * Noise reduction
  * Color
  * Audio monitoring
  * Customize analysis audio source input (up to 4 channels).
* Various display parameters
  * Retrieve spectrogram time, amplitude and frequency info at clickpoint.
  * Customize the rate (display length in seconds) of image printing
  * Add analysis grid and markers (frequency and time) with dynamic grid size
* Export the recorded images
* Dynamic control spectrograms logarithmic curve (linear to logarithmic)
* Spectral blurring effect (audio)
* GPU images processing
* In-depth *How to use* guide
* Report issues straight to the developer site

## Downloads
The application is one hundred percent open source and available on both OSX and WIN. It can be downloaded from the following places:
* [**FourMS GitHub page**](https://github.com/fourMs/AudioVideoAnalysis/releases)
* [**RITMO homepage at the University of Oslo**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/AudioVideoAnalysis/)

If you want to contribute to AudioVideoAnaysis' development you are free to do so. Simply fork the [**main AudioVideoAnalysis repo**](https://github.com/fourMs/AudioVideoAnalysis), clone it, develop some awesome stuff and make a pull request for us to review.


# Trias and Tribualations

<figure style="float: none;">
   <img src="/assets/img/2020_07_24_ava_motiongram.jpg" alt="Motiongram"
   title="Motiongram" width="800" />
   <figcaption> Motiongrams generated by AudioVideoAnaysis represent vertical motion (Y-axis) over time (X-axis).</figcaption>
</figure>

The most interesting challenge of this development project was figuring out how to build it in an OpenGL context. What this means is using specific video objects in MaxMSP that does video processing on the GPU rather than on the CPU. This was a highly relevant challenge considering the fact that AudioVideoAnaysis primarily is a realtime application that should have smooth visualizations and user experiences uninterrupted by any glitches or lags. It would also be informative in understanding how we might redesign VideoAnalysis in the future.

While I was able to move all the image processing within an OpenGL context in the end, I specifically struggled with a small but key processes of the application design. An important component of the video/motiongram section of the application is a process that calculates the mean values for every row (x-dimension) in a frame and spits out one-dimensional matrices that we print sequentially along a "canvas" to produce the video/motiongram. When confronted with this specific process I questioned whether doing mean calculations of frames was better on the GPU or the CPU. My worry was doing these calculations on the CPU (when the rest of the application is rendered on the GPU) would generate a bottleneck that would have negative effects on the applications performance. So in the spirit of this uncertainty, I decided to conduct some minor tests and see whether my basic assumptions were correct.

The CPU method of this process requires migrating the frames to the CPU, calculate the mean values there before sending the frames back to the GPU (as mentioned). So in OpenGL Jitter language this translates to this:
```
[jit.gl.asyncread] //migrate video processing from GPU to CPU
[xray.jit.mean @mean 1 @meandim 0] //calculate horizontal mean values
[jit.gl.texture @dim "original dim"] //migrate back to GPU
```

On the other hand, if we wanted to do the same process on the GPU, we would simply need to recreate the ```[xray.jit.mean]``` object in OpenGL. I figured that the easiest way to do this would be to use a for-loop inside a ```[jit.gl.pix]``` object. However, since hard coding in the OpenGL object only supports the notoriously ambiguous and frustrating GenExpr programming language, I was fortunate enough to receive some aid from a great 3D graphics artist and avid MaxMSP Jitter user named [**Federico Foderero**](https://www.federicofoderaro.com/). Federico actually expanded on this idea himself [**and made a video**](https://www.youtube.com/watch?v=hcHyiSKLpUA&list=UUvDUaH2fbXP_Yc5Lc9UXfqA) where we explores numerous ways of using an OpenGL equivalent of ```[xray.jit.mean]```. Anyway, we can recreate this object using this GenExpr code:       

```
main = vec(0,0,0,0);

if (cell.x > 0) {
	for (i=0; i<dim.x; i+=1) {
		main += nearest(in1, vec(i/dim.x, norm.y));
	}
	main /= dim.x;		
}
out1 = main;
```

In the end I chose to use the CPU method in AudioAnalysis 2.0 because I kept reading that this was standard practice for dealing with calculations of this kind. However, when later comparing the two methods, I was not convinced that the CPU version was best practice in my particular case. The reason is that I experienced very little or no performance differences whatsoever between the two methods. On the other hand, It's worth noting that this effect can be attributed to numerous other factors like battery life, hardware configs of different laptops and application size. Since this subject requires more testing to say anything for sure, I will dedicate an upcoming post to discussing and researching this matter more in-depth.

Another interesting and unresolved challenge of this project was the decreasing image resolution when increasing the printing rate (temporal resolution), as briefly mentioned above. The problem here is how to keep increasing the resolution of a image being printed along a dimension when the frame rate is relatively low without creating blank spaces as the frame rate cannot keep up with the amount of information that "wants" to be printed. So I guess there always has to be a trade off here, but there's probably a more optimal way of doing it than what I have done with this 2.0 version. Certainly a task the 3.0 version!

# References

- Charles, Jean-Francois (2008). [**A Tutorial on Spectral Sound Processing Using Max/Msp and Jitter**](https://www.mitpressjournals.org/doi/pdf/10.1162/comj.2008.32.3.87). MIT press journals.

If you use this application for research purposes, please reference this publication:
- Jensenius, Alexander Refsum (2005). [**Developing Tools for Studying Musical Gestures within the Max/MSP/Jitter Environment**](https://www.duo.uio.no/handle/10852/26907). Proceedings of the International Computer Music Conference, p. 282-285.