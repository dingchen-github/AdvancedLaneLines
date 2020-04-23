# Writeup

The whole code is contained in the IPython notebook located in "./Project.ipynb".

[image1]: ./output_images/undistorted.png "Undistorted"
[image2]: ./undistorted_images/1.jpg "Road Transformed"
[image3]: ./output_images/threshold2.jpg "Thresholding"
[image4]: ./output_images/warped2.jpg "Warp Example"
[image5]: ./output_images/histogram2.jpg "Histogram"
[image6]: ./output_images/warpToOri2.jpg "Output"
[image7]: ./output_images/searchAroundPoly2.jpg "searchAroundPoly"
[image8]: ./output_images/frame1.png "video output"
[image9]: ./output_images/frame2.png "video output"
[image10]: ./output_images/frame3.png "video output"
[image11]: ./challenge/frame1.png "challenge output"
[image12]: ./challenge/frame2.png "challenge output"
[image13]: ./challenge/frame3.png "challenge output"
[image14]: ./harder_challenge/frame1.png "harder challenge output"
[image15]: ./harder_challenge/frame2.png "challenge output"
[image16]: ./harder_challenge/frame3.png "challenge output"
[image17]: ./harder_challenge/frame4.png "challenge output"
[image18]: ./harder_challenge/frame5.png "challenge output"
[image19]: ./harder_challenge/frame6.png "challenge output"
[image20]: ./harder_challenge/frame7.png "challenge output"
[image21]: ./harder_challenge/frame8.png "challenge output"

## Image pipeline

#### 1. Camera Calibration

My steps are:

1. Prepare object points "objp", which are 3D coordinates of undistorted real world corner points, with Z as 0.
2. Create emtpy lists "objpoints" and "imgpoints" to store object points and image points from all the images.
3. Read in the images. Here I prefer to use `cv2.imread()` to get an image list directly instead of using glob.
4. Step through the list and search for chessboard corners using `cv2.findChessboardCorners()` and `cv2.drawChessboardCorners()`.
5. Get camera matrix and distortion coefficients using  `cv2.undistort()`.

I tested the calibration, used `plt.subplots()` and `plt.savefig()` to save the results and got this:
![alt text][image1]

I read in the test images and undistorted them. One of the undistorted images is:
![alt text][image2]

#### 2. Thresholding function `thresh()`

For this function, I combined the following algorithmen:
* Sobel gradients in x and y direction. Threshold is (20, 255).
* magnitude of gradients. Threshold is (10, 255).
* direction of gradients. Threshold is (0.7, 1.3).
* HLS color space with the S channel. Threshold is (90, 255).
* RGB color space with the R channel. Threshold is (20, 255).

Then I tested the function and one result is:
![alt text][image3]

#### 3. Perspective transform function `warp()`

I used `cv2.getPerspectiveTransform()` and `cv2.warpPerspective()` to realize the function.

I eyepicked the source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 220, 700      | 300, 720      |
| 578, 460      | 300, 0        |
| 705, 460      | 1000, 0       |
| 1100, 700     | 1000, 720     |

In order to test the function, I need to draw red lines on binary images, which is why I used the following code to first convert the binary images to RGB.
```python
warped_rgb = np.zeros((warped.shape[0],warped.shape[1],3), dtype="uint8")
warped_rgb[:,:,0] = warped * 255
warped_rgb[:,:,1] = warped * 255
warped_rgb[:,:,2] = warped * 255
```

One test result is:
![alt text][image4]

---

## Project video pipeline

#### 1. Find lane pixels using histogram `find_lane_pixels()`

I did the following steps:
* Put histogram on the lower half of the image
* Find the peak of the left and right halves of the histogram
* Set hyperparameters (number of sliding windows, width of the windows +/- margin, minimum number of pixels found to recenter window)
* Identify the x and y positions of all nonzero pixels in the image
* Step through the windows one by one
  * Identify window boundaries in x and y and left and right
  * Draw the windows on the visualization image
  * Identify the nonzero pixels in x and y within the window
  * Append these indices to the lists
  * If minimum number of pixels is found, recenter next window on their mean position
* Concatenate the arrays of indices (previously was a list of lists of pixels)
* Extract left and right line pixel positions

#### 2. Fit polynominal `fit_polynomial()`

* Use `find_lane_pixels()` to get lane pixels
* Fit a second order polynomial to each using `np.polyfit()`
* Generate x and y values for plotting
* Visualize

I tested the above two functions and one of the results is:
![alt text][image5]

#### 3. Measure real curvature `measure_curvature_real()`
* Define conversions in x and y from pixels space to meters
* Use `find_lane_pixels()` to get lane pixels
* Calculate corrected fitting
* Define y-value where I want radius of curvature, which is at the bottom of the image
* Implement the calculation of R_curve

#### 4. Calculate offset left of center `offset_left_of_center()`
* Take a histogram of the bottom half of the image
* Find the midpoint and the peak of the left and right halves of the histogram
* Calculate the lane middle using both peaks
* Define conversions in x from pixels space to meters
* Calculate offset using midpoint and lane middle

#### 5. Warp to original image `warp_to_original()`
* Create a color warped blank image
* Get polynomial fitting using `fit_polynomial()`
* Calculate the left and right x fitting values
* Recast the x and y points into usable format for `cv2.fillPoly()`
* Draw the lane onto the warped blank image
* Warp the blank back to original image space using inverse perspective matrix (Minv)
* Combine the result with the original image

I tested this function and one of the results is:
![alt text][image6]

#### 6. Search around polynominal `search_around_poly()`
* Choose the width of the margin around the previous polynomial to search
* Grab activated pixels
* Set the area of search based on activated x-values within the +/- margin
* Extract left and right line pixel positions
* Fit new polynomials
* Visualize
  * Create an image to draw on and an image to show the selection window
  * Set color in left and right line pixels
  * Generate a polygon to illustrate the search window area and recast the x and y points into usable format for `cv2.fillPoly()`
  * Draw the lane onto the warped blank image
  * Plot the polynomial lines onto the image

I tested this function and one of the results is:
  ![alt text][image7]

#### 7. Steamline the above functions for video processing

Now that the above tests have succeeded, I know the previous functions work. But some of them contain lots of plotting features, which were good for visualization in the tests, but are not necessary for video processing. Some input values should also be optimized; since I have search around polynominal function, I don't want to take in an binary warped image and use histogram function every time.

So I streamlined these functions by adjusting the input and output values.

Note that camera calibration, thresholding `thresh()` and perspective transform function `warp()` are still defined in the image pipeline.

|Original functions|Steamlined functions|Difference|
|:----------:|:----------:|:----------:|
|find_lane_pixels (binary_warped)|find_lane_histogram (binary_warped)|Does not return an output image|
|fit_polynomial (binary_warped)|fit_poly(leftx, lefty, rightx, righty)|Does not take the binary warped image and use histogram; Does not return an output image|
|measure_curvature_real (binary_warped)|measure_curvature_meter(leftx, lefty, rightx, righty)|Does not take the binary warped image and use histogram
|offset_left_of_center (binary_warped)|offset_left_of_center(leftx, rightx)|Does not take the binary warped image and use histogram|
|warp_to_original(undist, binary_warped, src, dst)|warp_to_orig(undist, binary_warped, left_fit, right_fit)|src and dst are defined inside the function; Does not use fit polynomial function, but takes the fitting results
|search_around_poly (binary_warped)|search_from_prior(binary_warped, left_fit, right_fit)|Does not return an output image, but the identified x and y values|

#### 8. Define a lane line class
* was the line detected in the last iteration? (self.detected)
* x values of the last n fits of the line (self.recent_xfitted)
* average x values of the fitted line over the last n iterations (self.bestx)
* polynomial coefficients averaged over the last n iterations (self.bestfit)
* polynomial coefficients for the most recent fit (self.current_fit)
* radius of curvature of the line in meter (self.radius_of_curvature)
* distance in meters of vehicle center from the line (self.line_base_pos)

#### 9. Sanity check
* Curvature check
  * Calculate the left and right curvature difference
  * If both calculated curvatures are > 3 km, they are deemed to be a straight lane, and I don't check the difference
  * Otherwise, I take a difference of more than 500% as failed
* Distance and parallel check
  * Pick top, middle and bottom value of y to see if left and right x always have pixel distances of near 700 (3.7 m)
* Return a boolean value with True as failed

#### 10. Video processing function `process_image()`
* Input values:
  * img: the frame
  * mtx, dist: camera matrix and distortion coefficients
  * count: an integer as counter
  * left_line, right_line: instances of identified lane lines
* Transform to undistorted image, do thresholding, get the warped binary image
* Get identified x and y pixels
  * If counter is 0 and left_line.detected is False, which means either this is the first frame, or we have lost lines for multiple frames in a row, use `find_lane_histogram()`
  * Otherwise, use `search_from_prior()`
* Get the fitted polynomial and the processed image
  * If x or y pixels could not be identified in the previous step, it is similar to sanity checked failed, and data from the previous frame will be used
  * Otherwise, use `fit_poly()` to get fitted polynominal
  * Create new instances to store latest data
  * Run sanity check on the instances
  * If failed, use data from the previous frame and add 1 to counter
  * If passed,
    * reset counter to 1
    * get fitted x and y values
    * if Line.recent_xfitted has 10 entries, delete the oldest entry (turn the list into an array and use `np.delete()`) and add the latest one (turn the array back into a list and use `append()`)
    * otherwise just add the latest values
    * calculate left and right x mean values using `np.mean()` on axis=0
    * use the mean values to calculate the best fitted polynomial
    * calculate the radius of curvature and offset
  * Use `warp_to_orig()` and line.best_fit to get the processed image
  * Use `cv2.putText()` to add text information about radius of curvature and offset to the frame
* Return the processed frame, the counter and the left and right lane line instances

#### 11. Video processing
* Read in the video using `VideoFileClip()`
* Create an empty list to store the processed frames
* Create instances of identified left and right lane lines
* Set counter to 0
* Do a loop to process every frame
  * Use `plt.close()` to erase possible visualization effects from the last frame
  * Get the frame using `get_frame()`
  * Get the processed frame, the counter and the left and right lane line instances using `process_image()`
  * Append the frame to the list
  ```python
  for i in np.arange(0.0, clip1.duration, 1 / clip1.fps):
      plt.close()
      img = clip1.get_frame(i)
      result, count, left_line, right_line = process_image(img, mtx, dist, count, left_line, right_line)
      clips.append(result)
  ```
* Turn the list into a sequence using `ImageSequenceClip()`
* Write the sequence into the output video using `write_videofile()`

#### 12. Result

The [output video](https://github.com/dingchen-github/AdvancedLaneLines/blob/master/project_output.mp4) is satisfactory. Some of the frames see below:
![alt text][image8]
![alt text][image9]
![alt text][image10]

---

## Challenge video pipeline

`thresh()`, `warp()` and `warp_to_orig()` are changed. I tested them and adjusted the parameter.

For thresholding, I used the following thresholds:
* Sobel gradients in x and y direction. Threshold is (10, 255).
* magnitude of gradients. Threshold is (5, 255).
* direction of gradients. Threshold is (0.7, 1.3).
* HLS color space with the S channel. Threshold is (3, 255).
* RGB color space with the R channel. Threshold is (120, 255).

For `warp()` and `warp_to_orig()`, I used the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 295, 700      | 300, 720      |
| 600, 490      | 300, 0        |
| 730, 490      | 1000, 0       |
| 1030, 700     | 1000, 720     |

The [challenge output video](https://github.com/dingchen-github/AdvancedLaneLines/blob/master/challenge_output.mp4) is mostly satisfactory. Some of the frames see below:

The shadow of the left wall as well as the black lines in the middle of the current lane and the adjacent lane do not cause confusion to the algorithmen.
![alt text][image11]
The shadow under the bridge does not bring a big problem, just the radius of curvature is a bit overestimated (see the far end of the green area).
![alt text][image12]
The adjacent car does not cause a problem.
![alt text][image13]

---

## Harder challenge video pipeline

This one is trully brutal!

Again, I adjusted the thresholds from the challenge video processing:
* direction of gradients. Threshold is (0.5, 1.3), because in a sharp turn, the lane lines can have a smaller radian.
* RGB color space with the R channel. Threshold is (50, 255), to let more white lanes to be identified.

For the source and destination points, I decided to cut the region of interest to almost a half, so that the lane lines are still within the warped image.

| Source        | Destination   |
|:-------------:|:-------------:|
| 200, 700      | 300, 720      |
| 350, 595      | 300, 0        |
| 880, 595      | 1000, 0       |
| 1000, 700     | 1000, 720     |

The [harder challenge output video](https://github.com/dingchen-github/AdvancedLaneLines/blob/master/harder_challenge_output.mp4) is not feasible for real autonomous driving. Some of the frames see below:

The motorcycle is ok. The yellow grass outside the right lane line is bright on the binary warped image. If I suppress it too hard, I lose lane lines in other frames.
![alt text][image14]
Slight horizontal light stripes are ok.
![alt text][image15]
Strong horizontal light stripes cause lane line to curve.
![alt text][image18]
In the shadow, the grass is even brighter.
![alt text][image16]
On the contrary, shadow and light together are not a big issue.
![alt text][image17]
And in the complete shadow, the code does its job well.
![alt text][image19]
In a sharp turn, the algorithmen struggle to keep on.
![alt text][image20]
When the right lane line is out of camera scope, the algorithmen go mad. A camera with a wider field of view could be useful.
![alt text][image21]

## Summary

Overall I am satisfied with my pipelines. The harder challenge shows that under difficult situations, the code needs to be further optimized. From the aspect of hardware configuration, other sensors like LIDAR and RADAR can be used to assist camera, which is why the Udacity Nanodegree Sensor Fusion is very interesting!
