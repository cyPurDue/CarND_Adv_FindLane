## Project Writeup

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

[image1]: output_images/chess_board_images/chess_board_undistorted.png "Undistorted Chessboard"
[image2]: output_images/straight_lines1/distortion_corrected_sl1.png "Undistorted"
[image3]: output_images/straight_lines1/combined_binary_sl1.png "Undistorted"
[image4]: output_images/straight_lines1/birds_eye_sl1.png "Undistorted"
[image5]: output_images/straight_lines1/fit_to_box_sl1.png "Undistorted"
[image6]: output_images/straight_lines1/fit_lane_sl1.png "Undistorted"
[image7]: output_images/straight_lines1/output_sl1.png "Undistorted"
[video1]: out3.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. The entire repo's structure is like this:
* "adv_lane_detection.ipynb": code file
* "out3.mp4": output video file
* "output_images/": output image folder, which then includes:
&nbsp;&nbsp;&nbsp;&nbsp; -- "README.txt": readme txt file
&nbsp;&nbsp;&nbsp;&nbsp; -- "chess_board_images/": 2x images, board corner detection and distortion corrected for a sample chessboard image.
&nbsp;&nbsp;&nbsp;&nbsp; Each of all other folders includes 6x test results based on every test image from test_image/ folder. 6x images are: distortion correction, combined binary, birds eye, fit lane to box, fit lanes, and output.
&nbsp;&nbsp;&nbsp;&nbsp; -- "straight_lines1/": 6x images based on test_images/straight_line1.jpg
&nbsp;&nbsp;&nbsp;&nbsp; -- "straight_lines2/": 6x images based on test_images/straight_line2.jpg
&nbsp;&nbsp;&nbsp;&nbsp; -- "test1/": 6x images based on test_images/test1.jpg
&nbsp;&nbsp;&nbsp;&nbsp; -- "test2/": 6x images based on test_images/test2.jpg
&nbsp;&nbsp;&nbsp;&nbsp; -- "test3/": 6x images based on test_images/test3.jpg
&nbsp;&nbsp;&nbsp;&nbsp; -- "test4/": 6x images based on test_images/test4.jpg
&nbsp;&nbsp;&nbsp;&nbsp; -- "test5/": 6x images based on test_images/test5.jpg
&nbsp;&nbsp;&nbsp;&nbsp; -- "test6/": 6x images based on test_images/test6.jpg

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell In[8] to In[11] of the IPython notebook adv_lane_detection.ipynb.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I wrapped it in a single function which takes all the calibation images and return matrices.

As a sample test, I applied the distortion correction to one of the chessboard image using the `cv2.undistort()` function in cell In[13]. Result image is exported to output_images/chess_board_images/ . A sample is shown as below: <br />

![alt text][image1] <br />

### Pipeline (single images)
Refer to sub-titles on the .ipynb code, the entire pipeline is from section 2) to section 7). 
In section 7+), I have summarized a compact pipeline that will do all the procedure but only show final output. 

Please note that here I only use straight line 1 result as a sample illustration, for all the results, please go to output_images/ and see all sub-folders for all result images.

#### 1. Provide an example of a distortion-corrected image.
This is in section 2) of the code. Basically I applied cv2.undistort to the test image, using the correction parameters calculated from chessboard images. A sample result is shown below: <br />

![alt text][image2] <br />

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
This is in the section 3) of the code. I used a combination of color and gradient thresholds to generate a binary image. First I used s color channel with threshold (170, 255) to get color binary image, then used 'x' direction to generated gradient binary image with threshold (10, 160). Also I have tried 'y' direction but did not see obvious help at the final result, and sometimes may lead to unexpected result, so I did not use it then.

These two binary results are then use logic OR to combine together and generate combined binary result. A sample is shown below: <br />

![alt text][image3] <br />

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

This is the section 4) of the code, called birdsEye(), using cv2.getPerspectiveTransform to get M matrix. Also used cv2.getPerspectiveTransform to calculate inversed matrix to be used in the last section output. Due to my first finished pipeline, I found especially on the bridge the lane detection drifted so I have tunned here very carefully to make sure the very left shadow lane (not traffic lane) is masked out, so I have artificially input source and destication coordinates as:  

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595, 450      | 270, 0        | 
| 685, 450      | 1010, 0       |
| 1080, 720     | 1010, 720     |
| 200, 720      | 270, 720      |

From the straight_line output shown below, I have verified the two lanes are parallel on the birds eye image. <br />

![alt text][image4] <br />

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

This is the section 5) of the code. Following the code learned in the class, I used 9x sliding windows to fit the lane into boxes based on histogram. Also used margin = 100, minpix = 50, and 2nd-order polynomial fit np.polyfit(_,_,2). Sample image is shown below: <br />

![alt text][image5] <br />

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This is in the section 6) of the code. Following the code learned in class, and using the fit results from the previous section, with margin = 100, to calculate left_fit, right_fit, leftx, lefty, rightx, righty. Sample result is shown below, as drawing the fitted lane to the warped binary image. <br />

![alt text][image6] <br />

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is in the section 7) of the code. Draw an image to show result, then recast to x and y and use cv2.fillPoly() to draw onto the warped image. Use the invsere matrix Minv that was calculated above to transfer it back to original image. To display the result, I have gone through online references and leanred to use cv2.putText() help put results on top of the image. Sample result is shown below: <br />

![alt text][image7] <br />

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!). 

Here's a [link to my video result](out3.mp4) <br />

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I have gone through this pipeline relatively smooth, except 1 item that costed me a lot of time. In the final video test, at around middle of the time, there is a stone bridge and my left lane detection falled onto the shadow of the bridge curb, rather than following the yellow lane. I have tuned my color binary parameter and added y direction to binary but did not help much. So I started to tune the parameter of the birds eye and lane fitting. I have used smaller margin value on the lane fitting which kind of solved the problem, but at the final 1/4 of the video, I found a little frames of fitting was too biased so I did not use it. Then I have tried the perspective transformation, and since the shadow of the bridge railings is on the very left, so I haved changed the destination of the birds eye to be a bit away from the center, and hence cropped the side out. This worked much better especially for rejecting side disturbance. However, in this case, the lanes are closer to the side of image, and if the turn is very sharp, the curve may not be fully presented on the birdes eye and lead to some fitting error of sharp turns.
