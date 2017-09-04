**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[undistort]: ./output_images/test-undist.png "Undistorted Calibration Image"
[image0]: ./output_images/0-raw.jpg "Camera Input"
[image1]: ./output_images/1-undistort.jpg "Undistorted Image"
[image2]: ./output_images/2-transform.jpg "Perspective Transform"
[image3]: ./output_images/3-binary.png "Gradient and Threshold"
[image4]: ./output_images/4-sliding.jpg "Sliding Window"
[image5]: ./output_images/5-polyfit.png "Polyfit Equation"
[image6]: ./output_images/6-lane.jpg "Detected Lane"
[image7]: ./output_images/7-readings.jpg "Curvature and Offset"
[video1]: ./project_output.mp4 "Output Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first two code cells of the IPython notebook located in "./Adv_Lane_Detect_P4.ipynb".

The OpenCV functions findChessboardCorners and calibrateCamera are the backbone of the image calibration. A number of images of a chessboard, taken from different angles with the same camera, comprise the input. Arrays of object points, corresponding to the location (essentially indices) of internal corners of a chessboard, and image points, the pixel locations of the internal chessboard corners determined by findChessboardCorners, are fed to calibrateCamera which returns camera calibration and distortion coefficients. These can then be used by the OpenCV undistort function to undo the effects of distortion on any image produced by the same camera. Generally, these coefficients will not change for a given camera (and lens).  

The below image depicts the corners drawn onto twenty chessboard images using the OpenCV function drawChessboardCorners:

![alt text][undistort]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
The image below depicts the results of applying undistort to one of the project dashcam images:
![alt text][undistort]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in "./Notebook.ipynb".  The `perspective_transform()` function takes as inputs an image (`image`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([
  [(width/2) - size_top, height*0.65],
  [(width/2) + size_top, height*0.65],
  [(width/2) + size_bottom, height-50],
  [(width/2) - size_bottom, height-50]
])
dst = np.float32([
  [(width/2) - output_size, (height/2) - output_size],
  [(width/2) + output_size, (height/2) - output_size],
  [(width/2) + output_size, (height/2) + output_size],
  [(width/2) - output_size, (height/2) + output_size]
])

```
This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 570, 468      | 280, 0        |
| 710, 468      | 1000, 0       |
| 1010, 670     | 1000, 720     |
| 270, 670      | 280, 720      |

To verify that my perspective transform was working as expected I drew  the `src` and `dst` points onto a test image and its warped counterpart.  As expected,  the lines appear to be parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First, I checked the histogram of the lower of image and find the two peaks for the left and right lines. Then, I used  the sliding window method to work my way upwards and found the relevant points in the image which marked the lane. Finally, I used the `np.polyfit()` method to fit a second degree polynomial to these points. I did this in the `fit_polynomials()` function.

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Given the detected points, I determined a second degree polynomial that fitted these points, and then I calculated the radius of curvature of this polynomial. Also, I converted from the pixel space to real space in meters. I did this in `get_curvature()` function.

![alt text][image7]

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the code cells titled "Draw the Detected Lane Back onto the Original Image" and "Draw Curvature Radius and Distance from Center Data onto the Original Image" in the Jupyter notebook. A polygon is generated based on plots of the left and right fits, warped back to the perspective of the original image using the inverse perspective matrix Minv and overlaid onto the original image. The image below is an example of the results of the draw_lane function:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_output_3.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Selecting optimal parameters for the gradient thresholds has been a challenge partly because we would like to do it in a way that works for a wide number of scenarios.

Producing a pipeline from which lane lines can reliably be identified was of utmost importance , but smoothing the video output by averaging the last n can be a logical solution.

Also, to make the pipeline more robust, we should check  that the radius of curvature is similar for both the left and right lane lines.

I hope to revisit some of these strategies in the future.
