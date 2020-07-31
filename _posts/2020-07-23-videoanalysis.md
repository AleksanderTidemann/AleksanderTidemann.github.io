---
layout: post
date: 2020-07-23
title: "VideoAnalysis"
image: /assets/img/2020_07_23_va_main.png
categories: Projects
excerpt: "A standalone application for creating visualizations and extracting motion features from video files, redeveloped by myself and Balint Laczko in collaboration with RITMO (Center for Interdisciplinary Studies in Rhythm, Time and Motion) and the FourMS lab at the University of Oslo."
Keywords: MaxMSP, Jitter, video, audio, spectral, analysis, standalone, application, interaction design, RITMO, University of Oslo
---

In the early fall of 2019, I and a few others were given the opportunity as research assistants for the RITMO Center for Interdisciplinary Studies in Rhythm, Time and Motion at the University of Oslo to update and refurbish a series of sound and video analysis tools developed at the [**FourMS lab**](https://www.uio.no/ritmo/english/research/labs/fourms/). Among these analysis tools were the [**Musical Gestures Toolbox for MaxMSP**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/musicalgesturestoolbox/mgt-max/), consisting of the VideoAnalysis, AudioVideoAnalysis and AudioAnalysis applications, as well as a few more powerful tools such as the [**Musical Gestures Toolbox for Matlab**](https://github.com/fourMs/MGT-matlab/) and the [**Musical Gestures Toolbox for Python**](https://github.com/fourMs/MGT-python).

I was first assigned to update the VideoAnalysis application together with the great [**Balint Lackzo**](https://github.com/balintlaczko), now a good friend and bottomless pit of creative inspiration (he's awesome). Our main task was to remove any outdated dependencies VideoAnalysis had on old MaxMSP externals, such as Jamoma, and replace them with more stable and long-term solutions. However, given our project time-frame and the condition the 1.0 version was in, we ended up completely redeveloping the software. Although many of its main features are still the same, the architecture was thoroughly updated, tested and (most importantly) optimized for non-realtime video processing.  

## Musical Gestures Toolbox for MaxMSP

<figure style="float: left; margin-right: 10px;">
   <img src="/assets/img/2020_07_23_va_1.jpg" alt="VideoAnalysis 1.0"
   title="VideoAnalysis 1.0" width="auto" />
   <figcaption>VideoAnalysis 1.0</figcaption>
</figure>

The Musical Gestures Toolbox for Max was originally developed by [**Alexander Refsum Jensenius**](http://people.uio.no/alexanje) in the early 2000s. The toolbox later became integrated as the first collection of video modules in the [**Jamoma**](http://www.jamoma.org) project. Read more about this in the cited research paper below.

It turned out that many people found the tools for video visualization useful, but they did not want to invest time in learning how to develop in Max. This led to the development of the standalone VideoAnalysis application, based on modules from the MGT. Over the years, it has been used by several researchers and practitioners in fields such as music, dance, medicine, and physiotherapy. However, in 2019 these unattended analysis applications were outdated and in dire need of revision.

# VideoAnalysis 2.1

<figure>
   <img src="/assets/img/2020_07_23_va_main.png" alt="Alternate Text"
   title="VideoAnalysis 2.1" width="auto" />
   <figcaption></figcaption>
</figure>

The VideoAnalysis is a standalone application where you add video-files and/or video-folders, pre-process those videos in various ways before exporting a whole array of motion data and interesting visualizations from those videos through a user-friendly and engaging GUI. The purpose of the application, other than to be a steady application for relatively low-level video and motion analysis, is to inspire its users to dig further down the motion and video analysis rabbit-hole. Hopefully, the 2.1 version is successful in this regard by being a **huge** upgrade from its predecessor, making it easier and more fun than ever to get some quality grade motion data. Cheers!

## Features
As briefly mentioned, the bulk portion of our development process went into redesigning the systems architecture. However, the 2.1 version harbors many new features as well. Let's look closer at some of these features to see what VideoAnalysis 2.1 is really capable of.

### Batch processing
One of the applications' key features is its ability to batch process folders of video files. This means we can quickly generate a bunch of data simply by dragging and dropping a folder into the application, configure some settings and hitting export. With batch processing came also the need for a system to store and recall video settings. To tackle this, we decided to implement a system that would let users decide to either ```Remember settings for each file``` or ```Use same settings for each file```, fully accessible from the applications menubar. When switching from ```Remember settings for each file``` to ```Use same settings for each file```, the current settings will be applied to all videos. However, It's also possible to switch back to ```Remember settings for each file``` to regain the individual video settings from before.

<figure style="float: left; margin-right: 10px;">
   <img src="/assets/img/2020_07_23_va_settings.png" alt="Settings"
   title="Settings" width="320" />
   <figcaption></figcaption>
</figure>

This feature is a big contribution to VideoAnaysis' *easy-to-use* design and can have a huge impact on its effectiveness. It's also possible to save your settings as a .json file which can make it easy to pick up your work where you left off or to share your settings with other users later on.

<br>

### Cropping, Trimming and Skipping
Being a non-realtime application, VideoAnaysis is designed to be a reliable exporter of analysis content. However, with this comes the issue of managing excessive processing speeds. Since we wanted the application to accommodate for videos of various sizes and/or lengths, we made three options directly aimed at combatting excessive processing speeds.

<figure style="float: left; margin-right: 10px;">
   <img src="/assets/img/2020_07_23_va_source_display.png" alt="Source Display"
   title="Source Display" width="320" />
   <figcaption></figcaption>
</figure>

Cropping can greatly reduce the video size and thus reduce export time by a considerable factor. In VideoAnalysis you can crop any video by enabling the ```Cropping``` button before clicking and dragging your preferred cropping rectangle in the preview window. We can also reduce video length by either trimming or skipping.

Trimming is a feature that sets new start and end times to our video and can be adjusted visually by configuring the yellow banner in the playbar or numerically from the applications main UI. This can be helpful if the desired analysis material is located at a specific place in our video or if there are portions at the beginning or end which can be discarded without compromising analysis.

Skipping, on the other hand, is a method where you specify a factor that decides which frames are processed. Let's say you have a skipping value of 1, this means we skip every other frame and only processes frames ```1, 3, 5, 7 etc.```. Consequently, a skipping value of 1 will reduce the processing time by 50%(!) as we are effectively cutting the video in half while preserving most of its content.

These features let us control the size of the video in various ways but would not be very helpful without a frame of reference. In light of this, we also added ```Total Frames``` and ```Duration``` counters in the applications main UI so you can always keep track of how many frames of the current video are scheduled for processing.

### Video Visualizations

<figure>
   <img src="/assets/img/2020_07_23_va_displaywindow.png" alt="displaywindow"
   title="displaywindow" width="auto" />
   <figcaption>Video Analysis display window showing Box, History and Motion video.</figcaption>
</figure>

In addition to extracting motion data, VideoAnaysis can also create useful visualizations of movement in videos. These include average and motion average images (see image below), video/motiongrams, history video, box video and motion videos. Although some of these visualizations are available to preview in the applications display window and UI, most of them are only available via export. You can read much more about these visualization techniques in [**this article by Alexander Jensenius**](https://www.duo.uio.no/handle/10852/26907).

To create a motion video we calculate the difference between the current frame and the previous frame. This will generate an image where only the motion in the video is represented in color while the rest is black. Furthermore, to reduce unwanted video static and noise, we've added two different noise filters (unary and binary) to discard irrelevant movement information. You can choose which filter to use and adjust the ```threshold``` setting directly from the ```Motion Video``` UI section.

<figure style="float: left; margin-left: 10px;">
  <img src="/assets/img/2020_07_23_va_pianist_avg.jpg"
  alt="motion average image" title="motion average image" width="320" />
  <figcaption>An average image of a pianist video. Notice the fog-like area over keyboard indicating where the biggest concentration of motion is located.</figcaption>
</figure>

<figure style="float:none">
  <img src="/assets/img/2020_07_23_va_pianist_motionavg.jpg"
  alt="average image" title="average image" width="320" />
  <figcaption>An motion average image of the same video efficiently filtering out useless information.
  Both images are generated with VideoAnaysis 2.1.</figcaption>
</figure>

Motiongrams are closely linked to motion videos in that they use a motion image to represent motion over time. In VideoAnaysis, and the earlier **Musical Gestures Toolbox for MaxMSP**, motiongrams are created by calculating mean values of either every row or column of a frame (depending on if we want to represent time on the X or Y-axis) using the ```[xray.jit.mean]```object in MaxMSP. This outputs a one-dimensional matrix of mean values that we can use to sequentially print across a "canvas" to produce a motiongram. Similarly, videograms are motiongrams that bypass the motion image processing before being sent to the ```[xray.jit.mean]``` object.

At the center of the display window shown above is a representation of a history video. This technique tracks the center of motion (centroid) in our motion image and visualizes it by drawing red circles onto our unprocessed video. If we change the ```Snake Length``` parameter in the UI, we can customize the length of our centroid snake (succession of circles) that we can use to track and visualize the motion history. Similarly, the box video analyzes the movement in a motion image and isolates it through a bounding box printed onto our unprocessed video. In the UI we can adjust the refresh ```Rate``` of our box, as well as the box ```Radius``` and noise ```Threshold``` for more control.   

### General
If you want a more detailed run-through of these features, and how to use the application in general, you can access an in-depth *How To Use* guide online [**from this website**](https://github.com/fourMs/VideoAnalysis/wiki) complete with lots of yummy illustrations and step-by-step instructions. And finally, a more general list of VideoAnaysis' (2.1) most prominent features:

* Open individual video files or batch-process folders of files
  * Drag and drop supported
* Realtime preview of visualizations*
  * Detached and scalable Display Window
* Pre-processing
  * Cropping
  * Trimming
  * Color
  * Rotation
* Extract motion features from video files
  * Quantity of motion
  * Centroid of motion
  * Area of motion
  * All data exported in .csv format
* Create various types of visualizations from video files
  * Average and motion average images
  * Motiongrams and Videograms
  * Motion video
  * Centroid of motion video (History)
  * Box video
* Store, use and recall all video settings
* In-depth *How to use* guide
* Report issues straight to the developers' site

<!--*
<sup>Its important to note that the VideoAnalysis is a *non-realtime analysis application* which means that it's optimized to export data for later viewing. "And why is this important to note", you might think. Well, if the application was optimized for realtime usage, we would have built it in an entirely different way. What this effectively means is that you might experience some discomfort (lagging, basically..) /playing back videos in the application. This is especially the case with videos with substantial file sizes.</sup>
-->

## Downloads
The application is one hundred percent open source and available on both OSX and WIN. It can be downloaded from the following places:
* [**FourMS GitHub page**](https://github.com/fourMs/VideoAnalysis/releases)
* [**RITMO homepage at the University of Oslo**](https://www.uio.no/ritmo/english/research/labs/fourms/downloads/software/VideoAnalysis/)

If you want to contribute to VideoAnaysis' development you are free to do so. Simply fork the [**main VideoAnalysis repo**](https://github.com/fourMs/VideoAnalysis), clone it, develop some awesome stuff and make a pull request for us to review.

# Trials and Tribulations

<figure>
   <img src="/assets/img/2020_07_23_va_pianist_motiongram.jpg" alt="pianist motiongram"
   title="pianist motiongram" width="auto" />
   <figcaption>Motiongram of our pianist example. Here, the mean of the motion image (y-axis) is printed over time (x-axis).</figcaption>
</figure>

Through our development, we encountered a bunch of strange problems, everything from mysterious unwanted video/image artifacts to the inability to send the app to anyone over the internet and for them to have it working without a nasty hack. In fact, we experienced so many problems that we discovered 2 major bugs with MaxMSP itself. One of these bugs was reported and readily updated/fixed in a later MaxMSP version update, but the other has yet to be. The "have-yet-to-be-fixed" bug is an issue with exporting images/matrices in TIFF format on Windows machines. For some reason, these images will only export if their dimensions are multiples of 4... A super weird bug, and a great find by Balint. Now that we knew why we couldn't export TIFF formatted images on Windows machines, we could create a hack the enabled us to.

A more fundamental issue of this development project is the fact that all of VideoAnaysis' processing happens on the CPU. What this means is that while it's very thorough and precise, the VideoAnalysis struggles with realtime previewing and is still slow to manage video with substantial sizes. In theory, you should always try to do video and image processing on the GPU because of its technical design. [**There's a great video on this by Federico Foderero**](https://www.youtube.com/watch?v=V3_p9R7YG-g), a MaxMSP Jitter guru, where he explains this in detail in a MaxMSP context. However, in its original 1.0 version, it was a good idea for the app to process most of its stuff on the CPU because some of its main tasks, like calculating the mean values for the motiongrams, are supposedly best done serially on the CPU. Additionally, the application size was smaller and the MaxMSP OpenGL framework was not nearly as powerful, well documented or supported as it is today.

The reason we didn't do this from the start of our development process is that we didn't think the application would grow to the size ended up having. It also wasn't originally part of our job. Almost none of this was. But looking back it pains me to know that the performance of VideoAnalysis might have been exponentially better if we had spent a little more time on project management. On the other hand, we can't know for sure whether an OpenGL context would significantly improve the performance of the application. The main reason being its size. VideoAnalysis 2.1 is a big an quite diverse application, relatively speaking of course, which means its performance depends on more than its image rendering capabilities. I guess the only way to know for sure is to test various processes and their OpenGL equivalents to better estimate whether a full transition would be worth the effort.

# References
If you use this application for research purposes, please reference this publication:

- Jensenius, Alexander Refsum (2005). [**Developing Tools for Studying Musical Gestures within the Max/MSP/Jitter Environment**](https://www.duo.uio.no/handle/10852/26907). Proceedings of the International Computer Music Conference, p. 282-285.

We've also been honored an acknowledgment by Cycling74 themselves (creators/gods of MaxMSP) at [**their official homepage**](https://cycling74.com/projects/video-analysis).
