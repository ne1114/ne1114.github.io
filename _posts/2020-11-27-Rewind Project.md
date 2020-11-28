---
layout: post
title : Rewind, a control-plane for live video stream processing
---

<div style="text-align: justify"><p style="text-indent: 25px;">
Personally, I realized the importance of a system - incorporating diverse components such as sensors, actuators, storage, computing, communication, etc. - for precision agriculture through reading papers on precision farming and through working at Sensorscall as an embedded firmware developer on a hardware device that requires diverse sensors, cloud and mobile app to provide a meaningful user experience. With the interest in systems, on August, 2019, I joined the Embedded Pervasive Lab (EPL) at Georgia Institute of Technology seeking to gain experiecne on systems research. This post is intended to briefly introduce <b>Rewind</b>, talk about my work in this project, and share my experience.
</p></div>

# Introduction

## Background 
<p></p>
<div style="text-align: justify"><p style="text-indent: 25px;">
With the growing number of cameras deployed in cities for surveillance and traffic control purposes, real-time video analytics on such video streams are becoming considerably important. Processing the requested task solely at the cloud imposes bottlenecks, bandwidth, latency. The field of Edge Computing allows mitigating such limitations by having multiple edge nodes near the application site and optimizing the usage of such edge nodes along with the cloud. Because the surveillance and traffic control using hundreds of cameras deployed throughout the city requires a considerable amount of bandwidth usage and computational resources, Edge Computing is considered to be the most promising solution.
</p></div>

## What is Rewind?
<p></p>
<div style="text-align: justify"><p style="text-indent: 25px;">
<b>Rewind</b> is a novel execution level system for real-time video analytics in a geo-distributed setting. More specifically, given geo-distributed edge nodes that have varying resources available (ex. # of CPU, GPU, Bandwidth), the <b>Rewind</b> enables efficient usage of such resources with lower latency to process the input video processing queries.
</p></div>

<p align="center">
    <figure>
        <img width="750" height="300" src="{{ site.url }}/public/images/Resources.png" > 
        <figcaption> Fig.1 - Example of diverse resources </figcaption>
    </figure>
</p>

<div style="text-align: justify"><p style="text-indent: 25px;">
As shown above, the components of the system vary tremendously ranging from cameras with accelerators, cameras connected through wired/wireless link, local servers, close to infinite computation resources on the cloud, and widely varying bandwidth between each link. The diversity of available resources is significantly increasing due to the trend of integrating more and more computational units to small devices on the edge. With such diverse distributed resources, given a series of queries, <b>Rewind</b> provides optimal allocation of resources to efficiently process such video stream analytics.
</p></div>

<p align="center">
    <figure>
        <img width="750" height="250" src="{{ site.url }}/public/images/Video_processing_queries.PNG" > 
        <figcaption> Fig.2 - Structure of video processing query </figcaption>
    </figure>
</p>


<div style="text-align: justify"><p style="text-indent: 25px;">
The video stream processing query consists of smaller processing <b>nodes</b> that are distinguished based on the functionality. For instance, if the query is to detect objects in the frame, the query could be divided into a reader node, resize node, and detection node. Reader node for reading the input video stream, resize to adjust the resolution, and detection node to detect objects in each frame.  Commonly, video stream processing systems choose continuous streaming topology where they deploy a series of nodes as mentioned above, and continuously stream all the frames. Usually, each node in video processing is considerably expensive, and just transmitting all the frames requires considerable bandwidth usage.    
</p></div>

<div style="text-align: justify"><p style="text-indent: 25px;">
<b>Rewind</b> suggests novel architecture for live video stream processing, by introducing two distinct paths for data flow. I won’t go into details but <b>Rewind</b> suggests a new topology that alleviates the shortcomings of continuous streaming - i) having to constantly allocate expensive nodes, ii) having to send every frame continuously – by having two distinct path for data flow. Since the paper isn’t out in the world yet, I won’t go into details on contributions of <b>Rewind</b>.
</p></div>

<br>

# Personal Contributions


<div style="text-align: justify">
My work with <b>Rewind</b> is centered in two main categories:
<ul>
    <li>Improving/Implementing control plane components of the system</li>
    <li>Developing libraries for end-to-end system evaluation</li>
</ul>
</div>

## Setup 
<p></p>
<div style="text-align: justify"><p style="text-indent: 25px;">
In the initial stage, I went through the pre-existing components that were designed for old architecture. Most of my first semester was committed to understanding the architecture, figuring out how the Controller code was functioning. Also since our system requires multiple Sites with different configurations, I worked on setting and configuring Raspberry Pi and Jetson TX1 so that the system can run on multiple distributed sites. To add more sites, I created multiple VMs in the server using KVM. Set up process included cross-compiling OpenCV targeting ARM architecture in host machine(x86), modifying CmakeLists to compile the source code on RPI & TX1, building required packages FFMPEG, ZeroMQ, Cap'n Proto, dlib, etc. Through numerous trials & errors and troubleshooting processes, I have gained knowledge and confidence in setup procedures that have been a great asset both in research and class works.
</p></div>

## Improving/ Implementing control plane components of the system 
<p></p>
<p align="center">
    <figure>
        <img width="750" height="350" src="{{ site.url }}/public/images/primitive_arch.png" > 
        <figcaption> Fig.3 - Schematics of the primitive architecture </figcaption>
    </figure>
</p>

<div style="text-align: justify"><p style="text-indent: 25px;">
I have inherited the work of the previous person where the <b>Rewind</b> project was at the beginning of the implementation stage. Above is the schematics of the primitive system architecture. I started from the stage where this design was not even fully implemented. And currently, the system has evolved as shown below.  
</p></div>
<p></p>
<p align="center">
    <figure>
        <img width="750" height="350" src="{{ site.url }}/public/images/Latest_system_architecture.PNG" > 
        <figcaption> Fig.4 - Schematics of the latest system architecture </figcaption>
    </figure>
</p>
<p></p>
<div style="text-align: justify"><p style="text-indent: 25px;">
Through multiple iterations of re-designing architecture and implementing new components as well as improving the pre-existing components, the <b>Rewind</b> system evolved to current state as shown above. I won’t go into full details of the system but briefly introduce the functionality of each components. 
</p></div>
<div style="text-align: justify"><p style="text-indent: 25px;">
The <i>Controller</i> receives video processing queries from the <i>Clients</i>. The required resources will be assigned to each node (functional processing unit consisting of a query, i.e. reader, filter, detector, etc.)  within the query by the <i>Profiler</i> based on the profile of the system. The <i>Optimizer</i> will then assign the optimal site for each node in the query to be deployed to. Based on the required resource allocation and site allocation, the <i>Scheduler</i> will generate a deployment plan. <i>Site Managers</i> will send messages to <i>Site Agent</i> to pre-allocate resources (CPU, GPU, Memory) to each <i>Rocklet Manager</i>, or if needed create a new <i>Rocklet Manager</i> to sites depending on the deployment plan. Then the <i>Execution Manager</i> will deploy each node within the query to assigned <i>Rocklet Managers</i>. The <i>Controller</i> keeps track of the current state of the whole system and stores it in the database which will be used to optimize the query and deallocate the nodes. The ZeroMQ library is used for every communication and Cap'n Proto is used for any external communication messages. My main role was to develop such control plane components, annotated with red marks, in a robust and coordinated manner. It was challenging to re-design/enhance the system architecture iteratively in a way that the system is robust and can manage multiple processes running simultaneously at distributed sites in a synchronized manner.  
</p></div>

## Developing libraries for system evaluation 
<p></p>
<div style="text-align: justify"><p style="text-indent: 25px;">
Based on the thorough and in-depth understanding of the system, I also developed libraries for end-to-end system evaluation. The library contains functions to deploy the system (source code, database, SSH key), setting SSH tunneling between distributed sites, transcoding and streaming benchmark video based on pre-defined configurations, executing the system, request batch of queries serving as a client, analyzing log files fetched from distributed sites based on evaluation metrics, cleaning up the system, etc. All the functions are easily configurable just by changing parameters in the config file or creating a new scenario (series of queries) by using the Client library which I also implemented to ease the evaluation process. I also worked on getting benchmark videos including Visual Road, UA-DETREC, live surveillance cameras, etc.
</p></div>

# Conclusion

<p></p>
<div style="text-align: justify"><p style="text-indent: 25px;">
After a considerable amount of time devoted to the project, we are now at the stage of evaluating the system and we are expecting to submit the paper to VLDB 2021. I am extremely happy and proud of our system and I cannot wait to publish the work out to the world soon. To sum up, four semesters of research, participating in the <b>Rewind</b> project provided me insight into systems research, taught me an enormous amount of skills and lessons. I still remember the time when I first joined the lab, I did not have much idea about research and what to do. However, looking back on the time I spent on this project, I can now tell that I have significantly grown semester by semester. Although I still have a lot to grow and learn as I proceed with my career in Ph.D. programs, I am confident that the skills and knowledge that I acquired throughout my research will be the crucial basis for my career as a researcher.  
</p></div>
