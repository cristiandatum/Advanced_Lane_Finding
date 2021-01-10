## Udacity - Self-Driving Car Engineer NanoDegree

## Advanced Lane Finding by Cristian Alberch - Jan 2021

### Overview

In this project, your goal is to write a software pipeline to identify the lane boundaries in a video.



![](project_video_preview.gif)

![Alt Text](C:\Git Projects\udacity_files\Project - Advanced Lane Finding\project-video-preview.gif)


When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle. Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

This is an project is will detect lane lines in video images using Python and OpenCV without the use of machine learning techniques and produce an output video showing the lane area. 

The pipeline used to produce the output video:

#### 1. Camera Calibration: 
A set of sample checkerboard images are used to calculate the necessary camera calibration to produce a distortion free image.

#### 2. Horizontal Image Crop: 
The road image of interest is cropped horizontally to display the road area only.

#### 3. Sobel Operators: 
Apply Sobel operators to transform the image from color to black & white with emphasis on road lane features. The Sobel operators applied include magnitude, direction (x & y axis), and gradient. The image is then processed to identify lane markings according to Hue-Saturation-Level.

#### 4. Region of Interest: 
The filtered image is cropped to identify only the road-only region of interest.

#### 5. Perspective Transform: 
The image is warped to show a birds eye view.

#### 6. Perspective Lane Selection: 
The birds eye view is subsequently cropped to reduce the width of the image for lane identificaiton.

#### 7. Polynomial Regression Estimate: 
The pixels in the lane identification are used to produce a best-fit 2nd order polynomial regression curve modelling the lane curve.

#### 8. Regression Estimate Value Checks: 
The left and right lane curves are verified for congruence, considering that the distance between the lanes is constant. The values are further averaged over the previous 10 image frames to reduce jaggedness and ensure the predicted path is consistent with the previous images.

#### 9. Radius of Curvature: 
The curvature radius is calculated according to the estimated lane curve and 2nd order polynomial coefficients.

#### 10. Image Overlay: 
An overlay fill is superimposed to the original image based on the lane curves identified.

#### 11. Text Overlay: 
The curvature radius value is overlayed to the image output.



Creating a great writeup:

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


### Camera Calibration
Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

### Apply Inveserse Perspective Transform

Apply inverse perspective transform on undistorted image.
Inverse Perspective Mapping (IPM), we want to produce a birds-eye view image of the scene from the front-facing image plane.

The image obtained by the car camera contains a large non-road area, such as sky, trees on roadside, etc. Globally processing of the image will increase the computational complexity and reduce the real-time capability.

Furthermore, the invalid area will interfere the lane information and affect the detection accuracy. Therefore, the valid region within the image should be selected to eliminate the invalid information.

Ref: https://medium.com/ai-in-plain-english/inverse-perspective-transformation-b62b5eedb44a

### radius
https://news.osu.edu/slow-down----those-lines-on-the-road-are-longer-than-you-think/

Each dashed line measures 10 feet, and the empty spaces in-between measure 30 feet. So every time a car passes a new dashed line, the car has traveled 40 feet.

In the picutre below, the distance from the camera to the end of lane detection is approximately 120feet long, and the width of 

Real world:
length: 120ft
width: 12ft

Pixels: 240pixels
width: 685-595 = 90pixels

This needs to be taken to account calculate the radius.




Discussion
Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?


    Images from .mp4 video are individually distilled and fed into an image processing pipeline.
    Conversion to black & white to facilitate edge detection.
    Canny processing of image in color to detect image edges.
    Gaussian blur to smooth the edges.
    Region of interest limits the area of edges detected in image.



Results & Discussion

    Canny algorithm: The image processing was done fully in greyscale prior to applying Canny algorithm. This yielded better results than when directly applying from colour.

    Gaussian blur: A smaller kernel size yielded much better results, probably due to processing in color.

    Region of interest vertices are static and were selected based on the expected vehicle visibility of lanes. As the vertices are fixed, this severly limits the range of lane detection.

    Applying the Hough Transform in the region of interest resulted in satisfactory identification of lines. This however, means that dashed lines were identified as individual lines.

    Hough Transform applied

    In order to identify dashed lines as single lines, line fitting was applied by calculating the mean among the individual lines from the Hough transform. The dataset obtained from the individual lines was normalized with a standard deviation of 1.0 and the mean obtained. This ensures removing outliers and accurate average representation/

    Hough Transform applied

Further Work

Below listed are limitations of the pipeline and algorithms used together with possible solutions to explore:

    The pipeline assumes a linear model for lane detection. This will result in incorrect results when the lane is curved. Polylinear regression models would improve this.

    The pipeline uses a region of interest in identification of lanes to simplify the image processing. This will result in an incorrect result when the car steers sideways from the region of interest, or when the road significantly changes its vanishing point, in hills for example.

    The pipeline would incorrectly detect as lanes, markings or objects near the lanes such as crash barriers, or fences. Machine learning algorithms would improve identification of objects.

    Vehicles positioned at the sides or close to the front of the car could be incorrectly classified as lane markings. Machine learning algorithms would improve identification of objects.

