# **Finding Lane Lines on the Road** 

### Goals of the Project
The Goals / Steps of this Project are the following:
- Make a Pipeline (using tools like color selection, region of interest selection, grayscaling, Gaussian smoothing, Canny Edge Detection and Hough Tranform line detection) that Finds Lane Lines [Left & Right] on the Road from the given Input Images and Videos and then, track the detected Lane Lines with:
  - Stage-1: Line Segments
  - Stage-2: Solid Lines (Averaged/Extrapolated Single Line Segments drawn on to the Line Segments in the image)
- Reflect on the Project Work as a project report

### Reflection
### 1. Description of the Lane Detection Pipeline
My pipeline draw_lines() function, consisted of 7 steps. 
  1. Convert the Input Image into Grayscale
  2. Smoothing [Gaussian Blur](https://docs.opencv.org/master/d4/d13/tutorial_py_filtering.html)
  3. Edge Detection [Canny Edge Detection](https://docs.opencv.org/master/da/d22/tutorial_py_canny.html)
  4. Apply a Region of Interest (ROI) Mask  
  5. Detect Lines [Hough Lines](https://docs.opencv.org/3.4/d9/db0/tutorial_hough_lines.html) & [Draw Lines](https://docs.opencv.org/master/d6/d6e/group__imgproc__draw.html) as      Line Segments
  6. Draw Single Solid Line per lane by Pre-Processing, Averaging and Extrapolation
  7. Overlay the Detected Line Segments / Lines on Original Input Image so as to track the Lane Lines


using the example image solidWhiteRight.jpg shown below, I will explain the output of each step above.
solidWhiteRight.jpg : 

![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/test_images/solidWhiteRight.jpg "Image_Input")

####  1.1 Convert the Input Image into Grayscale
  In this step, I converted the Color Image to a 1-Channel Grayscale Image. We do this to be convenient to perform the Image Processing Algos in the upcoming Steps.
  For this I used the cv2 Function "cv2.cvtColor(input_image, flag)", which converts an image from one color space to another. 
  Here, 
  - "input_image" is the Input Image
  - "flag" determines the type of conversion, which is cv2.COLOR_BGR2GRAY for BGR -> Gray conversion
  The output image of this step on the example image "solidWhiteRight.jpg" is as below. 
  
  ![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/Img_outputs/gray.jpg "Image_Input")

####  1.2 Smoothing Gaussian Blur
  Edge detection results are easily affected by the noise in the image. So, before the edge detection, it is essential to filter out the noise to prevent false detection caused by it. This step will slightly smooth the image by removing high frequency content (eg: noise, edges) from the image.
  For this, I used the cv2 Function "cv2.GaussianBlur(input_image, (kernel_size, kernel_size), 0)" 
  Here,
  - "input_image" is the Input Image, which is the Image converted into Grayscale
  - "kernel_size" is the Gaussian kernel size. Here I used a value of 3 as optimum value between noise removal and retaining the required details in image. 
  The output image of this step on the example image "solidWhiteRight.jpg" is as below. 
  
  ![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/Img_outputs/Blur.jpg "Image_Input")

####  1.3 Edge Detection using Canny Edge Detection
Canny Edge Detection is a popular multi-stage algorithm for edge detection. Using this, we turn our image into pure black and white, where white represents the largest gradients (drastic changes in the connected pixel values) and they are the edges in the origical image. 
For this I used the OpenCV Function cv2.Canny(input_image, low_threshold, high_threshold), in which, all required stages are combined into one function.
Here,
- "input_image" is the Input Image, which is the Smoothened Image from previous step. 
- The largest value "high_threshold", is used to find initial segments of strong edges. Here I used value of 150 as high_threshold. 
- The smallest value between low_threshold and high_threshold is used for edge linking. Here I used value of 50 as low_threshold. 
The output image of this step on the example image "solidWhiteRight.jpg" is as below. 

![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/Img_outputs/edges.jpg "Image_Input")

####  1.4 Apply a Region of Interest (ROI) Mask
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

I then Applied the Mask to extract the Image with our Region of interest. For this, I used the cv2 Function cv2.bitwise_and(img, mask) (i.e., Input Image '&' ROI Mask), where img is the Edge detected Image and mask is the Mask that I prepared above.

The output image of this step on the example image "solidWhiteRight.jpg" and the vertices of the Polygon used for Region of Interest are as below.

![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/Img_outputs/edges_roi.jpg "Image_Input")

####  1.5 Detect Lines [Hough Lines] and draw the Line Segments
In this stage, we pass the processed image through the Hough transform.  In basic sense, the Hough transform will process the image and detect, where the pixels form lines. If the parameters are tuned correctly, this will return our lane lines. 

In this step, the Line Segments are detected using the cv2 Function cv2.HoughLinesP(img, rho, theta, threshold, np.array([]), minLineLength=min_line_len, maxLineGap=max_line_gap),
Here,
- "img" is the Edge detected & ROI Mask applied Image
- "rho" and "theta" respectively are, the Distance Resolution in Pixels & Angular Resolution in Radians of the Hough Accumulator,
- "threshold" is the Hough Accumulator Grid Voting Threshold, i.e., Minimum No. of Points that are required in a Line to deduce a Line,
- "np.array([])" is an Empty List,
- "minLineLength" is the Minimum Length of Line Segments and
- "maxLineGapis" the Maximum allowed Gap between Points on the Same Line to Connect them.

For deciding the values, I used Image length on x axis as reference  rather than absolute values, so as to handle Images of Any Size. I found following as optimum Values for Detection of Line Segments in various Usecases: 
- rho = 1 Pixel,
- theta = 1 Degree (π/180 Radians),
- minLineLength = 6 and
- maxLineGap = 2
- threshold = 32,
- minLineLength = xsize//16  Ex: approx 60 Pixels in above example image of 960 x 540 pixels
- maxLineGapis =  minLineLength//2  Ex: approx 30 Pixels
- threshold = h_min_line_len//4  Ex: approx 15 pixels

I then used the Helper Function draw_lines(img, lines, color=[255, 0, 0], thickness=4) in its simplest Form to simply Draw the Line Segments detected by cv2.HoughLinesP().
Here, 
- img is a Blank Image to draw the Line Segments and
- lines are the Line Segments detected by cv2.HoughLinesP().

This Helper Function in-turn uses the cv2 Function cv2.line(img, (x1, y1), (x2, y2), color, thickness) to Draw the Lines, where (x1, y1) and (x2, y2) are the Co-ordinates of the Line Segments and color and thickness respectively are the Color and the Thickness of the Lines to be drawn.

The output image of this step on the example image "solidWhiteRight.jpg" is as below. 

![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/Img_outputs/HoughLines(1).jpg "Image_Input")

####  1.6 Draw Single Solid Line per lane by Pre-Processing, Averaging and Extrapolation
Ideally, if we set our parameters correctly, the Hough transform should give the lane line. But, in reality, its difficult to get solid single line to full extent, when the lane lines are dashed, contains Paint Distortions, Shadows, etc. 
One particular issue I encountered is how to extrapolate the fragmented lane lines to show continuous lines, since the result from Hough Transfrom is a bunch of segments shown in the abvove image. The expected result is shown in the below image. 

![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/Img_outputs/SingleSolidLine(1).jpg "Image_Input")

A continuous line like Y = a*X + b consists of two things:
- slope of the line: coefficient “a”
- intercept of the line: coefficient “b”

By knowing above two parameters, we can find the top and bottom points on the line, so we can draw them on the image. Following picture shows the illustration of steps I followed for the left lane. 
![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/Img_outputs/ExtrapolationProcess1.jpg "Image_Input")

- **Pre-Processing**
we start with a bunch of segments from Hough Transform, and calculate slope for each segment and seperate them for left name and right lane. Positive slope means the segment  belongs to left lane, while negative slope means right lane . For this, I used polyfit() function from the NumPy package, which is "slope(a), intercept(b) = np.polyfit ( X, Y, 1)" where,
  The first parameter(X) is the first variable,
  The second parameter(Y) is the second variable,
  The third parameter is the degree of polynomial we wish to fit. Here for a linear function, we enter 1.

Meanwhile, we can find the minimum Y-coordinate of all points on both the lanes ( “minY” as shown with the dashed red line in the middle of image above).

- **Avg. Slope and Avg. Position**
In this step, we calculate the average value of slopes and average value of X and Y coordinate are calculated for both left and right lanes using the numpy.mean function. 
From the average slope, we get the coefficient “a” in Y = a*X + b.
From the average  of all points in the line, we get the average position (X, Y) of corresponding lane line as shown in the image above. 
Finally, using avg. slope “a” and average position, we can easily calculate the intercept “b” as: [b = avg_y - (a * avg_x)]
  
- **Determine Top and Bottom Position**
In order to draw lane line on the image, we need to know the start and end points.  
top_x = (minY - b)/average_slope (note: minY is the minimum Y value of all points)
bottom_x = (maxY -b)/a = (image.shape[0] -b)/average_slope

now, we can draw left lane line between points: (top_x, minY) and (bottom_x, maxY). In the same way, we can also draw the right line for which,  slope has opposite sign.


####  1.7 Overlay the Detected Line Segments / Lines on Original Input Image so as to track the Lane Lines
In this step, we overlay the above prepared Single Solid Line on the iginal Input Image so as to track the Lane Lines.  For this i used the OpenCV Image Blending function cv2 Function cv2.addWeighted(initial_img, α, img, β, γ) where, 
- initial_img is the Original Input Image,
- img is the image of extrapolated line segments
- α is the Weight of initial_img, β is the Weight of img and γ is a Scalar added to each Sum.

This Final Step's Output for the Example Input Image is:

 ![Example Input Image](https://github.com/muliyaraghav/CarND-LaneLines-P1/blob/master/Img_outputs/Final_img_out.jpg "Image_Input")
 
### 2. Potential Shortcomings of the Current Pipeline
following assumptions are made for the simplified design and are also limitations of current design. 
- The camera is mounted always in the same position with respect to the road. So, many of the parameters like ROI vertices are fixed or hard-coded. 
- There is always a visible white or yellow line on the road is assumed. 
- We don’t have considered any vehicle in front of us. So, If there any car cutting into our lane, edge detection may not work properly. Also, large vehicle in the front blocking the lane visibility may fail the algorithm. 
- Highway scenario considered is assumed with good weather conditions. 
- Level horizontal road is assumed. When the car makes turns or move uphill/downhill, lane line distortions may fail the algorithm designed.

### 3. Suggest possible improvements to your pipeline
One first improvement can be, considering the higher order polynomials for extrapolating and drawing Single Solid Line per lane.  Straight line uses Y=a*X+b to model the lane, while curves can use higher order polynomials like Y=a*X² + b*X+c.

Also adaptive ROI based on the Environment can be considered, especially to better handle different Road Elevations and Curves. 

[![Watch the video](https://youtu.be/rEtzoX_cBi0)
