## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/undistort_image.png "Undistored image"
[image2]: ./output_images/Threshold_gradient_x.png "Threshold gradient x"
[image3]: ./output_images/Threshold_gradient_y.png "Threshold gradient y"
[image4]: ./output_images/Threshold_magnitude.png "Threshold magnitude"
[image5]: ./output_images/HLS.png "Threshold S channel"
[image6]: ./output_images/Combined_threshold.png "Combined thresholding"
[image7]: ./output_images/perspective_transform.png "Perspective transformed"
[image8]: ./output_images/Sliding_window.png "Sliding window"
[image9]: ./output_images/Lane_lines_detected.png "Lane lines detected"
[image10]: ./output_images/draw_lane.png "Lane identified"
[image11]: ./output_images/Metrics.png "Lane having metrics"

[video1]: ./project_video_solution.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. (Refer cell 4)

The code for this step is contained in the fourth code cell of the IPython notebook located in "./Advanced Lane Finding.ipynb".  

I started by defining camera_caliberate() which read all the images from camera_cal folder and used objpoints and imgpoints to store 3D points and 2D points of images respectively. I read each image , converted in gray scale and used findChessboardCorners() to find corners of the chessboard which are fed to calibrateCamera() which returns camera calibration and distortion coefficients.


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image. (Refer cell 5)

To demonstrate this step, I used cv2.undistort() and passed the cofficients returned from calibrateCamera() to undistort ./camera_cal/calibration1.jpg which is saved in ./output_images/undistort_image.png

![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.(Refer cell 6-13)

In this step, I applied different color transforms, gradient methods to create threshold binary image which are mentioned below:
abs_sobel_thresh() for directional gradient
![alt text][image2]
![alt text][image3]

mag_thresh() for magnitude of the gradient
![alt text][image4]

hls_select() as the S channel picks up the lines
![alt text][image5]

Later,I binary_thresholding() used to combine the thresholds, and produce the image which will be used to identify lane lines in later steps.
![alt text][image6]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image. (Refer cell 14)

The next step in our pipeline is to transform our sample image to birds-eye view which is defined in perspective_transform()

First, I selected the coordinates corresponding to a trapezoid in the image, but which would look like a rectangle from birds_eye view. Then, I defined the destination coordinates, or how that trapezoid would look from birds_eye view. Finally, Opencv function cv2.getPerspectiveTransform will be used to calculate both, the perpective transform M and the inverse perpective transform Minv. M and Minv will be used respectively to warp and unwarp the video images.

![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial? (Refer cell 15-18)

In order to detect the lane pixels from the warped image, the following steps are performed.

First, a histogram of the lower half of the warped image is created.

Then, the starting left and right lanes positions are selected by looking to the max value of the histogram to the left and the right of the histogram's mid position.

A technique known as Sliding Window is used to identify the most likely coordinates of the lane lines in a window, which slides vertically through the image for both the left and right line.

Finally, using the coordinates previously calculated, a second order polynomial is calculated for both the left and right lane line. Numpy's function np.polyfit will be used to calculate the polynomials.

Once we have selected the lines, we can assume that the lines will remain there in future video frames. detect_lane_lines() uses the previosly calculated line_fits to try to identify the lane lines in a consecutive image. If it fails to calculate it, it invokes fit_polynomial() function which further invokes find_lane_pixels() to perform a full search.

![alt text][image8]
[image9]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane. (Refer cell 19)

I did this in cell 19 to calculate radius of curvature of the lane.

left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly. (Refer cell 20)

First, we will draw the lane lines onto the warped blank version of the image. The lane will be drawn onto the warped blank image using the Opencv function cv2.fillPoly. Finally, the blank will be warped back to original image space using inverse perspective matrix (Minv). 

This step has been implemented in cell 20 in my code in the function `draw_lane()`.  Here is an example of my result on a test image:

![alt text][image10]

#### 7. Display lane boundaries and numerical estimation of lane curvature and vehicle position. (Refer cell 21)

Code for numerical estimation of lane curvature and vehicle position is in function add_metrics_image().This funstion receives an image and the line points and returns an image which contains the left and right lane lines radius of curvature and the car offset.

![alt text][image11]

#### 8. Run pipeline in a video (Refer cell 22)

In this step, we will use all the previous steps to create a pipeline that can be used on a video.

Create the ProcessImage class which allows to calibrate the camera when initializing the class and also keep some track of the previously detected lines.

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_solution.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The problems I encountered were almost exclusively due to lighting conditions, shadows, discoloration, etc. Also there would definitely be an issue in snow or in a situation where, for example, a bright white car were driving among dull white lane lines.

I've considered a few possible approaches for making my algorithm more robust. These include more dynamic thresholding (perhaps considering separate threshold parameters for different horizontal slices of the image, or dynamically selecting threshold parameters based on the resulting number of activated pixels), designating a confidence level for fits and rejecting new fits that deviate beyond a certain amount or rejecting the right fit if the confidence in the left fit is high and right fit deviates too much enforcing roughly parallel fits.
