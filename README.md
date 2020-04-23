# Advanced Lane Finding

This is my second project in the Udacity Nanodegree program [Self-Driving Car Engineer](https://www.udacity.com/course/self-driving-car-engineer-nanodegree--nd013)!

The code is in [Project.ipynb](https://github.com/dingchen-github/AdvancedLaneLines/blob/master/Project.ipynb) and the implementation explanation is in [writeup.md](https://github.com/dingchen-github/AdvancedLaneLines/blob/master/writeup.md).


The Project
---

The goals of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The `project_video.mp4` video is the one that this project should work well on.  
The `challenge_video.mp4` video is an extra (and optional) challenge to test the pipeline under somewhat trickier conditions.
The `harder_challenge_video.mp4` video is another optional challenge and is brutal!

The outputs are called `project_output.mp4`, `challenge_output.mp4` and `harder_challenge_output.mp4`. Check them out!

## `%matplotlib qt`

Udacity example code has always used `%matplotlib qt`, but I could not use pyqt5 ("Failed to import any qt binding"), although I have installed it. After searching online for more than one hour and trying numerous methods, including updating *matplotlib*, it still didn't work. Finally [here](https://stackoverflow.com/questions/41046299/pop-up-plots-using-python-jupyter-notebook), I got the solution - just use `%matplotlib tk`, then I can display images in a new pop-up window!
