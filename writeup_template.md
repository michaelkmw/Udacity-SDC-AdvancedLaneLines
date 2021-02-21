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
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

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

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
