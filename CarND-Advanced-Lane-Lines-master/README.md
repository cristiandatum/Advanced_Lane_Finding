## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
![Lanes Image](./examples/example_output.jpg)

In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  

Creating a great writeup:
---
A great writeup should include the rubric points as well as your description of how you addressed each point.  You should include a detailed description of the code used in each step (with line-number references and code snippets where necessary), and links to other supporting documents or external references.  You should include images in your writeup to demonstrate how your code works with examples.  

All that said, please be concise!  We're not looking for you to write a book here, just a brief description of how you passed each rubric point, and references to the relevant code :). 

You're not required to use markdown for your writeup.  If you use another method please just submit a pdf of your writeup.

The Project
---



<figure class="video_container">
  <iframe src="https://youtu.be/xBTPrPMG40Q" frameborder="0" allowfullscreen="true"> </iframe>
</figure>


The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The images for camera calibration are stored in the folder called `camera_cal`.  The images in `test_images` are for testing your pipeline on single frames.  If you want to extract more test images from the videos, you can simply use an image writing method like `cv2.imwrite()`, i.e., you can read the video in frame by frame as usual, and for frames you want to save for later you can write to an image file.  

To help the reviewer examine your work, please save examples of the output from each stage of your pipeline in the folder called `output_images`, and include a description in your writeup for the project of what each image shows.    The video called `project_video.mp4` is the video your pipeline should work well on.  

The `challenge_video.mp4` video is an extra (and optional) challenge for you if you want to test your pipeline under somewhat trickier conditions.  The `harder_challenge.mp4` video is another optional challenge and is brutal!

If you're feeling ambitious (again, totally optional though), don't stop there!  We encourage you to go out and take video of your own, calibrate your camera and show us how you would implement this project from scratch!

## How to write a README
A well written README file can enhance your project and portfolio.  Develop your abilities to create professional README files by completing [this free course](https://www.udacity.com/course/writing-readmes--ud777).

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

### Pipeline test images consists of:


#### Apply Sobel kernes filtered
- x-direction
- y-direction
- magniute threshold


## Description
Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.


Discussion
Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

Finding Lane Lines on the Road
Submitted by Cristian Alberch as part of Udacity Self Driving Car Engineering Nanodegree - December 2020
Overview

When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle. Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

This is an project is will detect lane lines in video images in a rudimentary manner using Python and OpenCV. The process followed are discussed together with improvements needed.
Process

The pipeline consists of the following steps: The video image processing pipeline consists of:

    Images from .mp4 video are individually distilled and fed into an image processing pipeline.
    Conversion to black & white to facilitate edge detection.
    Canny processing of image in color to detect image edges.
    Gaussian blur to smooth the edges.
    Region of interest limits the area of edges detected in image.
    Hough transform weas applied to detect lines within the image. This resulted in multiple lines representing a single curve.
    Curve fitting was applied by:
        Determine which lines generated from Hough algorithm are located in the left or right lane depending on the slope of the line.
        Determine the median of the slopes.
        Apply a line with the median of the slope starting from the bottom of the frame until the vanishing point.
    The curves are overlayed to the image.

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

