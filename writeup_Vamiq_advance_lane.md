## Advanced Lane Finding - Writeup
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

[image1]: ./output_images/undistort_chess.png "Undistorted"
[image2]: ./output_images/undistorted_road.png "Road Transformed"
[image3]: ./output_images/lanes_binary.png "Binary Example"
[image4]: ./output_images/perspective.png "Warp Example"
[image5]: ./output_images/perspective_lane "Warped histogram"
[image6]: ./output_images/lane_fit_polynomial.png "Fit Polynomial"
[image7]: ./output_images/lane_plot.png "Fit Lane"
[image8]: ./output_images/output_with_status.png "Unwarp and display"
[video1]: ./output_images/project_video_output.mp4 "Output Video"

---

### Writeup

#### 1. This writeup explains how each step in the [Rubric](https://review.udacity.com/#!/rubrics/571/view) was addressed in the code along with the images and videos where necessary. 

### Camera Calibration

#### 1. In this step, I computed the camera matrix and distortion coefficients. 

The code for this step is contained in the first code cell of the IPython notebook located in "./Advanced-Lane Finding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all 9 x 6 chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result in the second code cell: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I have selected straight_lines2.jpg loacated in "./test_images" and applied `cv2.undistort()` function and obtained this result in the third code cell. I have attached the original image and the indistorted image ontained by my code, in the following figure: 
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

In the fourth code cell of my Jupyter notebook, I have defined separate functions which apply different thresholds and output the resultant. They are as follows:
1. abs_sobel_thresh(): This function takes in an input image and based on the input orientation, and threshold, gives out the output
2. mag_thresh(): This function gives out a binary image with magnitude thresholding
3. dir_threshold(): This function carries out directional thresholding on a given input image.
4. Channnel_select(): This fucntion gives out a selected channel in the 'hls' or 'rgb' format.

I have done multiple iterations using these functions to arrive at a final result which brings out the lane lines with most clarity. Here's an example of my output for this step.  

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In the fifth code cell of my notebook, I have carried out a perspective transform of the undistorted image so that I am able to get a birds eye view as a result. For this purpose, I take as input from the earlier undistorted image (`undistorted`), and trace out the source points (`src`) which folow the lane lines on the image and define the destination (`dst`) points which I would like to see in the end as a bird's eye view. I chose the hardcode the source and destination points in the following manner:

```python
height = gray.shape[0]
width = gray.shape[1]
src = np.float32([[int(width*0.17), height],[int(width*0.475), int(height*0.61)],[int(width*0.525), int(height*0.61)],[int(width*0.87), height]])
dst = np.float32([[int(width*0.25),height],[int(width*0.25),0],[int(width*0.75),0],[int(width*0.75),height]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart respectively, to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the sixth coding cell I have expressed the first step in identifying the lanes. I map the histogram of the last row of pixels to identify the lane starting points and the output is shown in the following figure.

![alt text][image5]
In the seventh cell of my notebook, I have defined 2 functions to help identify the lane pixels and fit a polynomial based on the identified points.

1. find_lane_pixels(): This function takes in the binary warped image as input and calculates the histogram at the botton of row of pixels to determine the position of the lanes in that row. Based on this positioning, it creates boxes that are used to identify all the points(coordinates) that lie on the lane. This function also outputs an image with the search boxes plotted on them `out_img`.
2. fit_polynomial(): This functions takes the binatry warped image as the input and runs the `find_lane_pixels()` function to find the lane points. Based on these identified points, I did a 2nd degree polynomial fit and also colored the lanes in different colors. The output of these functions is expressed in the following image

![alt text][image6]

In the seventh code cell, I have defined a function `search_around_poly()`, which takes in the binary warped image of the next frame and also the polynomial fit of the lane from the previous frame to search for lane pixels in the existing frame. This reduces computational load and makes the algorithm faster. This algorithm, based on the polyfit, also fills the lane identified with green color. The output is shown here. 

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In the seventh code cell, in the `search_around_poly()` function, after finding the lane points and using the polynomial fit to draw the lane, I have used that information to calculate the curvature of the lane considering, meters per pixel in Y dimesion as, `ym_per_pix = 30/720` and meters per pixel in Y dimesion as, `xm_per_pix = 3.7/700`. We use the formula given in the notes to find the curvature after converting the pixel values into meters. Similarly, the difference between the midpoint of the lane and the midpoint of the vehicle is considered the shift in position of the vehicle from the center of the lane. We then convert this pixel shift to meters using `xm_per_pix`. The additional outpit of `search_around_poly()` finction is the `curvature` and the position of the vehicle from the middle of the lane expressed as the variable `shift`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the eight and ninth cell, we use `cv2.getperspectiveTransform()` function again with reversed inputs to unwrap the image to its original undistorted view and also using the function `status_vehicle()` we display the curvature of the lane in the present frame and the position of the vehicle from the center of the lane.

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Using this pipeline I create a function `pipeline()` in the 11th coding cell and my video result is located at `./output/project_video_output.mp4`



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main problems I faced was to find the correct combination of filters and threshold to creat the binary mask. This was a challenge as I had to do a lot of trials to finally get a mask I was satisfied with. However, as I tried running my code on the challenge videos, I realized that the parameters I had chosen would not work to the best efficiency in those cases. Expecially when there is a lot of shade and spots of sunlight on the road. Also when there are artifacts in the middle of the lane, like we had in the HOV lane in the challenge video.

After running the challenge videos, I also realizedd that hardcoding the lane markings before the perspective transform was also not a robust way of carrying out the perspective transform. As in the harder challenge video, the road winds a lot and out perspective transform fails in this case.

To solve these limitations, we need to find a robust way of detecting lane  pixels which is not effected  by the artifactis on the road and also not effected when the road is windy. The algorithm shoul be able to recognise the exact lane pixels and also recognize, based on the curvature of the lane, the range of lane detection. For example, on the highway, when the lanes are relatively longer, the alogrithm should detect the longer pixel range for the lane. Howeverm for windy roads, the detection should be shorter range.
