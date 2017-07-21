# **Finding Lane Lines on the Road** 

## Project 1: Detecting Lane Lines Writeup

## Author: William Gillespie

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

[grayscale]: ./intermediate_result_images/1_grayscale.jpg "MyGrayscaleImage"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline is in my process_image2() method in my P1 jupyter notebook.
There are 6 steps in my pipeline:
1) Convert to greyscale
2) Apply Canny Edge detection to image.
3) Apply Gaussian smoothing to the image (not explicitly listed in the process_image2() method; applied in step 2's method).
4) Define a region of interest, and bitwise and the image with that region to exclude non-lane-line-items.
5) Calculate the hough lines.
6) Draw lines on image by using the hould lines as input.

## Detailed Description of Each Step

# 1) Convert to greyscale
My first step was to create a greyscale image.
I did so with the following function:
greyscale_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
![alt text][grayscale]

--
My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I .... 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
