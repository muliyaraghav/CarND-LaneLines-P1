# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

### Goals of the Project
The Goals / Steps of this Project are the following:
- Make a Pipeline (using tools like color selection, region of interest selection, grayscaling, Gaussian smoothing, Canny Edge Detection and Hough Tranform line detection) that Finds Lane Lines [Left & Right] on the Road from the given Input Images and Videos and then, track the detected Lane Lines with:
  - Stage-1: Line Segments
  - Stage-2: Solid Lines (Averaged/Extrapolated Single Line Segments drawn on to the Line Segments in the image)
- Reflect on the Project Work as a project report below in this file
 
---

### Reflection

### 1. Description of the Lane Detection Pipeline
My pipeline draw_lines() function, consisted of 7 steps. 
  1. Convert the Input Image into Grayscale
  2. Smoothing [Gaussian Blur](https://docs.opencv.org/master/d4/d13/tutorial_py_filtering.html)
  3. Edge Detection [Canny Edge Detection](https://docs.opencv.org/master/da/d22/tutorial_py_canny.html)
  4. Apply a Region of Interest (ROI) Mask  
  5. Detect Lines [Hough Lines](https://docs.opencv.org/3.4/d9/db0/tutorial_hough_lines.html) & [Draw Lines](https://docs.opencv.org/master/d6/d6e/group__imgproc__draw.html) as      Line Segments
  6. Draw Single Solid Line per lane by filtering, Averaging and Extrapolation
  7. Overlay the Detected Line Segments / Lines on Original Input Image so as to track the Lane Lines


using the below example image, I will explain the output of each step above. 
solidWhiteRight.jpg
![Example Input Image](https://github.com/xxx/UDACITY_SDCarEngg-ND_P1--Prj01-Lane/blob/master/xxx/0_SolidWhiteCurve_In.jpg "Image_Input")

####  1. Convert the Input Image into Grayscale
  In this step, I converted the Color Image to a 1-Channel Grayscale Image. We do this to be convenient to perform the Image Processing Algos in the upcoming Steps.
  For this I used the cv2 Function "cv2.cvtColor(input_image, flag)", which converts an image from one color space to another. 
  Here, 
  - "input_image" is the Input Image
  - "flag" determines the type of conversion, which is cv2.COLOR_BGR2GRAY for BGR -> Gray conversion
  The output image of this step on the example image "solidWhiteRight.jpg" is as below. 

####  2. Smoothing Gaussian Blur
  Edge detection results are easily affected by the noise in the image. So, before the edge detection, it is essential to filter out the noise to prevent false detection caused by it. This step will slightly smooth the image by removing high frequency content (eg: noise, edges) from the image.
  For this, I used the cv2 Function "cv2.GaussianBlur(input_image, (kernel_size, kernel_size), 0)" 
  Here,
  - "input_image" is the Input Image, which is the Image converted into Grayscale
  - "kernel_size" is the Gaussian kernel size. Here I used a value of 3 as optimum value between noise removal and retaining the required details in image. 
  The output image of this step on the example image "solidWhiteRight.jpg" is as below. 

####  3. Edge Detection using Canny Edge Detection
  Canny Edge Detection is a popular multi-stage algorithm for edge detection.  
  For this I used the OpenCV Function cv2.Canny(input_image, low_threshold, high_threshold), in which, all required stages are combined into one function.
  Here,
  - "input_image" is the Input Image, which is the Smoothened Image from previous step. 
  - The largest value "high_threshold", is used to find initial segments of strong edges. Here I used value of 150 as high_threshold. 
  - The smallest value between low_threshold and high_threshold is used for edge linking. Here I used value of 50 as low_threshold. 
  The output image of this step on the example image "solidWhiteRight.jpg" is as below. 

####  4. Apply a Region of Interest (ROI) Mask
This step is about cropping out the original image to the Region of Interest which is, only the portion with the Lane Lines on the Road.
To actually do the cropping of the image, I Prepared a Mask using the cv2 Function cv2.fillPoly(mask, vertices, ignore_mask_color) where,
* "mask" is a Blank Mask to start with.
* "vertices" is the Vertices of the Polygon which specifies the Region of Interest.  For the vertices, I used, % of Image Size rather than Absolute Points so as to handle Images of Any Size. Considering the origin point (0, 0) is upper left corner of the image, I used:
  * 0.59 of y Size from 0 (Image Height from 0) for fixing Top Vertices of ROI
  * 0.48 of x Size from 0 (Image Length from 0) for fixing Left Top Vertex of ROI
  * 0.54 of x Size from 0 (Image Length from 0) for fixing Right Top Vertex of ROI and
  * Image Bottom Left & Right Corners (Full Image Height from 0) for fixing Bottom Vertices of ROI
I found these Points to be Optimum for fixing the ROI that includes both the Left & Right Lane Lines up to a Distance up to Horizon.
* "ignore_mask_color" is the Fill Color for filling pixels inside the polygon defined by "vertices" . I used 255 to Fill White.
I then Applied the Mask to extract the Image only where, the Mask Pixels are Non-Zero. Using the cv2 Function cv2.bitwise_and(img, mask) (i.e., Input Image '&' ROI Mask), where img is the Edge detected Image and mask is the Mask that I prepared above.

The output image of this step on the example image "solidWhiteRight.jpg" and the vertices of the Polygon used for Region of Interest are as below.

####  5. Detect Lines Hough Lines as Line Segments
  in this step, the Line Segments are detected using the cv2 Function cv2.HoughLinesP(img, rho, theta, threshold, np.array([]), minLineLength=min_line_len, maxLineGap=max_line_gap),
  Here,
  - "img" is the Edge detected & ROI Mask applied Image
  - "rho" and "theta" respectively are, the Distance Resolution in Pixels & Angular Resolution in Radians of the Hough Accumulator,
  - "threshold" is the Hough Accumulator Grid Voting Threshold, i.e., Minimum No. of Points that are required in a Line to deduce a Line,
  - "np.array([])" is an Empty List,
  - "minLineLength" is the Minimum Length of Line Segments and
  - "maxLineGapis" the Maximum allowed Gap between Points on the Same Line to Connect them.
  I found following as Optimum Values for Detection of Line Segments in various Usecases: 
    - rho = 1 Pixel,
    - theta = 1 Degree (π/180 Radians),
    - threshold = 32,
    - minLineLength = 6 and
    - maxLineGap = 2
  Especially, minLineLength = 6 ensures that Small Stray Elements like Lane Paint Distortions, Shadows, etc. are Filtered-out and maxLineGap = 2 ensures that only Lane Line Segments in the Same Line are connected and Not the Lane Paint Spills or Patches by the Sides.
  
I then used the Helper Function draw_lines(img, lines, color=[255, 0, 0], thickness=4)
in its Simplest Form to simply Draw the Line Segments detected by cv2.HoughLinesP().
Here, img is a Blank Image to draw the Line Segments and
lines are the Line Segments detected by cv2.HoughLinesP().
This Helper Function in-turn uses the cv2 Function cv2.line(img, (x1, y1), (x2, y2), color, thickness) to Draw the Lines,
where (x1, y1) and (x2, y2) are the Co-ordinates of the Line Segments and
color and thickness respectively are the Color and the Thickness of the Lines to be drawn.

####  6. Draw Single Solid Line per lane by filtering, Averaging and Extrapolation

####  7. Overlay the Detected Line Segments / Lines on Original Input Image so as to track the Lane Lines

First, I converted the images to grayscale, then I .... 
4. Apply a Region of Interest (ROI) Mask 
    by Draw a filled polygon and
    show the area that is the mask

First, I converted the images to grayscale, then I .... 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]

![][image1]
![](image.png) 

####  5. Detect Lines Hough Lines



One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
