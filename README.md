## Advanced Lane Finding.
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

The goals / steps of this project are the following:  

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply the distortion correction to the raw image.  
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view"). 
* Detect lane pixels and fit to find lane boundary.
* Determine curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle 
position.

---

The images for camera calibration are stored in the folder called `camera_cal`.  
The images in `test_images` are for testing your pipeline on single frames.  
The video called `project_video.mp4` is the video your pipeline should work well on.  
`challenge_video.mp4` is an extra (and optional) challenge for you if you want to 
test your pipeline.

If you're feeling ambitious (totally optional though), don't stop there!  
We encourage you to go out and take video of your own, calibrate your camera and 
show us how you would implement this project from scratch!

---

## Project Implementation by Alexey Simonov.

The entire project is implemented in jupyter notebook called `lane-detection-pipeline-final.ipynb`

First part of the notebook loads the camera calibration images of a chessboard pattern
and uses `cv2` functions `drawChessboardCorners` and `calibrateCamera` to find the calibration
matrix and distortion coefficients. 
These parameters are saved in `calibration_pickle.p` file.

Then I define functions to `undistort` image and function `thresholded_binary` 
to create a binary image from a color image which is most suitable for lane detection later. 
It combines function calls for sobel transforms, gradients/gradient magnitudes 
and S channel from HLS color space in a manner
that I empirically found to produce good results on various given test images.

Next is `perspective_transform` function that has a shape of transform area defined to
cover the part of the road in front of the car, that is rectangular and covers both lane lines
and goes some distance in front. This area is visualised in the notebook.
It gets transformed into a 'bird-eye' view (where lines should be parallel) 
for subsequent line detection.

Then I define `Line` class to detect one lane line in a binary image, given input bottom 
x coordinate
of its most probable position. The instances of this class hold the information about 
detection results between calls and re-use it if there are difficulties in line detection
in subsequent calls.

Next `LaneDetection` class is defined to detect both lane lines given unprocessed color image.
It keeps track of both detected lines using `Line` objects. It also checks for lines to 
be parallel and reasonable distance apart. 
It relies on Line objects holding the state from image to image and using it for successful
detection in subsequent images.
If detection fails in few consequtive frames
it tries to detect lines afresh using histogram window.


