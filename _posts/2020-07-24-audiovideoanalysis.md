---
layout: post
date: 2020-07-24
title: "AudioVideoAnalysis"
image: /assets/img/2020_07_24_ava_main.jpg
categories: Projects
excerpt: "A standalone application for realtime spectral analysis of audio and video, redeveloped by myself in collaboration with RITMO (Center for Interdisciplinary Studies in Rhythm, Time and Motion) and the FourMS lab at the University of Oslo."
Keywords: MaxMSP, Jitter, video, audio, spectral, analysis, standalone, application, interaction design, RITMO, University of Oslo
---

Welcome to the second installment of a three-part series on my adventures of redeveloping the [**Musical Gestures Toolbox for Max**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/musicalgesturestoolbox/mgt-max/) for the University of Oslo*. This post presents the AudioVideoAnalysis 2.0, a MaxMSP standalone application for realtime spectral analysis of audio and video I developed in collaboration with the [**RITMO center for interdisciplinary studies in rhythm, time and motion)**](https://www.uio.no/ritmo/english/) and their [**FourMS lab**](https://www.uio.no/ritmo/english/research/labs/fourms/) at the University of Oslo, spring 2020.

<figure style="float: left; margin-right: 10px;">
   <img src="/assets/img/2020_07_24_ava_1.jpg" alt="AudioVideoAnalysis 0.1"
   title="AudioVideoAnalysis 0.1" width="auto" />
   <figcaption>AudioVideoAnalysis 0.1</figcaption>
</figure>

AudioVideoAnalysis 0.1 was an application that recorded video/motiongrams and spectrograms in realtime from any available media source, developed by [**Alexander Refsum Jensenius**](http://people.uio.no/alexanje) in the early 2000s as a part of the [**Musical Gestures Toolbox for Max**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/musicalgesturestoolbox/mgt-max/). Its intended purpose was to provide a quick, reliable and easy way of recording and analyzing spectral content of both video and audio together on one platform.

My main task was to remove any outdated dependencies the app had on old MaxMSP externals and replace them with more stable and long-term solutions, same as before. However, since the VideoAnalysis 2.1 had recently reached such high standards, there was no chance that this application was to heed to any lesser bar. In the process, I tried to stay true to the original design improving it as much as I could in various ways.

*
<sup>Please [**visit my previous post on the VideoAnalysis**](https://aleksandertidemann.github.io/projects/2020/07/23/videoanalysis.html) to learn more about the context and history of these particular development projects.</sup>

# AudioVideoAnalysis 2.0

<figure style="float: none;">
   <img src="/assets/img/2020_07_24_ava_main.jpg" alt="AudioVideoAnalysis 2.0"
   title="AudioVideoAnalysis 2.0" width="auto" />
   <figcaption></figcaption>
</figure>

Similar to its previous version, AudioVideoAnaysis 2.0 is intended for generating realtime video/motiongrams and spectrograms simultaneously on one easy-to-use platform. Spectrograms are visual representations of the frequency spectrum of an audio signal as it varies over time. The frequencies range from lowest to highest (where highest is half of your sampling rate) along the display windows Y-axis. Time is therefore represented from left to right along the X-axis. Similarly, videograms and motiongrams are image representations of movement over time. in AudioVideoAnaysis 2.0, we create video and motiongrams from the movement in any camera source connected. We call these images for spectral representations because they visualize the spectral content, or the various spectrums, of the medium in question. In audio programming and sonology, this shift is often referred to as moving our attention from the time domain to the frequency domain.

<!--*
While (I and) [**Balint Lackzo**](https://github.com/balintlaczko) pushed the capabilities of MaxMSP Jitter with the VideoAnalysis software, we also effectively set the standard for the remaining two MaxMSP-based analysis applications still scheduled for updating, namely AudioVideoAnalysis and AudioAnalysis. Nevermind that the sole objective of these projects was to simply remove any dependencies on "old" MaxMSP externals, once the VideoAnalysis turned out the way it did there was no way the AudioVideoAnalysis could be of any lesser standard. At least in my mind, it could not. A decision that resulted in a tiresome, challenging, and extremely valuable experience altogether.
-->
## Features
The newly developed 2.0 version features more complex audio, video and display functionalities than its predecessor as well as the ability to export images. Speaking of display functionalities, let's look more in-depth at what the 2.0 version has grown capable of.

### Rate of Play
Because the application records and previews information across the X-axis of a display window in realtime, I naturally had to address whether the rate of this printing process should be fixed or modifiable by users. There are pros and cons with both methods but I settled on a modifiable printing rate because adjusting the printing rate in our case is equivalent to increasing the "temporal resolution" of the images, a valuable asset of an analysis toolbox like this.

<figure style="float: none">
   <img src="/assets/img/2020_07_24_rate_window.jpg" alt="AVA rate"
   title="rate" width="800" />
   <figcaption>A spectrogram generated by AudioVideoAnalysis 2.0. The Rate function controls the printing rate and temporal resolution of our images</figcaption>
</figure>

The printing rate is modified by using the ```rate``` dial in the applications UI, with a rate value of 100% corresponding to a printing "length" of 60 sec from start to finish. This start-to-finish time is also expressed explicitly under ```Total Display Time```. It's worth mentioning that this particular rate functionality operates by zooming in on our image from an initial fixed position. This fixed position is effectively the display windows maximum resolution, an ```1024 x "user-specified Y-dim"``` image with the printing rate at 100% (60 seconds). What this means is while increasing the rate might give you a much higher temporal resolution, it will effectively decrease the overall image resolution. I used this method because it's safe and ensures smooth printing processes while simultaneously accommodating for relatively low frame-rates. Accommodating for low frame-rates is also necessary to make the application more stable on various systems. However, setting a higher video device resolution can help compensate for some of these loses.

### Analysis Markers
<!--*
Another mentionable display feature in AudioVideoAnalysis is its various analysis markers. These consist of optional frequency and time markers along the XY-axis, grid view with customizable size and clickpoint data retrieval.
-->
As I introduced features like printing ```Rate``` and a ```Logarithmic``` dial, giving users the ability to adjust the spectrograms frequency distribution dynamically from linear to logarithmic, the need for a set of analysis markers grew stronger. More so, it's essential to have a frame of reference when analyzing spectral images. The result was an implementation of automatically re-scaling frequency and time markers in the display window accessible from the menubar. Changing the ```Grid Size``` parameter in the UI adds or subtracts the number of analysis markers visible. Furthermore, by enabling ```View Display Grid``` in the menubar, a grid matrix is added on top of the images which can further improve our frame of reference.

<figure style="float: none;">
   <img src="/assets/img/2020_07_24_ava_analysis_markers.gif" alt="Grid"
   title="Grid" width="800" />
   <figcaption> Spectrogram (audio) in green and motiongram (video) in grayscale. Adding a grid to the display window can further assist analysis by providing a better frame of reference.</figcaption>
</figure>

Finally, I decided to implement something called clickpoint data retrieval, allowing for an even more detailed inspection of the spectral audio content. I first came over this feature in a paper by [**Jean-Francois Charles**](https://www.jeanfrancoischarles.com/) called [**A Tutorial on Spectral Sound Processing Using Max/MSP and Jitter**](https://www.mitpressjournals.org/doi/pdf/10.1162/comj.2008.32.3.87). In this fantastically detailed and informative paper on various ways to generate and view spectral audio content in MaxMSP and Jitter, he introduces a method for retrieving Time (ms), Hz (frequency) and Amplitude (volume) from anywhere in a recorded spectrogram via physically engaging the image with our mouse.

<figure style="float: left; margin-right: 10px;">
   <img src="/assets/img/2020_07_24_ava_clickpoint.jpg" alt="Clickpoint"
   title="Clickpoint" width="300" />
   <figcaption></figcaption>
</figure>

I used Jean-Francois' algorithms as the basis my implementation of this feature in AudioVideoAnaysis, only reconfiguring the actual text to appear in the display window as opposed to in the main UI, to display seconds instead of milliseconds, and finally to change color based on the display mode and spectrogram color. The displayed information (specific values in the text) also had to account for the dynamic printing rate and logarithmic feature previously mentioned.

### Image Layering
Being able to record and preview the spectral content of both video and audio simultaneously is one of AudioVideoAnaysis' greatest strengths. With this in mind, I thought it would serve well to incorporate multiple display options that enable users to view the images in different configurations. Originally, the application presented the images layered in a cake-like fashion with video at the bottom and the audio directly above it. In the 2.0 version, we can view the images individually (only audio or only video), layered cake-wise (as before), or layered on top of each other. The latter being the most recent and interesting for exploring relationships between movement and sound.

### General
If you want a more detailed run-through of these features, and how to use the application in general, you can access an in-depth *How To Use* guide online [**from this website**](https://github.com/fourMs/AudioVideoAnalysis/wiki) complete with lots of yummy illustrations and step-by-step instructions. Finally, a more general list of AudioVideoAnaysis' (2.0) most prominent features:

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
* Dynamically adjust spectrograms logarithmic curve (from linear to logarithmic)
* Spectral blurring effect (audio)
* In-depth *How to use* guide
* Report issues straight to the developer site

## Downloads
The application is 100% open source and available on both OSX and WIN. It can be downloaded from the following places:
* [**FourMS GitHub page**](https://github.com/fourMs/AudioVideoAnalysis/releases)
* [**RITMO homepage at the University of Oslo**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/AudioVideoAnalysis/)

If you want to contribute to AudioVideoAnaysis' development you are free to do so. Simply fork the [**main AudioVideoAnalysis repo**](https://github.com/fourMs/AudioVideoAnalysis), clone it, develop some awesome stuff and make a pull request for us to review.


# Trials and Tribulations

<figure style="float: none;">
   <img src="/assets/img/2020_07_24_ava_motiongram.jpg" alt="Motiongram"
   title="Motiongram" width="800" />
   <figcaption> Motiongrams generated by AudioVideoAnaysis represent vertical motion (Y-axis) over time (X-axis).</figcaption>
</figure>

The most interesting challenge of this development project was figuring out how to build it in an OpenGL context. This means using specific video objects in MaxMSP that processing images on the GPU rather than on the CPU. This was a highly relevant challenge because AudioVideoAnaysis primarily is a realtime application that should have smooth visualizations and user experiences uninterrupted by any glitches or lags. It would also be informative in understanding how we might redesign VideoAnalysis in the future.

While I was able to move all the image processing within an OpenGL context in the end, I specifically struggled with a small but key feature of the video processing. An important component of the video/motiongram section of the application is a process that calculates the mean values for every row (X-dimension) in a frame and spits out one-dimensional matrices that we print sequentially along a "canvas" to produce the video/motiongram. When confronted with this specific process I questioned whether doing mean calculations of frames was better on the GPU or the CPU. My worry was doing these calculations on the CPU (when the rest of the application is rendered on the GPU) would generate a bottleneck that would have negative effects on the performance of the application. So in the spirit of this uncertainty, I decided to conduct some minor tests and see whether my basic assumptions were correct.

The CPU method of this process requires migrating the frames to the CPU and calculate the mean values there before sending the frames back to the GPU (as mentioned). In OpenGL Jitter language, this translates to:
```
[jit.gl.asyncread] //migrate video processing from GPU to CPU
[xray.jit.mean @mean 1 @meandim 0] //calculate horizontal mean values
[jit.gl.texture @dim "original dim"] //migrate back to GPU
```

On the other hand, if we wanted to do the same process on the GPU, we would simply need to recreate the ```[xray.jit.mean]``` object in OpenGL. I figured that the easiest way to do this would be to use a for-loop inside a ```[jit.gl.pix]``` object. However, since hard coding in the OpenGL object only supports the notoriously ambiguous and frustrating GenExpr programming language, I was fortunate enough to receive some aid from a great 3D graphics artist and avid MaxMSP Jitter user named [**Federico Foderero**](https://www.federicofoderaro.com/). Federico expanded on this idea himself [**and made a video**](https://www.youtube.com/watch?v=hcHyiSKLpUA&list=UUvDUaH2fbXP_Yc5Lc9UXfqA) where we explore numerous ways of using an OpenGL equivalent of ```[xray.jit.mean]```. Anyway, we can recreate this object using this GenExpr code:       

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

In the end, I chose to use the CPU method in AudioAnalysis 2.0 because I kept reading that this was standard practice for dealing with calculations of this kind. However, when later comparing the two methods, I was not convinced that the CPU version was the best practice in my particular case. The reason is that I experienced very little or no performance differences whatsoever between the two methods. On the other hand, It's worth noting that this effect can be attributed to numerous other factors like battery life, hardware configs of different laptops and application size. Since this subject requires more testing to say anything for sure, I will dedicate an upcoming post to discussing and researching this matter more in-depth.

Another interesting and unresolved challenge of this project was the decreasing image resolution when increasing the printing rate (temporal resolution), as briefly mentioned above. The problem here is how to keep increasing the resolution of an image being printed along a dimension when the frame rate is relatively low without creating blank spaces as the frame rate cannot keep up with the amount of information that "wants" to be printed. So I guess there always has to be a trade-off here, but there's probably a more optimal way of doing it than what I have done with this 2.0 version. Certainly a task the 3.0 version!

# References

- Charles, Jean-Francois (2008). [**A Tutorial on Spectral Sound Processing Using Max/Msp and Jitter**](https://www.mitpressjournals.org/doi/pdf/10.1162/comj.2008.32.3.87). MIT press journals.

If you use this application for research purposes, please reference this publication:
- Jensenius, Alexander Refsum (2005). [**Developing Tools for Studying Musical Gestures within the Max/MSP/Jitter Environment**](https://www.duo.uio.no/handle/10852/26907). Proceedings of the International Computer Music Conference, p. 282-285.

I've also been honored an acknowledgment by Cycling74 themselves (creators/gods of MaxMSP) at [**their official homepage**](https://cycling74.com/projects/audiovideoanalysis).
