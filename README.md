
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

[image1]: ./output_images/chessboard_undistort.png "Chessboard Calibration"
[image2]: ./output_images/road_distortion_calibration.png "Road Calibration"
[image3]: ./output_images/thresholded_binary_image.png "Threholded Binary Example"
[image4]: ./output_images/warpped_image.png "Warped Image Example"

[image5]: ./output_images/laneline_fitted.png "Fit Visual"
[image6]: ./output_images/annotated_image.png "Annonated Image"
[video1]: ./project_video.mp4 "Fit Visual"

---

###Camera Calibration

####1. Compute the camera calibration using chessboard images. 
#####p4.ipynb Section 1

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

####2. Apply a distortion correction to raw chessboard images.
#####p4.ipynb Section 2-1

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

####1. Apply a distortion correction to each raw roard image.
#####p4.ipynb Section 2-2
Again `objpoints` and `imgpoints` that were previously computed when calibratin the camera is used to perform distortion correction to each raw images.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

####2. Use color transforms, gradients, etc., to create a thresholded binary image.
#####p4.ipynb Section 3 (thresholded_img class)
The imgage is then put through a pipeline to create a thresholded binary image. Basically Y channel from YCrCb space is used for detecting white line and the Cb channel is used for yellow line detection; h channel is also used in conjunction as a filter to take out shadow pixles. To create a threholded binary each of these chanel is also fed through Sobel threshold algorithm to identify, 
 - x-axis gradient
 - y-axis gradient
 - absolue gradient gradient
 - directional gradient. 
 
A combined result of the differnet gradient scheme on each channel returns a output binary imiage as the follow example,

![alt text][image3]

####3. Apply a perspective transform to rectify binary image ("birds-eye view")
#####p4.ipynb Section 2 `thresholded_img.warp_img()` ; p4.ipynb Section 4
The code for warping the image is implemented also into the `thresholded_img` class called `warp_img()`. The  function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 540, 470      | 200, 0        | 
| 750, 470      | 1080, 0      |
| 1130, 690     | 1080, 720      |
| 200, 690      | 200, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Detect lane pixels and fit to find the lane boundary and fitted with a polynomial.
#####p4.ipynb Section 5
From the warpped image I could divide the lane line points into left and right lane line points. Then I fitted 2nd order polynomial to each lane line using `numpy.polyfit()` function. For example, 

![alt text][image5]

####5. Determine the curvature of the lane and vehicle position with respect to center and warp the detected lane boundaries back onto the original image.
#####p4.ipynb Section 6,7 `anonate_img`
Using the fitted 2nd order polynomial parameters, the lane can be drawn and warp back to the original undistorted image. At the same time vehicle position and curvature of the lane can be found.
![alt text][image6]

---

###Pipeline (video)

####1. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position
#####p4.ipynb Section 6,8

A few extra features are added to the image processing pipeline to improve the performance of the video processing pipeline:
 - Dynamically selection for region of interest selection if lane lines were successflly found previously
     - `set_dynimic_region_vertices()`
 - Dynamically tighten region of interest again after first identifying the lane line poinst. This help reduces the possibility of identify noise as lane line points. 
     - `dynamic_tighten_region()`
 - Runtime averaging of a few lane line fitting polynomial constants. 
     - `set_leftfit()` and `set_rightfit()`

![Final Result Gif] (./result.mp4)

---
##Discussion

My general approach is to process the image as best as I could then apply it to the video. While processing the video a few extra features are also added to increase the performance. It is relatively easy to process images/videos where road condition is nice and clean. However, when it comes to shadows, road material change or cases where background road colour is relative light, the pipeline does not work as well.


There are still lots of places where the pipeline can be improved. Following are just a few ideas that could imporve the pipeline:
 - Explore different colour space
 - Parameter tuning.
 - Explore different way of removing pixel noise or low pass filtering. Maybe even something in the laplacian space
 - Implement a better weighted average for polynomial constant smoothing/averaging. Possibly something similar to a PID control scheme.


