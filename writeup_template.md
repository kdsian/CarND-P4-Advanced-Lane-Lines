# Advanced Lane Finding Project

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
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## 1. [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

## 2. Writeup / README

#### 1) Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

## 3. Camera Calibration

### 1) Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

<center><img src="./output_images/cal_undist_fig.png"></center>


## 4. Pipeline (single images)

### 1) Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

<Original>
<center><img src="./test_images/test6.jpg"></center>

<Undistored>
<center><img src="./output_images/undist_test6.jpg"></center>

### 2) Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at ln[7]~ln[9] in `Advanced-Lane-Lines.ipython`).  Here's an example of my output for this step.  

<center><img src="./output_images/binary_test6.jpg"></center>

### 3) Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `unwarp()`, which appears in ln[15] in the file `Advanced-Lane-Lines.ipython`.  The `unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([(575,464),
                  (707,464), 
                  (258,682), 
                  (1049,682)])
dst = np.float32([(450,0),
                  (img_size[0]-450,0),
                  (450,img_size[1]),
                  (img_size[0]-450,img_size[1])])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 464      | 450, 0        | 
| 258, 682      | 450, 720      |
| 1049, 682     | 1280, 720      |
| 707, 464      | 1280, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

<center><img src="./output_images/unwarped_test6.jpg"></center>

### 4) Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

ln[32]의 81~93 line에서 볼 수 있듯이 line을 2차 함수로 np.polyfit 함수를 이용하여 근사화 했습니다.

아래 그림에서와 같이 A,B,C 상수가 left_fit[0], left_fit[1], left_fit[2]에 해당합니다.

<center><img src="./examples/color_fit_lines.jpg"></center>



### 5) Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 98 through 110 in my code ln [23] in  `Advanced-Lane-Lines.ipython`

### 6) Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines ln[23]  in my code in `Advanced-Lane-Lines.ipython` in the function `line_detect()`.  Here is an example of my result on a test image:

<center><img src="./output_images/result_test6.jpg"></center>

---

## 5.Pipeline (video)

### 1) Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

## 6.Discussion

### 1) Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

* treshold에서 channel 크기에 대한 조건 만을 적용했을 때가 sobel 알고리즘 결과를 같이 적용했을 때 보다 결과가 좋게 나타남.
* 기존 frame에서 찾은 line의 정보를 이용할 수 없음
* advanced_challenge 영상에 대해 수행할 경우 line을 못찾는 경우가 다수 발생 