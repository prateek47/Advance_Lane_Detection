
## Writeup Template:-

#### These are the steps used to detect lane lines in the project:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
*Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


[//]: # (Image References)

[im01]: ./output_images/undistorted%20camera%20calibration%20image2.png
[im02]: ./output_images/undistorted_image.png
[im03]: ./output_images/thresholded_image1.png
[im04]: ./output_images/different_thresholds_applied.png
[im05]: ./output_images/src_points_img.png
[im06]: ./output_images/undistortedVSwraped.png
[im07]: ./output_images/original_wrapimg1.png
[im08]: ./output_images/original_wrapimg2.png
[im09]: ./output_images/original_wrapimg3.png
[im10]: ./output_images/original_wrapimg4.png
[im11]: ./output_images/original_wrapimg5.png
[im12]: ./output_images/original_wrapimg6.png
[im13]: ./output_images/sliding_window.png
[im14]: ./output_images/histogram_sliding_window.png
[im15]: ./output_images/using_prevfit.png
[im16]: ./output_images/colored_lane.png
[im17]: ./output_images/data_oncoloredlane.png
[im18]: ./output_images/testing_slidingwindow.png

[video1]: ./project_video_output.mp4 "Video"

----
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the jupyter notebook: `Camera Calibration Code.ipynb`.

The camera_cal folder consist of all the images(20), taken from different angles with the same camera, is used as input.
while, the output_files folder consist of all the calibrated images, there are only 17 images present as compared to 20 images in camera_cal folder because the function wasn't able to detect the desired corners for those images.
The final calibrations is stored in `calibration_pickle.p`.

The OpenCV functions `findChessboardCorners` and `calibrateCamera` are the backbone of the image calibration function. The main aim is to To map the corner of the 2-d chessboard image (image points) to the 3-D coordinates of undistorted chessboard corners(obj points). Arrays of object points, i.e corresponding 3-D location of corners and image points, the 2-D location of corners in images plane are created. 

`findChessboardCorners`, is used to find the internal corner of a chessboard, which are then fed to `calibrateCamera` which returns camera calibration and distortion coefficients. These coeff are then used in OpenCV `undistort` function to undo the effects of distortion on any image. For a given camera, the coefficients do not change. The below image presents an example of the calibrated and undistorted chessboard image.

![alt tag][im01]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this part is present in `Pipeline(Single Images) Code.. Project4` code cell 6. The image below shows the results of applying OpenCV function `undistort` to one of the images:

![alt tag][im02]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.


I tried several combinations of sobel gradient thresholds and color channel thresholds. I ended up with using a combination of HLS color gradient and sobelx and sobely gradient threshold. The code for this part is present in `Pipeline(Single Images) Code.. Project4` from code cell 7 to 11 (under Q2). Below is the example of the image for the combination

![alt tag][im03]

I also tried the same image for all threshold I tried. Below is an image showing all the results

![alt tag][im04]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this part is present in `Pipeline(Single Images) Code.. Project4` under the heading Question 3, from code cell 12 to 18. 

Firstly, its important to check the points in the image which can be used to form a polygon that represents the lane, eg in the below image

![alt tag][im05]

using these points as reference points, source and destination points are decided are a particular camera calibration

```
src = np.float32([[570,475], [270, 705], [1125,705], [755, 475]])
dst = np.float32([(320,0), 
                  (320,img.shape[0]),
                  (img.shape[1]-320,img.shape[0]),
                  (img.shape[1]-320,0)])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 475      | 320, 0        | 
| 270, 705      | 320, 720      |
| 1125, 705     | 960, 720      |
| 755, 475      | 960, 0        |


using the OpenCV `getPerspectiveTransform` function, which takes in the source and destination points and returns the mapping as a perspective image, a wraped image was created

![alt tag][im06]

The prespective transform is working as planned, as the lanes appear to be parallel to each other.

Below images show a combination of first performing a perpective transformation and then finding the threshold of the image, we looking at the original image and the new created **wraped_binary** image we can observe that the lanes lines are correctly recognised.

**Test Image 1***
![alt tag][im07]

**Test Image 2***
![alt tag][im08]

**Test Image 3***
![alt tag][im09]

**Test Image 4***
![alt tag][im10]

**Test Image 5***
![alt tag][im11]

**Test Image 6***
![alt tag][im12]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Using the **binary wraped** image from previous section, the functions `sliding_window_fit` identifies lane lines and fit a second order polynomial to both left and right lanes, the code for the function is present in `Pipeline(Single Images) Code.. Project4` under Q4. from code cell 19-20.

The function identifies ten windows(equally spaced wrt to height of image) to identify lane pixels. Each window is centered on the midpoints of the pixels in the window below it. Each window effectively follows the lane lines adjusting itself as lane curves right or left. By plotting it over a binary images, the speed of processing is increased as it searches only for activated pixels over a small portion of the image. The Numpy `polyfit()` method used to fit a second order polynomial to each set of pixels. 

**NOTE: ** Test image 6 is used below.

![alt tag][im13]

The code also computes a histogram of the bottom half of the image and find the bottom-most 'x' position of the left and right lanes. Originally the locations were detection as local maxima of the both left and right halves, but I changed the implementation to 2nd and 3rd quater (i.e. left and right quater of mid-point) when we split the image in 4 parts. The aim for changing the implementation is to avoid other lane detection.

![alt tag][im14]

The `Fit_using_prev_img`, replicates the same process, but reduces the difficult and processing time spent on searching. It utilizes the lanes line draws created from the previous image, until and unless there is a drastic difference between new and old lane line. It seaches for lanes lines withing a certain `margin` of the previous fit. In the below picture, the green portion shows the margin it searches (created using previous fit) and yellow lines show the present fit, while red and blue pixels represents lanes.The code for the function is present in `Pipeline(Single Images) Code.. Project4` under Q4. from code cell 22-23.

**NOTE:** images test6, test4, test1 and test5 appears to taken one after the other on the same road and conditions. so, they can be used as continous images for testing before modifying the code for video(which can be considered as just a set of continous images taken one after another). There test image 4 is used below.

![alt text][im15]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code to calculate radius of curvature is written in form of a function in `Pipeline(Single Images) Code.. Project4` under Q5 from code cell 24-25. The formula is based upon the explanation provided [here](http://www.intmath.com/applications-differentiation/8-radius-curvature.php):

```
 radius of curve = ((1 + (2*fit_cr[0]*y_val*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
 
```
In this example, `fit_cr[0]` is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and `fit_cr[1]` is the second (y) coefficient. `y_val` is the y position within the image upon which the curvature calculation is based (the bottom-most y - i.e. the position of the car in the image - was chosen). `y_m_per_pixel` is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters. 

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:
```
# Distance from center is image x midpoint - mean of left_fit and right_fit intercepts 
    if left_fit is not None and right_fit is not None:
        center_ofimage = binary_wraped_image.shape[1]/2
        left_fit_x = left_fit[0]*h**2 + left_fit[1]*h + left_fit[2]
        right_fit_x = right_fit[0]*h**2 + right_fit[1]*h + right_fit[2]
        
        lane_center_position = (right_fit_x + left_fit_x)/2
        center_dist = (center_ofimage - lane_center_position) * xm_per_pix
```
`right_fit_x` and `left_fit_x` are the x-intercepts of the right and left fits, respectively. This requires evaluating the fit at the maximum y value (719, in this case - the bottom of the image) because the minimum y value is actually at the top (otherwise, the constant coefficient of each fit would have sufficed). The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle).

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.


The code for this part is present in Pipeline(Single Images) Code.. Project4 under Q6 from code cell 26-30.

The function `color_driving_lane` is written that takes in left and right fit from question 4 and binary wraped image and inverse perspective coefficient to create a colored polygon wrt to the left and right fits and overlays it over the original image. The coloured polygon represents the lane w.r.t the camera on which the vehicle should move.
I used test image 2 as input for this part( as the road turns left side as compared to right side in input examples used above, which would help test the code)

![alt tag][im16]

The function `plot_rad_dcenter`, takes in the radius of curvature and distance from center value from `curvature_rad_center_dist` function and prints the data on the image obtained above.

![alt tag][im17]

Just for testing, I also drew the sliding window function on the image

![alt tag][im18]


### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).


The code for this part is written in form a function called `process_images` present in `Pipeline(Single Images) Code.. Project4` under **Pipeline(Video)** from code cell 32. I have also defined a class `Line()` to store the values of previous fits and add new fits to be laid over the video images in cell 31. 

Here's a [link to my video result](./project_video_output.mp4)

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

** Problems Encountered and Steps Taken :**
1. Firstly, the lighting, shadows caused some problems while detection of lane lines. But using different combinations of thresholding functions, like sobel x and y gradients and colours(HLS) helped mostly by pass most of the difficulties. for example even on the ligher gray bridge sections present in project video, the algorithm does reasonably well to detection lane lines. 
    While trying to extend the same pipeline to challenge video, the lane lines do not necessarily predict the lane line prefectly, due to shadow and color of lane and pretty close edge of the wall in the left. I think using combination of different color channels can help improve the prediction on the challenge videos.
    This could also be a issue during snow and night images, where the white lanes could not be easily differentiated from other part of the road on basis of color. Smoothing the video output by averaging n fits can help.
    Also, I used another approach to invalidate left and right lanes, if they are not present at a certain distance(with some margin), under the assumption that lane width does not change.
    
2. Selecting the area to be used for mapping the prespective image. Due to car movement within the lanes, the polygon chosen that fits one images does not necessarily perfectly fit other images, due to which, the polygon drawn over the lane lines sometimes does not cover the entire lane.
 
** Steps Considered to Robust the model: **
1. Dynamic thresholding is an option to look into i.e. using different thresholding parameters for different parts of the image.
2. Using a confidence interval for selecting the fits and rejecting new fits is also an option
3. Using lane width as a parameter to reject new fits(implemented in the code in `process_images` function.
4. Assigning a limit of the change of radius of curvature between left and right fit. If one fit curve quite more than the other fit, it can used as parameter to reject that fit(assumption the fit width always remains constant(with some margin)).



Here are the link to my [challenge video](./challenge_video_output.mp4)


```python

```
