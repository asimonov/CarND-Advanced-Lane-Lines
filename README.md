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

Next `LaneDetection` class is defined to detect both lane lines given unprocessed color images
(individual or from video stream).
It keeps track of both detected lines using `Line` objects. It also checks for lines to 
be parallel and reasonable distance apart. 
It relies on Line objects holding the state from image to image and using it for successful
detection in subsequent images.
If detection fails in few consequtive frames
it tries to detect lines afresh using histogram window.
The lane with low confidence of right detection is annotated with lines shown in red.
Frames with high confidence detection have lines shown in blue.



## Rubrics Comments

### Camera Calibraion

**_OpenCV functions or other methods were used to calculate the correct camera matrix and distortion coefficients using the calibration chessboard images provided in the repository. The distortion matrix should be used to un-distort the test calibration image provided as a demonstration that the calibration is correct_**

**Alexey Simonov**: 
`cv2` functions `drawChessboardCorners` and `calibrateCamera` used. See first part of the notebook.



### Pipeline (Single Images)

**_Distortion correction that was calculated via camera calibration has been correctly applied to each image._**

**Alexey Simonov**:
Function `undistort` is called as part of pipeline in `LaneDetector.process_image` for each processed image.



**_At least two methods (i.e., color transforms, gradients) have been combined to create a binary image containing likely lane pixels. There is no "ground truth" here, just visual verification that the pixels identified as part of the lane lines are, in fact, part of the lines._**

**Alexey Simonov**:
The function that creates binary image is `threshold_binary`.
I have combined: 
* S channel in HLS color space (see `hls_s_thresholds`), 
* sobel transforms for X and Y (`sobel_abs_thresholds`), 
* sobel gradient magnitude (`sobel_magnitude_thresholds`),
* sobel gradient direction (`sobel_gradidir_thresholds`)

They are combined as:

`binary = (sx & smag) || ( (sy & sgrad) || ( (smag & sgrad) || (sx || s) ) ) `

where:
* sx is sobel x, 
* smag is sobel gradient magnitude, 
* sy is sobel y, 
* sgrad is sobel gradient,
* s is S channel in HLS space

I have found this emprically superior to other combinations I tried. The resulting image has quite distinct lane lines that are good input for later stages in the pipeline.



**_OpenCV function or other method has been used to correctly rectify each image to a "birds-eye view"_**

**Alexey Simonov**:
The function that creates 'top-down' image is `perspective_transform`.
It calls `getPerspectiveTransform` and `warpPerspective` functions from `OpenCV`.
I have found the shape of the area to be transformed using empirical tests -- looking for resulting top down image to have lane lines as parallel as possible on provided 6 test images.



**_Methods have been used to identify lane line pixels in the rectified binary image. The left and right line have been identified and fit with a curved functional form (e.g., spine or polynomial)._**

**Alexey Simonov**:
Two functions from `Line` class do this.
`find_left_right_x` (static) method uses histogram to find most probably locations for left and right lines in the 'top-down' view of the road in front. It does some basic checks to estimate the confidence that the lines are detected correctly. If returns two x coordinates -- one for each line. If no good candidates found one or both x returned are None. This is a static method so it does not change state of Line object on which it is called.
`fit_from_x_on_image` method is given initial x coordinate at the bottom of the image to then apply sliding window procedure to find other pixels along the potential line. It then fits second order polynomial through these points.
`use_last_good_fit` method is similar to `fit_from_x_on_image` but is NOT given initial x coordinate. It uses the last fit results to draw line on new image. It is called from the outside by `Detector` class when two detected lines are either too close or not parallel. The lines shown using this method are denoted in red by `Detector` to signify low confidence of detection.



**_Here the idea is to take the measurements of where the lane lines are and estimate how much the road is curving and where the vehicle is located with respect to the center of the lane. The radius of curvature may be given in meters assuming the curve of the road follows a circle and the position of the vehicle within the lane may be given as meters off of center._**

**Alexey Simonov**:
Function `calc_radius_of_curvature` calculates radius of curvature using polynomial coefficients. It calculates that in both pixel and meter coordinates. The position of the line compared to vehicle center is calculated inside




**_The fit from the rectified image has been warped back onto the original image and plotted to identify the lane boundaries. This should demonstrate that the lane boundaries were correctly identified._**

**Alexey Simonov**:
`LaneDetector.annotate_undistorted_image` function warps fitted polynomials and the region between them back from 'top-down' image into original undistorted image.
`LaneDetector` class also has the ability to show 'diagnostic' view, combining the final annotated image with binary thresholded image, top-down view and annotated top-down view to easier identify where pipeline struggles.



### Pipeline (Video)

**_The image processing pipeline that was established to find the lane lines in images successfully processes the video. The output here should be a new video where the lanes are identified in every frame, and outputs are generated regarding the radius of curvature of the lane and vehicle position within the lane. The identification and estimation don't need to be perfect, but they should not be wildly off in any case. The pipeline should correctly map out curved lines and not fail when shadows or pavement color changes are present._**

**Alexey Simonov**:
The notebook produces `project_video_annotated.mp4` video, using the provided video of the driving. We can see in the resulting video that detection is achieved after first 5 frames. And it is maintained for the duration of the video. In couple of moments the confidence in detected lines drops and they are show in red. The detection results from previous frames are used in this case, up to 5 failed frames. At which point the pipeline is trying to detect initial x coordinates afresh using histogram method.



**_In the first few frames of video, the algorithm should perform a search without prior assumptions about where the lines are (i.e., no hard coded values to start with). Once a high-confidence detection is achieved, that positional knowledge may be used in future iterations as a starting point to find the lines._**

**Alexey Simonov**:



**_As soon as a high confidence detection of the lane lines has been achieved, that information should be propagated to the detection step for the next frame of the video, both as a means of saving time on detection and in order to reject outliers (anomalous detections)._**

**Alexey Simonov**:



### README

**_The Readme file submitted with this project includes a detailed description of what steps were taken to achieve the result, what techniques were used to arrive at a successful result, what could be improved about their algorithm/pipeline, and what hypothetical cases would cause their pipeline to fail._**

**Alexey Simonov**:




