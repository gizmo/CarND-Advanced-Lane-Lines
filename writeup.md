## Writeup Template

---

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

[image1]: ./output_images/distorted_checkerboard.png "Distorted Checkerboard"
[image2]: ./output_images/undistorted_checkerboard.png "Undistorted Checkerboard"
[image3]: ./output_images/binary_combo.png "Binary Combo"
[image4]: ./output_images/original_straight.png "Original Straight"
[image5]: ./output_images/perspective_straight.png "Perspective Straight"
[image6]: ./output_images/original_curved.png "Original Curved"
[image7]: ./output_images/perspective_curved.png "Perspective Curved"
[image8]: ./output_images/histogram.png "Histogram"
[image9]: ./output_images/poly_segmented.png "Poly Segmented"
[image10]: ./output_images/poly_window.png "Poly Window"
[image11]: ./output_images/poly_fit.png "Poly Fit Visual"
[image12]: ./output_images/color_fit_lines.png "Color Fit Lines"
[image13]: ./output_images/example_output.png "Output"
[video1]: ./output_iamges/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/gizmo/CarND-Advanced-Lane-Lines/blob/master/writeup.md) is writeup for this project.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "./advanced_lane_finding.ipynb" (second code cell: lines 3 through 35).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image6]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cell 2: lines 54 through 174). Note that the function generate_binary_image(img) was the result of experimentation seen in cell 3.  This involved experimentation of binary combinations of sobel gradient directions, gradient magnitudes, and color channels: RGB and HLS.  The selected combination was a R and S color channels and gradient in x.   Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in cell 2: lines 176 through 183 of the IPython notebook.  The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points based on the straight lane image, 'test_images/straight_lines1.jpg'.  :

```python
perspective_src = np.float32(
    [[586.0, 456.5],
     [695.5, 456.5],
     [206., 720.],
     [1099., 720.]])
perspective_dst = np.float32(
    [[290.,  0.],
     [1099., 0.],
     [290.,  720.],
     [1099., 720.]])


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.  I tested this on a straight lane image and curved lane image.

![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To identify the lane line I first computed a histogram based on the warped perspective of the binary combination image.  The histogram is shown for a curved image as shown here:

![alt text][image6]
![alt text][image8]

Using this histogram I locate the two highest peaks of this histogram representing the left and right lanes.  Using these peaks as a starting point for the lanes I apply a sliding window beginning at the bottom of the image and working my way up.  For each window I capture the indices for both left and right lane windows.

![alt text][image9]

I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image12]
![alt text][image11]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I compute the curvature in meters in cell 14 in a function called, 'curvature_meters()'.
I computed the center offset in meters in cell 16 in a function called, 'center_offset()'.  A negative value is left of center and a postive value is right of center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell 17 lines 1 through 29.  Here is an example of my result on a test image:

![alt text][image13]

As part of the pipeline cell 18 perform the above steps in the function process_image() which takes as input an image file that is fed in by VideoFileClip.fl function.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

There are instances in the video outptu where the lane detection jumps around for a few frames as a result a white streak in the middle of the road that isn't a lane line.
One way to address would be to track the previous lines and try to ensure the lines more or less remain parallel and that the mean distance between the lines does not suddenly change by a large amount.
