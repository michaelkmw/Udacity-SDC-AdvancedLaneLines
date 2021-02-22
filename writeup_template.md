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

[image1]: ./output_images/undist_calibration1.jpg "Undistorted Calibration Image"
[image2]: ./output_images/undist_signs_vehicles_xygrad.png "Undistorted Test Image"
[image3]: ./output_images/combined_test5.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines2.jpg "Warp Example"
[image5]: ./output_images/detected_test5.jpg "Fit Visual"
[image6]: ./output_images/final_test5.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell under `Camera Calibration` section of the IPython notebook located in "./P2.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. Thresholding functions are defined in `Gradient Thresholding` and `Color Thresholding` section in `P2.ipynb`. The specific implementation can be found at lines #23 through #65 in the `Test Image Pipeline` section of the notebook. Through experimentation with test images, it is found that the following combination of gradient and color thresholding methods can identify the lane markings clearly:
* Gradient thresholding with Sobel x gradient
* Color thresholding with S-channel in HLS color space and R-channel in RGB color space

It is found that a combination of S-channel and R-channel thresholding allows light-color lane marking to be highlighted while eliminating the noises caused by shadow.

The following image shows an example of the pipeline's output for this step. Red color is contributed by color thresholding, whereas Green color is contributed by gradient thresholding.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in `Perspective Transform` section in `P2.ipynb`. The function is called by the pipeline at lines #67 through #69 in the `Test Image Pipeline`. The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
# Four source coordinates
src = np.float32(
    [[203, 720],
     [581, 460],
     [702, 460],
     [1099, 720]])
# Four desired coordinates
dst = np.float32(
    [[350, 720],
     [350, 0],
     [950, 0],
     [950, 720]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 581, 460      | 350, 0        |
| 203, 720      | 350, 720      |
| 1099, 720     | 950, 720      |
| 702, 460      | 950, 0        |

I verified that my perspective transform was working as expected by testing the function on straight line test images to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Once an image has been warped by perspective transform, a sliding window search method is then applied to identify the lane marking pixels on the picture. The code for the search function, `sliding search`, can be found at lines #113 through #220 in `Lane Finding` section of the notebook. The function is called by the pipeline at lines #71 through #72 in the `Test Image Pipeline`. Once the lane marking pixels have been identified, `fit_poly` function at lines #1 through #111 in `Lane Finding` section is called to fit a second order polynomial over the identified lane markings.

The following image shows an example of the pipeline's output for this step. Red color highlights the identified left lane line, Blue color highlights the identified right lane line, Green color shows the sliding window, Yellow color shows the fitted polynomial over the identified lane lines.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature and the position of the vehicle with respect to center are calculated using the fitted polynomial over the lane line pixels. The code for the search function, `measure_curvature_real`, can be found in `Calculate Curvature` section of the notebook. The function is called by the pipeline at lines #49 through #51 in the `fit poly` function in `Lane Finding` section.

The radius of curvature and lane position are computed at the bottom of the image (ie. pixel 720). The radius of curvature is computed using the y-pixel position (720) and the coefficients of the fitted polynomials. The radius of curvature of left and right lane lines are then averaged at line #120 in `Test Image Pipeline` section.

The lane line position with respect to the vehicle center is computed by subtracting x-pixel position of the lane from the image center (ie. it is assumed that vehicle center aligns with image center). X-pixel position of the lane can be computed using the fitted polynomial evaluated at bottom of the image. The left and right lane line position w.r.t. vehicle center are then added together to obtain the distance between vehicle center and lane center. The code can be found at lines #123 through #129 in `Test Image Pipeline` section.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The area between the fitted polynomials are drawn and inverse perspective transformed into the original image to illustrate the identified lane area. This step is implemented in lines #92 through #110 in my code in `Test Image Pipeline` section. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The pipeline described in the previous sections form the basis of the pipeline used to process the project video. The pipeline for video analysis can be found in function `process_image` in `Video Pipeline` section. This function is called by `Test on Video` section. The following are the changes of video pipeline over image pipeline.
* To improve efficiency, lane line pixels are identified through the fitted polynomials (`search_around_poly` function at lines #222 through #289 in `Lane Finding` section)
* Sliding window search is called when no polynomial has been fitted or when `fit_poly` function fails to fit polynomial for 25 frames consecutively (`search_lane` function at lines #291 through #308 in `Lane Finding` section)
* To reject noises in lane line pixel detection, fitted polynomials are averaged over the past 50 frames (lines #68 through #109 in `Lane Finding` section)

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The following issues came up during development of the pipeline:

1. Camera calibration can't process 9x6 chessboard
`cv2.findChessboardCorners` can't identify 9x6 corners in some calibration images. To circumvent the issue, 9x5 is chosen instead. This can potentially lead to inaccurate calibration as the undetected corners are located near the edge of the image, which is the place with highest amount of distortion.

2. Color thresholding using S-channel in HLS color space also highlights shadow in the background
This issue is noticeable when applying S-channel thresholding on `test5.jpg` in `test_images` folder. The shadow casted by the tree on the left side is picked up in S-channel. It is noticed that R-channel in RGB color space doesn't detect the shadow. Therefore, S-channel and R-channel are used in conjunction for color thresholding. S-channel in HSV color space has also been explored. However, it performs worse than S-channel in HLS color space.

3. Warped image from perspective transform loses resolution when trying to transform lane line in the distance
This is to be expected as the resolution of the lane line in the distance in the original image is lower. In order to enhance the resolution of the warped image, the y-pixel of the source points in the further end of the lane lines has to be increased.

4. Radius of curvature and vehicle position w.r.t. lane center fluctuates from frame to frame
Depending on the frame, lane line marking may significantly alter the polynomials fitted over the detected lane line pixels. To increase the stability of the results, radius of curvature and vehicle position are calculated using fitted polynomials that are averaged over 50 frames.

The following are potential issues with the current pipeline:

1. Failure to detect lane line if multiple high contrast line markings exist
The pipeline has been tested on the challenge video, and it is found that the sliding window method can possibly pick up the wrong lane line. This is probably caused by the histogram trying to locate the bottom of the lane line with `np.argmax` function. If there is a line with higher contrast than the lane line in the vincinity, this function would mistakenly identify that line as the lane line. Potential method to circumvent the issue is to narrow down the search area based on lane line distance from vehicle center and/or radius of curvature from previous frames

2. Failure to detect lane line if another vehicle merges into the lane
The gradient and color thresholding may mistakenly highlight the other vehicle as lane line pixel, which leads to incorrect curve fitting. Potential method to circumvent the issue is to compute histogram of activated pixels higher intervals (e.g. divide the image into several horizontal region and compute histogram of each region). The histogram of each neighbouring region can be cross-compared to check for consistency in horizontal location of activated pixels. If there is inconsistency with upper region, only the lower region is used for curve fitting

3. Failure to detect lane line if the vehicle is making sharp turn
As evident in the harder challenge video, the lane line could potentially be outside of the frame if the turn radius is small. This could cause the sanity check and lane detection function to fail to detect lane lines completely. Potential method to circumvent the issue is to analyze the historical trend of the lane line orientation and location with respect to the vehicle center and make the pipeline adaptive to the changing orientation and location of the lane line.