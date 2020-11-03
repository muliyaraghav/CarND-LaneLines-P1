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
My pipeline draw_lines() function, consisted of 6 steps. 
  1. Convert Input Image to Grayscale
  2. Smoothing [Gaussian Blur](https://docs.opencv.org/master/d4/d13/tutorial_py_filtering.html)
  3. Edge Detection [Canny Edge Detection](https://docs.opencv.org/master/da/d22/tutorial_py_canny.html)
  4. Apply a Region of Interest (ROI) Mask  
  5. Detect Lines [Hough Lines](https://docs.opencv.org/3.4/d9/db0/tutorial_hough_lines.html) & [Draw Lines](https://docs.opencv.org/master/d6/d6e/group__imgproc__draw.html) as      Line Segments
  6. Draw Single Solid Line per lane by filtering, Averaging and Extrapolation
  7. Overlay the Detected Line Segments / Lines on Original Input Image so as to track the Lane Lines

First, I converted the images to grayscale, then I .... 
4. Apply a Region of Interest (ROI) Mask 
    by Draw a filled polygon and
    show the area that is the mask

First, I converted the images to grayscale, then I .... 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
