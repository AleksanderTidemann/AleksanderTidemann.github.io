---
layout: post
date: 2020-07-23
title: "VideoAnalysis"
image: /assets/img/2020_07_23_va_main.png
categories: Projects
excerpt: "VideoAnalysis is a standalone application for creating visualizations and extracting motion features from video files, developed by myself and Balint Laczko in collaboration with RITMO (Center for Interdisciplinary Studies in Rhythm, Time and Motion) and the FourMS lab at the University of Oslo."
---

In the early fall of 2019, myself and a group of research assistants at the RITMO center at UiO were given the opportunity to refurbish and develop a series of video and audio analysis software through the [**FourMS lab**](https://www.uio.no/ritmo/english/research/labs/fourms/). Among these were VideoAnalysis, AudioVideoAnalysis and AudioAnalysis (standalone applications developed in MaxMSP), as well as a few more powerful analysis tools such as the [**Musical Gestures Toolbox for Matlab**](https://github.com/fourMs/MGT-matlab/) and the [**Musical Gestures Toolbox for Python**](https://github.com/fourMs/MGT-python).

I began this project by redeveloping and updating the VideoAnalysis application together with the great [**Balint Lackzo**](https://github.com/balintlaczko), now a good friend and my own personal bottomless pit of creative inspiration (he's awesome).

## Musical Gestures Toolbox for MaxMSP

<figure style="float: left; margin-right: 10px;">
   <img src="/assets/img/2020_07_23_va_1.jpg" alt="VideoAnalysis 1.0"
   title="VideoAnalysis 1.0" width="auto" />
   <figcaption>VideoAnalysis 1.0</figcaption>
</figure>

The MaxMSP analysis applications were started by [**Alexander Refsum Jensenius**](http://people.uio.no/alexanje) in 2003, first in the form of the [**Musical Gestures Toolbox for Max**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/musicalgesturestoolbox/mgt-max/). The toolbox later became integrated as the first collection of video modules in the [**Jamoma**](http://www.jamoma.org) project. Read more about the project in the cited research paper below in the reference section.

It turned out that many people found the tools for video visualization useful, but they did not want to invest time in learning how to develop in Max. This led to the development of the standalone VideoAnalysis application, based on modules from the MGT. Over the years, it has been used by several researchers and practitioners in fields such as music, dance, medicine, and physiotherapy.

However, in 2019 these unattended applications were outdated and in dire need of revision.

# VideoAnalysis 2.1

<figure>
   <img src="/assets/img/2020_07_23_va_main.png" alt="Alternate Text"
   title="VideoAnalysis 2.1" width="auto" />
   <figcaption></figcaption>
</figure>

Our main task was to take the VideoAnalysis 1.0 and remove any dependencies it had on "old" MaxMSP external packages, such as Jamoma, and replace them with more stable and long-term solutions. However, given our project time-frame, Balint's otherworldly capabilities in MaxMSP and our general view on the state of the application, we ended up completely redeveloping the software. Though the same general features persist (the ability to export a series of motion data and generate some interesting motion visualizations), everything around them was thoroughly updated, tested and (most importantly) optimized for non-realtime video processing.  

The purpose of our newly developed 2.1 version, other than to be a steady application for relatively low-level video and motion analysis on video files, is to inspire its users to dig further down the motion and video analysis rabbit-hole. More specifically, down where the MatLab and Python versions are. Hopefully, it achieves this by being a fast, easy and reliable source of motion data, a kind of data that otherwise would be both time-consuming and difficult to attain for beginners.

## Features

 In VideoAnalysis (2.1) you can pre-process, view and export a whole array of motion data and interesting visualizations through a user-friendly and engaging GUI. Among its many features, the following prominently stick out:

* Open individual video files or batch-process folders of files
  * Drag and drop supported
* Realtime preview of visualizations*
  * Detached and scalable Display Window
* Pre-processing
  * Cropping
  * Trimming
  * Color
  * Rotation
  * etc..
* Extract motion features from video files
  * Quantity of motion
  * Centroid of motion
  * Area of motion
  * All data exported in .csv format
* Create various types of visualizations from video files
  * Motiongrams and Videograms
  * Motion images
  * Centroid of motion video
  * Box image video
  * Motion video
  * etc..
* Store, use and recall all video settings
* In-depth *How to use* guide
* Report issues straight to the developer site

<figure align="middle">
   <img src="/assets/img/2020_07_23_va_cropping.gif" alt="Alternate Text"
   title="VideoAnalysis 2.1" width="auto" />
   <figcaption align="middle">Cropping and display window preview with VideoAnalysis 2.0</figcaption>
</figure>

If you want a more detailed run-through of these features, and how to use the application in general, you can access the in-depth *How To Use* guide online [**from this website**](https://github.com/fourMs/VideoAnalysis/wiki) complete with lots of yummy illustrations and step-by-step instructions.

*
<sup>Its important to note that the VideoAnalysis is a *non-realtime analysis application* which means that it's optimized to export data for later viewing. "And why is this important to note", you might think. Well, if the application was optimized for realtime usage, we would have built it in an entirely different way. What this effectively means is that you might experience some discomfort (lagging, basically..) playing back videos in the application. This is especially the case with videos with substantial file sizes.</sup>

## Downloads
The application is one hundred percent open source and available on both OSX and WIN. It can be downloaded from the following places:
* [**FourMS GitHub page**](https://github.com/fourMs/VideoAnalysis/releases)
* [**RITMO homepage at the University of Oslo**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/VideoAnalysis/)

If you want to contribute to VideoAnaysis' development you are free to do so. Simply fork the [**main VideoAnalysis repo**](https://github.com/fourMs/VideoAnalysis), clone it, develop some awesome stuff and make a pull request for us to review.

# Trials and Tribulations

Through our development, we encountered a bunch of strange problems, everything from mysterious unwanted video/image artifacts to the inability to send the app to anyone over the internet and for them to have it working without a nasty hack. In fact, we experienced so many problems that the development process resulted in the discovery of at least 2 major bugs with MaxMSP itself. Some of these were reported and readily updated/fixed in later MaxMSP version updates. So in other words, quite a lot of interesting stuff happened which I will definitely write more about in the future.

One of our biggest regrets, however, is not moving the main image/video rendering from the CPU to the GPU through MaxMSP's OpenGL framework. In theory, you should always try to do video and image processing on the GPU because of its technical design. [**There's a great video on this by Federico Foderero**](https://www.youtube.com/watch?v=V3_p9R7YG-g), a MaxMSP Jitter guru, where he explains this in detail in a MaxMSP context.

In VideoAnalysis 2.1, all the processing happens on the CPU. What this means is that while it's very thorough and precise, the VideoAnalysis struggles with realtime previewing and is slow to export videos of any substantial size. Originally, back in the early 2000s, it was a good idea for the app to process its content on the CPU because some of its main processes, like calculating the running mean of video pixels values, are best done serially and therefore on the CPU. Additionally, the application size smaller and the MaxMSP OpenGL framework was not nearly as powerful, well documented or supported as it is today.

The reason we didn't do this from the start of our re-development process is that we didn't think the application would grow to the size ended up having. It also wasn't originally part of our job. Almost none of this was. But looking back it pains me to know that the performance of VideoAnalysis might have been exponentially better if we had spent a little more time on project management.

# Reference

<figure>
   <img src="/assets/img/2020_07_23_va_ui1.png" alt="Alternate Text"
   title="VideoAnalysis 2.1" width="auto" />
   <figcaption align="middle">VideoAnalysis 2.0 main UI</figcaption>
</figure>

If you use this toolbox for research purposes, please reference this publication:

- Jensenius, Alexander Refsum (2005). [**Developing Tools for Studying Musical Gestures within the Max/MSP/Jitter Environment**](https://www.duo.uio.no/handle/10852/26907). Proceedings of the International Computer Music Conference, p. 282-285.

We've also been honored an acknowledgment by Cycling74 themselves (creators/gods of MaxMSP) at [**their official homepage**](https://cycling74.com/projects/video-analysis).
