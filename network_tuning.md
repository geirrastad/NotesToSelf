# ROS2 Tuning

I have issues with video  streaming over ROS2 (using fastrtps). Frame drops, slow discovery etc. 
The documentation on this matter in both ROS2 and FastRTPS is, in best case, poor. Ther's a lot
of links to this and that document, but NOT A SINGLE HOWTO.
The network part is ok: Jumbo Frames and write / read UDP buffers.

What I need is:

1) How to set up multicast?
2) How to configure buffer sizes?
3) Discovery is SLOW (takes up to 30 seconds). How to "lock" to a spesiffic network?

This document is work in progress, and is organized chronologically as I go along. I will
sort this out later, if and when I find a solution.

My first task: Getting static or better discovery, so subscribers immediately handles
video frames.

OK, FastRTPS supports automatic loading of 'DEFAULT_FASTRTPS_PROFILES.xml'
this fil could contain all kinds of settings. Probably place it in same directory as executable.

[Full description of xml](https://eprosima-fast-rtps.readthedocs.io/en/latest/xmlprofiles.html#examplexml)

My XML ended up like this:
```

```
