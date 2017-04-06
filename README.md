This is my submission for the Udacity Self-Driving Car Nanodegree Advanced Lane Detection Project. You can find my code in this [Jupyter Notebook](https://github.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/blob/master/AdvancedLaneDetection.ipynb). Here I improve on my first [Lane Detection Project](https://pszczesnowicz.github.io/SDCND-P1-LaneDetection/) by employing more advanced image thresholding and detection techniques.

## Summary

My pipeline's steps are as follows:
1. Calibrate the camera
2. Threshold the image gradients and color spaces
3. Mask the image outside of a region of interest
4. Warp the image into a bird's eye view perspective
5. Detect the lane locations
7. Calculate the road's curvature radius and the vehicle's relation to lane center
8. Highlight the detected lane and draw information onto the image

The following sections explain my pipeline's steps in more detail.

## Pipeline

### Camera Calibration

I start off my pipeline by computing a camera calibration matrix which will be used to correct for radial distortion caused by the camera lens. 

Images of a chessboard taken at various angles and distances are used to compute the camera calibration matrix. The chessboard inner corners are represented by a 9x6 matrix of coordinate object points. The calibration images are converted into grayscale and passed into `cv2.findChessboardCorners` which, as the name implies, finds and outputs a matrix of coordinate image points. The object and image points are passed into `cv2.calibrateCamera` which outputs the camera calibration matrix. Finally, `cv2.undistort` is used to correct images taken with the same camera as the calibration images.

Chessboard calibration images:

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/chessboard.jpg" width="800">

A test image before distortion correction:

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/input.jpg" width="400">

And after:

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/undistorted.jpg" width="400">

### Gradient and Color Thresholding

In order to reliably detect the lanes in varying lighting and road surface conditions, several thresholds of the image gradients and color spaces are computed.

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/thresholding.jpg" width="800">

Once the binary images are computed they are combined using `AND` and `OR` operators:

`binary_comb[((binary_gradx == 1) & (binary_grady == 1)) | ((binary_grad_mag == 1) & (binary_grad_dir == 1)) | ((binary_rch == 1) & (binary_sch == 1))] = 1`

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/combined_thresholding.jpg" width="400">

### Masking

Even though gradient and color thresholding does a good job of isolating the lanes there is still noise in the form of tree and car outlines. That is why the image is masked outside of a region of interest defined by the following mask points.

| Mask Points |
|:------------|
| 745, 450    |
| 1130, 720   |
| 150, 720    |
| 555, 450    |

Masked image:

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/masked.jpg" width="400">

### Warping

A bird's eye view perspective shows the lane curvature more accurately than the driver's perspective. The following source and destination points are used to compute a transformation matrix which is used to warp the image into the bird's eye view perspective.

| Source Points | Destination Points |
|:--------------|:-------------------|
| 685, 450      | 990, 0             |
| 1090, 720     | 990, 720           |
| 190, 720      | 290, 720           |
| 595, 450      | 290, 0             |

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/perspectives.jpg" width="800">

Warped image:

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/warped.jpg" width="400">

### Lane Detection, Road Curvature Radius, and Vehicle Relation to Lane Center

A sliding window technique is used to identify the lanes in the first image that is passed through the pipeline. The starting coordinate points are identified by the peaks of a histogram evaluated at the bottom half of the image. The following coordinate points are identified by sliding the windows up from their starting points and locating the non-zero pixels that fall within the window bounds. The windows are centered on the average location of these non-zero pixels if their quantity is greater than a set value. A polynomial is then fit to each set of left and right coordinate points.

Following the sliding window technique, the lanes are identified by locating all non-zero points that fall within the previously computed polynomials +/- a margin. A polynomial is then fit to each set of left and right coordinate points.

The current and previously identified coordinate points are compared and their polynomial coefficients are stored if they fall within a margin of each other. Otherwise, the lanes are identified from scratch using the sliding window technique.

To achieve a greater detection accuracy I averaged the left and right lane polynomial coefficients with each other and those previously computed. This reduces the effects of incorrectly identified co-ordinate points due to noise in the image.

I then used the polynomial coefficients to compute the road's curvature radius at the bottom of the image. I also compute the vehicle's relation to lane center by subtracting the polynomial x-point at the bottom of the image from half of the image's width.

Additionally, I included an arrow pointing in the direction the driver would need to steer in order to center the vehicle; it shows up as either green, yellow, or red depending on the vehicle's deviation from the lane center. This is purely cosmetic and I added it while I was stuck on another section of the pipeline to keep myself busy.

Output image with the lane highlighted and the road's curvature radius and vehicle's relation to lane center displayed:

<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/output.jpg" width="400">

## Results

[<img src="https://raw.githubusercontent.com/pszczesnowicz/SDCND-P4-AdvancedLaneDetection/master/readme_images/video_01.jpg" width="800">](https://www.youtube.com/watch?v=xs40-6d_F4E "Click to watch")

## References
[Udacity Self-Driving Car ND](http://www.udacity.com/drive)

[Udacity Self-Driving Car ND - Term 1 Starter Kit](https://github.com/udacity/CarND-Term1-Starter-Kit)

[Udacity Self-Driving Car ND - Advanced Lane Detection Project](https://github.com/udacity/CarND-Advanced-Lane-Lines)
