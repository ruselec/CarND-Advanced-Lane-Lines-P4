# CarND-Advanced-Lane-Lines-P4

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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./examples/test_undistort.png "Road Transformed"
[image3]: ./examples/binary_combo_example1.png "Binary Example"
[image4]: ./examples/binary_combo_example2.png "Binary Example"
[image5]: ./examples/binary_combo_example3.png "Binary Example"
[image6]: ./examples/binary_combo_example4.png "Binary Example"
[image7]: ./examples/binary_combo_example5.png "Binary Example"
[image8]: ./examples/binary_combo_example6.png "Binary Example"
[image9]: ./examples/binary_combo_example7.png "Binary Example"
[image10]: ./examples/binary_combo_example8.png "Binary Example"
[image11]: ./examples/warped_straight_lines.png "Warp Example"
[image12]: ./examples/color_fit_lines.png "Fit Visual"
[image13]: ./examples/formula.png "Formula"
[image14]: ./examples/example_output.png "Output"
[video1]: ./project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

---
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first and second code cells of the IPython notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the parameters from above, I applied the distortion correction on a raw image. The code for the distortion correction is contained in the third code cell of the IPython notebook.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

First, I converted the image to grayscale. Then, I applied yellow and white filter. For that I converted the image to HSV color scheme. I chose the upper and lower values for yellow and white colors. Then I combined masks for these colors and applied full mask on the grayscale. The code for the thresholded binary image is contained in the fourth code cell of the IPython notebook.  
Here are some results of the binary thresholds on the test images:

![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]
![alt text][image7]
![alt text][image8]
![alt text][image9]
![alt text][image10]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_view()`, which appears in the 5th code cell of the IPython notebook.  The `perspective_view()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```
    point_src = np.float32(
        [(200,720),
         (595, 450), 
         (685,450), 
         (1130,720)])
    point_dst = np.float32(
        [(280,720),
         (280,0),
         (1000, 0),
         (1000,720)])

```

Here is an example of a transformed image:

![alt text][image11]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for my identifing lane-line pixels includes a function called `window()`, which appears in the 8th code cell of the IPython notebook. To detect the lanes I calculated a histogram and used argmax to determine two peaks - one peak left of the midpoint and one peak right of the midpoint. Then I use sliding window method to find coordinates of pixels for left and right lines. After that I use np.polyfit to approximate each line by function of second order. Finally, I used a function 'gen_polygon', which appears in the 11th code cell of the IPython notebook and uses cv2.fillPoly to draw the lane on blank image. 

![alt text][image12]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for my calculation of the radius of curvature of the lane and the position of the vehicle includes functions called `calc_curvature()` and `calc_offset()`, which appears in the 12th code cell of the IPython notebook. The pixel values of the lane are scaled into meters using the scaling factors defined as follows:

```
ym_per_pix = 30.0/720 # meters per pixel in y dimension
xm_per_pix = 3.7/700 # meters per pixel in x dimension
```

Then I used this forumla to calculate the estimated radius of each lines:

![alt text][image13]

To calculate the position of the vehicle, I assume the camera is mounted at the center of the car and the deviation of the midpoint of the lane from the center of the image is the offset. 

```
middle = (left_fitx[-1] + right_fitx[-1])//2
veh_pos = image.shape[1]/2
offset = (veh_pos - middle)*xm_per_pix
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 13th code cell of the IPython notebook. To plot back down the lane area I used a function 'perspective_unwarp()', which defined in the 5th code cell of the IPython notebook, and a function`cv2.addWeighted()`. To output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position, I used a function 'show_info()', which defined in the 14th code cell of the IPython notebook. Here is an example of my result on a test image:

![alt text][image14]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline works well for the first video but fails on the second and third videos.
I've found that color filters are better than gradient methods for the first video. But for the second challenge video they are not working because of the part of lines has another colors than yellow and white. To make the algorithm more robust I think to combine these two group of methods. The third video has different brightness for frames so I think first use histogram equaliztion and then apply color filters to find lines. 
Another things that I would like to implement are:
* defining of the confidence for each line  
* use previous frame to define region of interest
* outlier rejection
* low-pass filter to smooth the lane detection over frames

