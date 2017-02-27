##Writeup Report

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

[image1]: output_images/calibrated_input.jpg "Original"
[image2]: output_images/calibrated_output.jpg "Undistorted"
[image3]: output_images/test3.jpg.jpg "Test Image"
[image4]: output_images/test3.jpg_undistorted.jpg "Undistorted Test Image"
[image5]: output_images/test3.jpg_thresholded_binary.jpg "Thresholded Test Image"
[image6]: output_images/test3.jpg_transformed.jpg "Transformed Image"
[image7]: output_images/test3.jpg_output.jpg "Final output image"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `advanced_lane_lines.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image provided in the repository (note these are 9x6 chessboard images, unlike the 8x6 images used in the lesson).  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 


Original :
![alt text][image1]

Corrected :
![alt text][image2]

###Pipeline (single images)

####1. Distortion correction.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image3]

Distortion correction was calculated via camera calibration. The camera matrix (`mtx`) and distortion coefficients (`dist`) are fed as arguments to the `cv2.undistort` function (see second code cell in `advanced_lane_lines.ipynb`). An example of a distortion corrected image to the above test images is included inline here :

![alt text][image4]

####2. Threshold.
I used a combination of color and gradient thresholds to generate a thresholded binary image. The gradient detection was conducted using Sobel filters. I combined various aspects of gradient measurements (x, y, magnitude, direction and HLS) to isolate lane-line pixels. The combined binary image was returned by selecting pixels where both the x and y gradients meet the threshold criteria,OR the gradient magnitude and direction are both within threshold values,OR the HLS S channel is within the threshold values (see third code cell in `advanced_lane_lines.ipynb`). An output sample of the binary thresholded image on the test image looks like follows :

![alt text][image5]

####3. Perspective Transform ("birds-eye view")

The code for my perspective transform includes a function called `cv2.warpPerspective`, which appears in the fourth code cell in `advanced_lane_lines.ipynb`. The `perspective_transform` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

```

I verified that my perspective transform was working as expected by comparing the test image with its warped counterpart to verify that the lines appear parallel in the warped image. Here is the warped image :

![alt text][image6]

####4. Detect lane line pixels and fit thier positions with a polynomial
Methods have been used to identify lane line pixels in the rectified binary image. The left and right line have been identified and fit with a curved functional form (e.g., spine or polynomial). The first step for finding the lines was to get a histogram of the frequency of pixels in the lower half of the image. This is used for finding the candidate x,y points where we will assume the center of the lines will be located. After this, a sliding window search is performed in the left and right halves of the image. This is done starting from the lower position, where the line is initially detected from the histogram, then by progressively moving upwards and repositioning the window center towards the mean of the x-positions for all the non-zero pixels within the current window. The positions of all pixels for each window are all added up into a single array. From all the positions of these pixels,which hopefully will describe approximately the positions of the lines, we can perform a polynomial fitting using the `np.polyfit()` function, which takes the `y` and `x` coordinates of a set of points and returns an array
 of coefficients for a polynomial fit of the requested degree. The code for the sliding window detection and polynomial regression is contained in the `detect_lane` functionin the fifth code cell in `advanced_lane_lines.ipynb`.

####6. Determine the curvature of the lane and vehicle position with respect to center.
The radius can be found by using the formula as given in http://www.intmath.com/applications-differentiation/8-radius-curvature.php. As  given, this method will return the radius in pixel units of the curve. To convert this into meters we first convert these parameters to the physical measurements by multiplying both x and y measurements by the ratio of meters/pixel we found by measuring the lane lines and taking into account the line width and an approximated value of 30m for the length
of the considered area. I implemented this step as reflected in the sixth code cell in `advanced_lane_lines.ipynb`.

####7. Plotting the lane back into the image
The code reflected in the seventh code cell in `advanced_lane_lines.ipynb` under the `pipeline` function shows how the lane is plotted into the original images. This is achieved after performing the inverse perspective transformation and overlaying it on the original image.
The result looks like follows:
![alt text][image7]

---

###Pipeline (video)

####1. Provide a link to your final video output.  My pipeline performed reasonably well on the entire project video. There are some jitters but but no catastrophic failures that would cause the car to drive off the road!.

Here's a [link to my video result](https://www.youtube.com/watch?v=GYC1fYdH578)

####2. Challenge video solution link. My pipeline on the challenge video however shows the plots with a lot of jitters especially when navigating with lane curvature.

Here's a [link to the challenge video result](https://www.youtube.com/watch?v=qiRsMu2fz9M) 

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline implemented combining various aspects of gradient measurements (x, y, magnitude, direction, HLS) to isolate lane-line pixels worked well for the project video as can be seen in the Pipeline video step #1. However, during lane curves where there were shadows there is some jitter effect seen. I ran the implemention against the challenge video and as can be seen in the  results video (Step #1 from Pipeline (video) section) there is some vigourous jitters observed. Perhaps, the gradient thresholding values that were empirically determined for the video solution needs to be tweaked for the challenge video.

