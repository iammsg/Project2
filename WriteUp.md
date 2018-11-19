
# Advanced Lane Finding Project

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

[image1]: ./output_images/camera_distortion.png "Undistorted"
[image2]: ./output_images/Undistorted_image.png "Road Transformed"
[image3]: ./output_images/combined_binary.png "Binary Example"
[image4]: ./output_images/warped_image.png "Warp Example"
[image5]: ./output_images/Curvefitted_image.png "Fit Visual"
[image6]: ./output_images/Final_image.png "Output"


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
Camera calibration process is used to transform a 3D point in the real world to a 2D point on the image plane considering parameters like focal length of camera, distortion, resolution, shifting of origin. For this purpose, we utilize OpenCV functions findChessboardCorners and caalibrateCamera. A number of images of a chessboard, taken from different angles with the same camera, comprise the input. From these we identify the `object points` which are the $(x, y, z)$ coordinates of the chessboard corners in the real world. The chessboard is fixed on the $(x, y)$ plane at $z=0$, such that the object points are the same for each calibration image.  Thus, `objp` is a replicated array of coordinates, and `objpoints` will be appended with a copy of objp every time chessboard corners are successfully detected in a test image.  `imgpoints` will be appended with the $(x, y)$ pixel position of each of the corners in the image plane with each successful chessboard detection. These functions will return camera calibration and distortion coefficients. The code for this can be found in `StartupCode.ipynb` in cells number 3 and 4.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
The mtx and dst matrices from calibrateCamera function can then be used by the OpenCV `cv2.undistort()` function to undo the effects of distortion on any image produced by the same camera as shown below. Generally, these coefficients will not change for a given camera (and lens). This can be identified in cell number 5 in `StartupCode.ipynb` 
![alt text][image2]




#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Various combinations of color and gradient thresholds are used to generate a binary image where the lane lines are clearly visible. I have used gradient on x and y direction, manginutude of the gradient, direction of the gradient, and color transformation techniques to get the final binary image.To uniquley identify the contribution of each thresholding method, RGB color spaces are used.
        * Red - Gradient on x and y direction, manginutude of the gradient, direction of the gradient
        * Green - Image with l channel thesholding
        * Blue - Image with s channel thesholding
For this purpose,four different functions are defined:
        * abs_sobel_thresh(): to threshold the gradients in the x and y direction
        * mag_thresh() : to threshold the magniture of the gradient
        * dir_thresh() : to threshold the direction of the gradient
        * hls_thresh() : to threshold the images in the HLS color space

![alt text][image3]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_image()`, which appears in the second cell along with the rest of the functions. The function takes as inputs an image (`img`) and for the sake of the exercise, the source and destination points were hardcoded as:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
|  570, 470     |  200, 0       | 
|  720, 470     |  200, 680     |
|  260, 680     | 1000, 0       |
| 1045, 680     | 1000, 680     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For the purposes of the single image pipeline, I created a function called `find_lane_pixels`(cell 2 - `StartupCode.ipynb`) which identifies lane-line pixels by plotting a histogram of where the binary activations occur across the image. The functions then uses sliding windows moving upward in the image (further along the road) to determine where the lane lines go. I chose the number of windows to use to be 9, with the window margin to be $\pm 100$ and the threshold to recenter the window to be a 100 detections. 

The fitting of these lane pixels were done by anoter function called `fit_polynomial` which calls the `find_lane_pixels` function and then fits a second degree polynomial to these points. 

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the radius of the curvature, I created another function called `radius_curvature`(cell 2 - `StartupCode.ipynb`). This function utilizes the previously attained coeffecients for the second degree polynomial fit to the lane, to calculate the curvature using the formula:
                     ]$R_{curve} = ((1 + (2Ay+B)^{2)})^{3/2})/(|2A|)$

The same function also calculates the position of the vehicle with respect to the center. This is done by calculating the bottom point of both left & right lanes, then  calculating the midpoint of these two points and then deriving the deviation from the center of the frame ($x = 640$) which I assume to be the camera position. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The above steps were consolidated into a single function called `process_image` found in `StartupCode.ipynb`. This function takes in an image and then returns the same image with lanes drawn on it alongwith the values of position of the car and the curvature of the lane.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

To generate my final video, I used all the methods I had outlined above. Doing this as is, i.e. treat each frame separately and generate lanes independently, delivered reasonably good results. There were some instances however where they failed:
    * The right lane marker was faded
    * Excessive shadows

As a way to improve this, I implemented a line class that kept tabs on whether a good fit was detected on the previous frame and store those co-effecients. This allowed me to do a guided search around the polynomial rather than doing a search from scratch using histograms for each frame. This method also prevents the capture of spurious lane edges when the actual lanes are faint. 

There are still other scenarios where this pipeline can fail, as evidenced by the *challenge_video*. Some of the key reasons:
    * Excessive Shadows
    * Color variations across the lane
    * Sharp curves
Some of these can be alleviated by:
    * Better dynamic threshold controls
    * Smoothing across previous frames 
    * Including other color spaces
    * Fitting the using higher order polynomials