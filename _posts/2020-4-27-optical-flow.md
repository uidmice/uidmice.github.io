---
layout: post
title: Optical flow for stream velocity estimation
tag: [algorithm, iot]
---
We have developed a prototype implementation in python that uses OpenCV to identify the Flow Rate of a stream from a video by using the Lucas-Kanade Optical Flow detection technique iteratively.

This implementation extracts features from individual frames of the video (using a Shi-Tomasi Filter) and tracks their motion using correlation filters between successive frames. As a feature moves from one point to another across successive frames, we keep track of its motion and compute the distance/direction it has moved (in terms of pixels). We then average out the direction vectors for all the features being tracked in the frame to obtain a mean direction vector for the stream (in pixels/frame) at a given instant of time.

![demo]({{ site.baseurl }}/images/opticalflow.gif)

## Possible Techniques for Optical Flow Detection

Some well known techniques for Optical Flow Detection that we considered are:
* **Simple Flow Method**: This is a non-iterative Optical Flow Detection Method that uses linear interpolation to estimate flow and can be efficiently implemented on parallel architectures such as GPUs.
* **Lucas-Kanade Method**: This method assumes that the flow is essentially constant in a local neighbourhood of the pixel under consideration, and solves the basic optical flow equations for all the pixels in that neighbourhood, by the least squares criterion. IT works well when applied iteratively to a sparse feature set.
* **Gunnar Farneback Method**: The Farneback algorithm generates an image pyramid, where each level has a lower resolution compared to the previous level. When you select a pyramid level greater than 1, the algorithm can track the points at multiple levels of resolution, starting at the lowest level. Increasing the number of pyramid levels enables the algorithm to handle larger displacements of points between frames. However, the number of computations also increases. This is a dense optical flow algorithm, i.e., it requires a dense feature set and large number of computations.

Eventually, we went ahead with the Lucas-Kanade method since it was designed for a sparse feature set which seems to be better than dense optical flow detection when considering power constraints on embedded devices.

## Optimizations used in the Optical Flow Implementation

Some optimizations used in the Optical Flow Implementation are:
* Run algorithm on stripped down frames (Consider frame slice that contains start of the flow, i.e. upstream image slice): We tried reducing the search space in each frame by slicing images so that we only monitor the part of the image that has upstream data.

* Upper limit on the maximum number of features detected in the frame: We observed that the computational cost of the algorithm increases with increase in the number of features being tracked across frames. Hence, we imposed an upper limit on the number of features we detect in each frame so as to reduce the computations.

* Track features across alternate frames instead of all frames: We tried to down sample the input video by a factor of 2 or 3 before processing it so as to reduce the computational cost. This means we process 1 out of 2 or 3 frames instead of all frames. The downside with this approach could be a loss in accuracy in case the video is not high resolution or high fps.

* Clear tracking data after select intervals of time to avoid tracking a feature for too long

## Accuracy vs. Efficiency Trade-offs in Optical Flow algorithm
[Reference](http://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/LIU2/ECCV96.html)
