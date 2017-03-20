
# **Advanced Lane Finding Project**

## Goals of the project:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


[//]: # (Image References)

[image1]: ./output_images/undist/test2_undist.jpg "Undistorted"
[image2]: ./output_images/Sobel_diag_schan.PNG "Sobel Example (S channel and diagonal sobel )"
[image3]: ./output_images/Transform.PNG "Road Transformed"
[image4]: ./output_images/poly/straight_lines1_undist.jpg "Polygon calculation"

[image5]: ./output_images/pipeline/3warp_color.jpg "Bird´s eye"
[image6]: ./output_images/pipeline/3warp_roi.jpg "Binary Threshold"
[image7]: ./output_images/pipeline/3detected.jpg "Detect lines"
[image8]: ./output_images/pipeline/3detected_color.jpg "Detect lines"
[image9]: ./output_images/pipeline/3result.jpg "Detect lines"
[image10]: ./output_images/test2_sobel.jpg "Messy lines"

[video1]: ./Good_to_go.mp4 "Video"

### Camera Calibration and distortion matrix

Given the chessboard images for a camera, the calibratiion and distortion matrxi are calculated in the first section [Camera Calibration] of the `Cam_Calib.ipynb` 

The process starts defining the object points (x,y,z) of the chessboard corners and image points (x,y)
Then, for each image, will try to find the chessboard corners on each image.

Camera calibration and distorion coefficients are obtained using the `cv2.calibrateCamera` function, this coefficients are stored in a pickle file named `camCal_pickle.p`

Camera distortion is made on section 2: [Distortion Correction], taking parameters from `camCal_pickle.p`

A example of this distortion correction can be seen here:

![alt text][image1]

### Thresholded image

The process of obtaining a thresholded binary image is sections 3 and 4 of `Cam_Calib.ipynb` [Image preprocessing (Gradient threshold)]

An example of this interactive tool can be seen here:

![alt text][image2]

### Prespective transform

A perspective transform is made to the image as to generate a "Birds eye"
look of the road on section 8 of `Cam_Calib.ipynb`

![alt text][image3]

Further description of the pipeline will be made over `video_pipeline.ipynb`

For exemplification purposes, `test2.jpg` will be used as a template, for 
a different test image, the pipeline images can be found on `.\output_images\pipeline`

# Pipeline
## The pipeline makes the following process on this order:
* Distortion correction
* Perpective Transformation
* Binary Thresholding
* Find lane lines
* Calculate Curvature and distance to center

### Distortion Correction
Is made on section 1, takes values from `camCal_pickle.p`, returns
undistorted image (See image above)

### Perspective Transform

For perspective transform the following points were selected:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 230, 700      | 300, 720      | 
| 610, 440      | 300, -200     |
| 670, 440      | 950, -200     |
| 1080, 700     | 960, 720      |

These points were selected using the undistorted `straight_lines1.jpg` by
drawing a polygon over it, and stretching the result as to take the lower
part of the image out of the image (car front)
![alt text][image4]

Here is the result for our example image:
![alt text][image5]

### Binary Thresholding
For binary thresholding, Saturation channel was used along with X sobel,
on the transformed image. Aditionally, a Region of interest was defined as a way to reduce noise (Section 3, `def region_of_interest`)

![alt text][image6]

### Finding Lanes
Lane finding was done with a convolution function, (Section 4,
`def find_window_centroids`), additionally, a custom convloution
function was used; a triangular window was used as to give more significance
to the center of the window.

Once the convolution centers were found, a polynomial regression is used
to fit all the nonzero pixels of the Binary thresholded image to generate
a polimonial representing the lane curvature (`def draw_regression`)

![alt text][image7]

### Curvature and distance to center
Curvature is calculated on `def tangent_circle` using previously found polynomials
using a scale factor to convert between pixels and meters.
Lane center is returned along with the convolution function on `def find_window_centroids`

Regression lines are drawed and then unwarped back to the original image size,
curvature and lane center are then imprinted on the transformed image on
`def unwarp_color`

![alt text][image8]

Then, the detected image is added to the original image inside the pipeline (`def new_pipeline`) to generate the result

![alt text][image9]



# Discussion

The first aproach was to use diagonal direction sobel detect, this worked
pretty good (see image 2, red pixels), however, when doing the birds eye
the binary image got really messy:
![alt text][image10]

The solution for this was to get the birds eye first, and then the binary image,
however, in doing so, diagonal sobel performed poorly, so I took it back to
X sobel, wich worked quite good.

For video stability, I´m using an average of the last 4 measurements along
the current one, conditional on having a sane detection.
Sanity function (Section 5 `def sanity`) takes into account the distance
between lines on the top of image and a maximum variation on this top.
If 15 consecutive frames are marked as Faulty, the curent faulty measurment
is taken into account for the average.

As for ways the pipeline might fail, the first one is going too fast, as
a 5 frames on 60km/h on a 45 frames per second is 24m, the second condition
is fast changing tracks, as the one in the harder challenge video, where
curvature changes really fast to make the buffer average fail.

Things that were not experimented on are lighting conditions, were the 
saturation channel might change considerably, as a way to prevent this,
equalization could be tried out.

Here's a [link to my video result](./Good_to_go.mp4)