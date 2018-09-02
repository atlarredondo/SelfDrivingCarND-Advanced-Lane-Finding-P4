## Write Up Template

### You can use this file as a template for your write up if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./writeup_imgs/distortion_correction.png "Undistorted"
[image2]: ./writeup_imgs/road_distortion_correction.png "Road Transformed"
[image3]: ./writeup_imgs/gradient_image.png "Binary Example"
[image4]: ./writeup_imgs/warped.png "Warp Example"
[image5]: ./writeup_imgs/finding_lanes.png "Fit Visual"
[image6]: ./writeup_imgs/find_lane.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Write Up / README that includes all the rubric points and how you addressed each one.  You can submit your write up as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The camera was calibrated on the first three code cells from the pipeline.ipynb notebook.
First, I declare the object points which represent the chessboard corners on the x, y and z coordinates. In the project I ignored the z coordinate and set it to 0.
To calibrate the camera the checkerboard images on the `camera_cal` directory were used. These images were converted to grayscale and the `cv2.findChessboardCorners` function was used to obtain the corners on the checkboards. 
The object points and corners for each of the images was appended to two arrays if the corners were found on the image. 

To find the calibration parameters the `cv2.calibrateCamera` was used. 
Finally the images were undistorted by using the `cv2.undistort` function on the chessboard images for validation and later on the video frames.

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image. 

At each frame the `cv2.undistort` function is called. The parameters used on this function came from the third cell on this notebook by using the checkerboard images and on the `cv2.findChessboardCorners` and `cv2.calibrateCamera` functions. 
Here is an example image:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

To obtain the final gradient of the frames, I used a combination of the gradients in the x, y, the magnitude, direction of the gradient and color threshold on the saturation channel. This is specified in the `get_image_gradient` method on the Line object.
Here's an example of my output for this step:
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform part of the pipeline is applied on the `Line.apply_perspective_transform` class method. To change the perspective of the frames to a bird like view I used the `cv2.warpPerspective` function. To use this function I first collected the points on the image that corresponded to the road(src_pnts) and the destination points. 
These is how the destination and source points are declared on the class:

```python
self.src_pnts = np.float32([(550, 465), (730, 465), (1090, 680), (210, 680)])

offset = 100
self.dst_pnts = np.float32([[offset, offset], [self.img_width-offset, offset], 
                                         [self.img_width-offset, self.img_height-offset], 
                                         [offset, self.img_height-offset]])
        
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 550, 465      | 100, 100        | 
| 210, 680      | 100, 620      |
| 1090, 680     | 1180, 620      |
| 730, 465      | 100, 620        |

To verify the source and destination points I used the matplotlib interactive view.
This is an example of the results:

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To find the lane pixels I first calculated the middle, left and right line starting point by identifying the peaks on the histogram from the perspective transformed image.

Then after I found my starting point I started looking for the pixels that belong to the line by sliding a window upwards in the y direction. I marked all the pixels that I believe belonged to the lane and move the widows with respect to the pixels found. After all of pixels were found I use the polyfit function in order to find the best 2nd line polynomial parameters.
On the subsequent frames, I decided to find the new pixels based by looking within a margin of the previous frame pixels.

This operations happened on the `Line.fine_first_lines` and `Line.find_line` methods.

Here is an image of the lane pixels found colored in red and green, and the fitted lines.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center. TODO

I found the radius of curvature by first fitting a second line using the lane pixels scaled to meters.  These new pixels were calculated by calculating x values using the polynomial and over the y pixel axis.  After I got the second polynomial I used the radius of curvature formula:

Rcurve​=∣2A∣(1+(2Ay+B)2)3/2​


The position of the vehicle with respect of the center was calculated by looking at the camera position first. This was done by getting the camera center based on the left and right lanes x axis. After obtaining the center the difference was calculated by getting the difference between the center and the middle of the frame(width of the image divided by 2). After, getting the pixel difference I just transformed the pixels to meters.
These operations are written on the `Line.get_curvature_offset` method

In order to reduce noise I used a rolling window array in order to take the average of the current frame and the last 4. This operations is written on the `Line.find_lines` method where the `Line.average_value` is called on the curvature and center_diff values.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly. TODO

The results were plotted back to the image on the `Line.draw_lines` method. This method takes in the original image the warped image and the left and right line pixel coordinates. `cv2.fill_polly` is called to draw the line pixel coordinates. After the lines are drawn `cv2.warpPerspective` is then called to warp the image down to the original perspective. 
Finally, the final original image and the warped line images get merge together by using the `cv2.addWeighted` function.
Here is an example of the image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Link:
https://www.youtube.com/watch?v=d3lzxEIXhMU

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The first issue that I encounter was on finding the pixels that belong to the lanes. This task was the most complex for me I had to dedicate an extra amount of time understand it well. At the beginning I was not getting the correct line parameters and when I plotted on the perspective transformed image the line it was far off. After some debugging I noticed that this was related to the noise of the gradient image pixels.

I decided to check the pipeline downstream and I discovered that I was missing important data and that the gradient parameters were not the best ones. To fix this I changed the source points for the perspective transform to get as many lane pixels as possible. Then, I took a look at the gradient parameters. I changed the direction gradient parameters to the ones that worked on the lesson and incorporated a mix all of the different gradient sources. These changes fixed the curved lines for most of the test images.

The second issue that I encounter was the tree shadow on the road. One of the test images had a really big shadow that messed up with the gradient pixels. I noticed that the source of this shadow issue was mostly coming from the saturation channel. I increased the min threshold parameters to be more strict and it fixed the issue. 
I encounter this issue again when I ran the pipeline on the video. I tuned the parameters again and fixed it for the most part.

In the future I would like to invest more time on the edge cases. The shadows was a really good example of edge cases where the pipeline just doesn't work and can lead to a catastrophic failures. I think these issues can be fixed by ignoring areas of the image where line pixels are not found. As an example, we could filter pixels on the center between the lanes. In addition, using rolling averages or filters can help with these type of data streams.



