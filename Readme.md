
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

[im01]: ./camera_cal/calibration2.jpg "Original Image"
[im02]: ./output_files/calibrated2.jpg "Undistorted Image"

----
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the jupyter notebook: `Camera Calibration Code.ipynb`.

The camera_cal folder consist of all the images(20), taken from different angles with the same camera, is used as input.
while, the output_files folder consist of all the calibrated images, there are only 17 images present as compared to 20 images in camera_cal folder because the function wasn't able to detect the desired corners for those images.
The final calibrations is stored in `calibration_pickle.p`.

The OpenCV functions `findChessboardCorners` and `calibrateCamera` are the backbone of the image calibration function. The main aim is to To map the corner of the 2-d chessboard image (image points) to the 3-D coordinates of undistorted chessboard corners(obj points). Arrays of object points, i.e corresponding 3-D location of corners and image points, the 2-D location of corners in images plane are created. 

`findChessboardCorners`, is used to find the internal corner of a chessboard, which are then fed to `calibrateCamera` which returns camera calibration and distortion coefficients. These coeff are then used in OpenCV `undistort` function to undo the effects of distortion on any image. For a given camera, the coefficients do not change. The below image presents an example of the calibrated and undistorted chessboard image.

![alt tag] [im01][im02]


```python

```


```python

```
