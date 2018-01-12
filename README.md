# Advanced Lane Finding Project Report

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
[calibration]: ./figs/chessboard.png "Camera Calibration"
[undist]: ./figs/test_images_undist.png "undistorted image"
[transform]: ./figs/perspective_transform_example.png
[color_binary]: ./figs/YUV_images.png
[lane_curve_center]: ./figs/lane_curve_center.png
[lane_find]: ./figs/lane_find.png

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for Camera Calibration is contained under the "Calibration Header". After importing the camera calibration images, each image was converted to grayscale to calculate the chessboard corners using the cv2.findChessboardCorners() function. The corner points were then used for the  cv2.calibrateCamera() function to generate the distortion matrix and coefficients. The image below is a comparison of a distorted and undistorted chessboard.

![Camera Calibration][calibration]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Images are undistorted using the distortion matrix and coefficients calculated from the camera calibration process.

![alt text][undist]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I tested various color transforms to find the right final binary image. To perform the color transform I used the cv2.cvt2Color with the appropriate colorspace code. To do the binary thresholding, I defined a low and high threshold for each channel and then set any pixel within the thresholds. For color channels with very high contrast for the line (such as the B channel of LAB), I did a mean + standard deviation method to calculate the threshold. To experiment and test the efficacy of each transform, I plotted each test images color transforms with the 3 channels and each channels binary result so that I could experiment with the thresholding to find the right channel to identify lines. Below is an example for the YUV colorspace.

![Transforms and binary][color_binary]

For the final binary image, I ended up using the B channel of LAB and the B channel of RGB since they identified the yellow and white line respectively. 

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for the perspective_transform function and src/dst calculation is in the "Perspective Transform" section. 

```python
# [bottom left,
# top left, 
# top right, 
# bottom right]
M = test_image_size[0]
N = test_image_size[1]
src = np.float32(
    [[273, 680],
    [595, 450],
    [686, 450],
    [1050,680]])
x_offset = 280
dst = np.float32(
    [[x_offset, N],
    [x_offset, 0],
    [M-x_offset,0],
    [M-x_offset, N]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 273, 680      | 280,  720     | 
| 595, 450      | 1000, 0       |
| 686, 450      | 1000, 0       |
| 1050, 680     | 280,  720     |

Using the two provided straight lane images, the lane lines were lined up with two red lines drawn based on the dst coordinates.

![alt text][transform]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for the line finding is located in the "Lane Finding Algorithm" section. I used the histogram and sliding window approach. The contrast for the final binary images was good enough to use this approach throughout the video without using smoothing or skipping the sliding windows.

![alt text][lane_find]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code to calculate the Curvature and Center offset are in the "Curvature and Center Offset" section. To calculate the curvature, I used the provided derived equation, which used the first and second order coefficient of the line polynomial and 720 for the y value since that represented the bottom of the image, which was where the car was located. To measure the meter per pixels for the X dimension, I eyeballed the pixel distances between the left and right line using the top down images. For the y-dimension, I used the distance between two short dashes on the lane line.

To calculate the center offset, I used the position of the left and right line at the bottom of the image to get left_offset and right_offset. Using their pixel distance from the center of the image, I took the difference of those two values and divided by two to get the center offset, which was then multiplied by the X dimension meters per pixel. The min value between left_offset and right_offset determined which side the car was on.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt_text][lane_curve_center]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

I created a pipeline function that combines all the previously mentioned techniques. For the input image, it goes through distortion, perspective transform, color transform, binary thresholding, lane-finding algorithm, reverse perspective warp (of the generated line polynomial), and curvature/center offset calculation. The pipeline function is used to feed the VideoFlipClip.fl_image() function, which takes a function and transforms each video frame to create a new video.

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

When I tested my process with the challenge video, I found that the lighter gray road created less contrast with almost every color channel. Only the B channel of LAB colorspace had strong contrast. Using tighter thresholds didn't work either because of the variation of the car moving under the bridge and part of the lane turning from black to gray. Using some sort of adaptive threshold, perhaps using histogram information, could solve the problem.

The algorithm could also be improved using smoothing and confidence calculation. If some sort of confidence mechanism were to be used, the algorithm could decide between using the histogram/sliding window and the within the margins method. An idea I had to calculate confidence was to count the number of lane pixels within the margin and compare it to the total number of binary pixels on the respective half of the image. Also with the confidence, the smoothing could be applied so that only the last X number of curves were used to smooth the line. 