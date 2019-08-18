## Advanced Lane Finding Project


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

[image1]: ./test_images/test5.jpg "Distorted image"
[image2]: ./readme_images/undistorted_image.jpg "Undistorted image.jpg"
[image3]: ./readme_images/combined_threshold.jpg "Combined Threshold"
[image4]: ./readme_images/warped_image.jpg "Warped Image"
[image5]: ./readme_images/blind_search.jpg "Blind lane boundry search"
[image6]: ./readme_images/search_from_prior.jpg "Search around previous polynomial"
[image7]: ./output_images/with_lane_markings_test5.jpg "Image with Lane Markings"


[video1]: ./out_video.mp4 "Video"


The code for this step is contained in the first code cell of the IPython notebook located in "./advanced_lane_finding.ipynb" 


### Camera Calibration

The first step in the pipeline is to calibrate the camera and correct for the distortion it causes to the detected images. Chessboards are a famous way to achieve this goal since corners can be easily detected. Hence, by translating the corners for the distorted chessboard image to the correct one we could calculate the camera's intrinsic coefficients and use it to undistort any later image taken by the same camera.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
]

#### 1. Undistort the image


The next step would be to actually undistort the images. An example of the original distorted image and the undistorted one, after applying `cv2.undistort()` function is shown below
![alt text][image1]
![alt text][image2]

#### 2. Apply threshold to detect lines start points

I used a combination of the s-channel of the HLS color space and the x-gradient thresholds to generate a binary image. I then both filter to obtain the binary image. In the case of the s-channel threshold an high low threshold was applied to filter out the shadows in the image. Here's an example of my output for this step!

![alt text][image3]

#### 3.Perspective transform

The next step would be to perform a perspective transform and convert the image to bird's eye view. As lane would appear parallel in such view and could be detected in an easier way.the `transform_to_birdseye_viewfunction` takes as inputs an image  and returns the warped image along with the inverse matrix. The points for the source and destination points were hardcoded in the following manner:


This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 1200, 710      | 1200, 720        | 
| 0, 710      | 0, 720      |
| 546, 460     | 0, 0      |
| 732, 460      | 1200, 0        |

An example of the warped binary image is shown below

![alt text][image4]

#### 4. Lane Boundary finding

In order to find the lane pixels and hence, the lane markings the following steps where applied:
1. Compute a histogram of the activation at each x-point by summing over the activations in the y-axis (since lanes are expected to be mostly vertical lines) and use that as a start point for the left and right lanes.

2. For image and the 1st frame in a video we take those start points and use windows of arbitrary sizes to search for the activations within each window, sizes and number of windows are defined on a trial and error basis. After we detect all the possible pixels we fit a second degree polynomial for both lanes and use that to deduce the points consituting the boundaries.

![alt text][image5]

3. In case of videos we ideally have preceding frames where there were valid lanes detected, in order not to start from scratch we check whether in the previous step a polynomial fitting was possible if so we make use of that information and and search around the detected line.

![alt text][image6]

if the previous frame was deemed invalid we revert back to our blind search methodology.

#### 5. Radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is then after converting the pixel points to the real world values. The US standards for lane width and length were used for such conversion. The Vehicle offset is computed as the difference between the middle of image and delta of the two lane lines, all in real measurements domain.

Sanity checks are also performed to assert that the left and right lane slopes and curvatures are roughly the same.

#### 6. Projection on original image.

Finally, The lane markings as well as the radius of curvature and vehicle offset position are reprojected upon the original(undistorted image). An example of this could be found in the below image.

![alt text][image7]

---

### Pipeline (video)

The pipeline for the video processing is similar to the image one. However, I additionally implemented a line class to keep track of the information of the detected lines in the previous frames for two main proposes:
1- Look ahead filter: Decide which lane finding technique to use based on whether the previously detected lines were deemed valid or not.
2- Apply exponential smoothing by weighting the previously detected lanes and the current one.

Here's a [link to my video result](./out_project_video.mp4)

---

### Discussion: What could be Improved?

#### Challenges:
The main challenge was to decide on which combination of thresholds to use and also to decide on how to make use of previously the detected lanes in case of video processing.

#### Where is the algorithm expected to fail:
I tested the algorithm on the challenge videos in faces difficulty detecting the lanes in case of extreme shadows or lighting conditions and rapid changes in the lane curvature.

#### Possible Improvements:
More color channels could be used for example the HSV color space could be tried to see if it works best with the shadows.
A reset method could be implemented to reset the line parameters i.e previously detected poly and lanes after failure of lane detection for a number of frames. Sanity checks could be used to decide whether to go for a blind search for lane detection or to build upon previous frame information. Also a learning object detection methodology could be used for example based on (YOLO or R-CNN) instead of using a method with so many heuristics and thus it could be expected that such method would better generalize to more scenarios.
