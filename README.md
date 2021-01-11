## Udacity - Self-Driving Car Engineer NanoDegree

## Advanced Lane Finding by Cristian Alberch - Jan 2021

### Overview

When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle. 

This project is will detect lane lines in video images using Python and OpenCV without the use of machine learning techniques and produce an output video showing the lane area. 

![Lane Detection in Python](project_video_preview.gif)

The pipeline used to produce the output video is summarised. The full code for each item number below is included in the Jupyter Notebook using the same reference.

#### 1. Camera Calibration: 
The output image from the camera has some distortion due to the camera lens. In order to accurately measure distances in a picture, it is necessary to remove the camera distortion.

A set of sample checkerboard images are used to calculate the correct image object points that are used by CV2 calibrateCamera function to produce a distortion free image. 

![](report/checkerboard.jpg)  |  ![](report/checkerboard_undistorted.jpg)

Although it is less obvious, the below image is also distorted:

![](report/original_image.jpg)

Applying the camera calibration using function,
```
def cal_undistort(img, objpoints, imgpoints):
    gray = cv2.cvtColor(img,cv2.COLOR_BGR2GRAY)
    ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, gray.shape[::-1], None, None)
    undist = cv2.undistort(img, mtx, dist, None, mtx)
    return undist
```

yields the same image undistorted.

![](report/original_image_undistorted.jpg)


#### 2. Horizontal Image Crop: 
The road image of interest is cropped horizontally to display the road area only.

![](report/horizontal_crop.jpg)

The red lines indicate the region of interest for road markings.

The end of the road markings to identify are shown as the two red dots. In a 2D bird's eye view image the distance between the two blue dots and the two red dots is the same.



#### 3. Sobel Operators: 

Sobel operators are applied to transform the image from color to black & white with emphasis on road lane features. The Sobel operators applied include magnitude, direction (x & y axis), and gradient. The image is then processed to identify lane markings according to Hue-Saturation-Level.

```
def threshold_binary(img):
    gradx = abs_sobel_thresh(img, orient='x', sobel_kernel=5, thresh=(30, 180))
    grady = abs_sobel_thresh(img, orient='y', sobel_kernel=5, thresh=(50, 160))
    mag_binary = mag_thresh(img, sobel_kernel=7, mag_thresh=(50, 160))
    dir_binary = dir_threshold(img, sobel_kernel=15, thresh=(0.8, 1.3))
...
    combined_thresh[((gradx == 1) & (grady == 1)) | ((mag_binary == 1) & (dir_binary == 1))] = 1
    combined_hls=hls_color(img, s_thresh=(100, 255), sx_thresh=(40, 80))/255
 ...
    combined_thresh_hls=np.clip(combined_thresh_hls, a_min = 0, a_max = 1)        
    return combined_thresh_hls
```
Applying the threshold function, results in an improved identification of lane markings.

![](report/fuzzy_lines.jpg)


#### 4. Region of Interest: 
The filtered image is cropped to select only the region of interest as indicated in the red lines.

![](report/fuzzy_lines_threshold.jpg)


#### 5. Perspective Transform: 

The image containing the region of interes is transformed to a 2D bird's eye view. As per Para 2, a perspective transform is applied to the image using the start points and end points (red dots and red dots in Para. 2) as references. 

```
def warp(img_roi):
    src = np.float32([[0, 240], [1280, 240], [0, 0], [1280, 0]])
    dst = np.float32([[595, 240], [685, 240], [0, 0], [1280, 0]])
    M = cv2.getPerspectiveTransform(src, dst)
    img_warped = cv2.warpPerspective(img_roi, M, (1280, 240))
    return img_warped
```

![](report/thresholds_warped.jpg)

#### 6. Perspective Lane Selection: 
The birds eye view is subsequently cropped to reduce the size of the image for lane identification.

#### 7. Polynomial Regression Estimate: 
The pixels in the lane identification are passed to find_lane_pixels function which provides an estimate of the pixels located on the left and right side. This is done by first passing the matrix image to a histogram calculation. The histogram peaks determine where the most pixels are located, and thereby where the lane markings are most likely to occur. The calculation is carried out iteratively through a number of windows.

```
def find_lane_pixels(binary_warped):
    # Take a histogram of the bottom half of the image
    histogram = np.sum(binary_warped[binary_warped.shape[0]//2:,:], axis=0)
    # Create an output image to draw on and visualize the result
    img_out = np.dstack((binary_warped, binary_warped, binary_warped))
    # Find the peak of the left and right halves of the histogram
    # These will be the starting point for the left and right lines
    midpoint = np.int(histogram.shape[0]//2)
    leftx_base = np.argmax(histogram[:midpoint])
    rightx_base = np.argmax(histogram[midpoint:]) + midpoint

    # HYPERPARAMETERS
    # Choose the number of sliding windows
    nwindows = 6
    # Set the width of the windows +/- margin
    margin = 20
    # Set minimum number of pixels found to recenter window
    minpix = 20

    ...

    return leftx, lefty, rightx, righty, img_out
```

A best-fit 2nd order polynomial regression is carried out on the pixels representing the left and right lanes respectively. curve modelling the lane curve.

```
def fit_polynomial(binary_warped):
    left_fit = np.polyfit(lefty, leftx, 2)
    right_fit = np.polyfit(righty, rightx, 2)

    ...

    return img_out, left_fit, right_fit, left_fitx, right_fitx, ploty
```

The function returns left_fit list containing [A,B,C] for 2nd degree polynomial function.

![](https://latex.codecogs.com/svg.latex?\Large&space;f_{right}(y)=A_{right}y^2+B_{right}y+C_{right})

This can then be plotted to show the pixels image, windows showing search area and polynomial curves.

![](report/best_fit.jpg)


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

