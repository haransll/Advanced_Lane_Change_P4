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

The code for this step is contained in the first few code cells of the IPython notebook located in "./Notebook.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][undistort]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
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

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

We first check the histogram of the lower of image and find the two peaks for the left and right lines. Then we use the sliding window method to work our way upwards and find the relevant points in the image which mark the lane. Next, we use the `np.polyfit()` method to fit a second degree polynomial to these points. I did this in the `fit_polynomials()` function.

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Given the detected points, we determine a second degree polynomial that fits these points, and then we calculate the radius of curvature of this polynomial. Also, we convert from the pixel space to real space in meters. I did this in `get_curvature()` function.

![alt text][image7]

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_output.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Selecting optimum parameters for the gradient thresholds is a challenge - because we want to do it in a way that works for a wide number of scenarios. This was achieved by testing it on the 6 test images that are given.

The pipeline currently fails on the test images, mostly because we don't keep state of previous video frames as of now. We can make the processing more efficient by making it remember the last detected lane line and then using that as a hint for detecting the lane in the next video frame.

Also, to make it more robust, we should add checks like checking that the radius of curvature is similar for both the left and right lane lines. Also, it should continue to work if just a single lane line is visible.
