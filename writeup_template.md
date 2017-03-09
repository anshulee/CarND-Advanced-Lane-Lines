##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./examples/chessboardundistort.png "Undistorted chessboard"
[image2]: ./examples/roadImageUndistort.png "Road ImageTransformed"
[image3]: ./examples/gradient.png "Gradient applied"
[image4]: ./examples/SChannel.png "SChannel applied"
[image5]: ./examples/Stacked.png "Stacked Gradient and SChannel applied"
[image6]: ./examples/perspectiveSingle.png "Road perspective applied"
[image7]: ./examples/perspectiveGradient.png "Gradient perspective applied"
[image8]: ./examples/drawOnRoad.png "Draw back  applied"

[image9]: ./examples/drawWithRadius.png "draw back with radius "
[image10]: ./examples/window.png "Sliding Window  applied"



[video1]: ./project_video_output4.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the  cells 3,4 and 5 of the IPython notebook titled AdvancedLaneFinding-Working. I read the calibration images provided with the repository using  the glob.glob('camera_cal/calibration*.jpg') . A visual inspection of the images showed them to have 6 by 9 corners and i used that info to instantiate the objectPoints array. I then  used the cv2.findChessboardCorners method to collect an array of imagepoints. I then used the imagePoints and objectPoints to calibrate the camera using cv2.calibrateCamera. I then used cv2.undistort to undistort the images.

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
Code is present in section labelled "Use color transforms, gradients, etc., to create a thresholded binary image."
I used a combination of Sobelx, magnitude threshold and direction threshold to get a proper gradient across all test images. I iterated through all the test images to see that the combined gradient threshold gave a uniform output and configured parameters accordingly.

![alt text][image3]

I then used the HLS color space S Channel output and combined it with the gradient output above to generate a binary image

# SChannel Applied

![alt text][image4]

# Gradient and SChannel Stacked

![alt text][image5]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp()`, which appears section labelled "Perspective Transform:- * Apply a perspective transform to rectify binary image ("birds-eye view")."
 The `unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points by visual inspection of the images . I then tested all the pipeline images to see the result

# Perspective tranform on a road image
![alt text][image6]

# Perspective transform on a gradient applied image
![alt text][image7]


####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the Sliding Window functionality to determine lanes lines further and code is present in section labelled "# Detect lane pixels and fit to find the lane boundary." I tested the detected lanes with about 10 windows and iterated through all the pipeline images to see how it performed. As per the labs i also set logic to not use sliding window for each frame and only when lines are not being detected properly using the previous frames 

![alt text][image10]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in section labelled "Determine the curvature of the lane and vehicle position with respect to center" The radius of curvature is based upon a formula i saw in the forums using this line of code (altered for clarity):

curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])
In this example, fit[0] is the first coefficient of the second order polynomial fit, and fit[1] is the second (y) coefficient. y_0 is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). y_meters_per_pixel is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters.

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:

lane_center_position = (r_fit_x_int + l_fit_x_int) /2
center_dist = (car_position - lane_center_position) * x_meters_per_pix
r_fit_x_int and l_fit_x_int are the x-intercepts of the right and left fits, respectively. 

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in section labelled # Warp the detected lane boundaries back onto the original image. It basically uses the inverse perspective matrix to map the gradient back onto the image 

![alt text][image9]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output4.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had issues in adjusting gradients for the various images especially where there were shadows etc. Also the gradient kind of falters when the car turns. I had to change code to go back to Sliding window detection when the distance between the parallel lines varies by more than 50 units. I had earlier set this to 100 but the gradient was going off the lines. Setting it to 50 caused the sliding window detection to happen way too many times and i think that is sub-optimum and an overall tweaking of paramters and algorithm all over the project might help in increasing this range.
