## Writeup


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

[image1]: ./output_images/Undistorted%20Image.png "Undistorted Image"
[image2]: ./output_images/Undistorted%20and%20Warped%20Image.png "Undistorted and Warped"
[image3]: ./output_images/calibration_plot.png "calibration_plot"
[image4]: ./output_images/Undistorted.png "Undistorted"
[image5]: ./output_images/Sobel%20X.png "Sobel X"
[image6]: ./output_images/Sobel%20Y.png "Sobel Y"
[image7]: ./output_images/Gradient%20Magnitude.png "Gradient Magnitude"
[image8]: ./output_images/Gradient%20Direction.png "Gradient Direction"
[image9]: ./output_images/Color%20Thresholds.png "Color Thresholds"
[image10]: ./output_images/Multi%20Thresholds.png "Multi Thresholds"
[image11]: ./output_images/Region%20Masking.png "Region Masking"
[image12]: ./output_images/Perspective%20Transform.png "Perspective Transform"
[image13]: ./output_images/Sliding%20Windows.png "Sliding Windows"
[image14]: ./output_images/Shaded%20Lanes.png "Shaded Lanes"
[image15]: ./output_images/Lane%20Mapping.png "Lane Mapping"
[video1]: ./project_video.mp4 "Video"
[video2]: ./project_video_output.mp4 "Video Output"
[video3]: ./challenge_video.mp4 "Challenge Video"
[video4]: ./challenge_video_output.mp4 "Challenge Video Output"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in cell #1 through #5 of the file called `Code.ipynb`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

Then I applied perspective transformation and draw corners on the chess board:

![alt text][image2]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

![alt text][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination (defined in `multi_thresholds` function in cell #7 in `Code.ipynb`) of:
 - sobel X and Y thresholds (defined in `abs_sobel_thresh` function in cell #7 in `Code.ipynb`)
 - gradient magnitude threshold (defined in `gradient_magnitude` function in cell #7 in `Code.ipynb`)
 - gradient direction threshold (defined in `gradient_direction` function in cell #7 in `Code.ipynb`)
 - color threshold (defined in `color_thresholds` function in cell #7 in `Code.ipynb`) 
to generate a binary image .  
Here's an example of my output for this step.  

![alt text][image10]

I also used a region masking (defined in `region_of_interest` function in cell #7 in `Code.ipynb`):

![alt text][image11]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_perspective`, which appears in cell #7 in the file `Code.ipynb`.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.array([[(width*0.4, height*0.65),
                 (width*0.6, height*0.65),
                 (width, height),
                 (0, height)]], 
                 dtype=np.float32)
dst = np.array([[0,0], 
                [img.shape[1], 0], 
                [img.shape[1], img.shape[0]],
                [0, img.shape[0]]],
                dtype = 'float32')
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 512, 468      | 0, 0        | 
| 768, 468      | 1280, 0      |
| 1280, 720     | 1280, 720      |
| 0, 720      | 0, 720        |

![alt text][image12]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used `sliding_windows` function and `shaded_lanes` function in cell #7 in `Code.ipynb` to fit 2nd order polynomial. Here's the result:

![alt text][image14]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I used `roc_in_meters` function and `offset` function in cell #7 in `Code.ipynb` to calculate the radius of curvature of lane and position of vehicle with respect to center:

| test image        | Left Radius of Curvature   | Left Radius of Curvature | Offset from Lane Center   |
|:-------------:|:-------------:|:-------------:| :-------------:|  
| test1         | 167.61 meters                  | 728.85 meters            | -0.12 meters       |
| test2         | 198.52 meters                  | 718.29 meters            | -0.24 meters       |
| test3         | 2682330587 meters              | 720.00 meters            | -0.17 meters       |
| test4         | 167.68 meters                  | 733.47 meters            | -0.22 meters       |
| test5         | 296274414 meters               | 714.48 meters            | -0.08 meters       |
| test6         | 200.19 meters                  | 742.25 meters            | -0.32 meters       |
| straight_lines1      | 174.47 meters           | 695.07 meters            | -0.07 meters       |
| straight_lines2      | 3250028050 meters       | 694.33 meters            | -0.07 meters       |


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The lane mapping (defined in `lane_mapping` function in cell #7 in `Code.ipynb`) is:

![alt text][image15]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to project video result](./project_video_output.mp4). It can also be viewed on [this youtube link](https://youtu.be/dR4aHXyCI9U)

Here's a [link to challenge video result](./challenge_video_output.mp4). It can also be viewed on [this youtube link](https://youtu.be/WBozdwYHUY4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

This is a very exiting project since it is to improve the performance of lane finding in project 1 by implementing steps like camera calibration , finding the gradient magnitude and direction, color thresholding and perspective transformatoin, etc. It is performing well and robust on project_video.mp4. But for challenge_video.mp4 it is not performing very well on conditions like having an extra dark line on the road which is not the lane line, shaded areas like under the bridge and when a car very close to the lane line passed by that is generating a dark line that is more obvious than the lane line. The pipeline could be made more robust by normalizing image to account for illumination variations.
