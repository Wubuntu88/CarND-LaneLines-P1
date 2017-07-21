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

[grayscale]: ./intermediate_result_images/1_grayscale.png "MyGrayscaleImage"

[grayscale]: ./intermediate_result_images/1_grayscale.png "MyGrayscaleImage"

[canny]: ./gaussian_blur_images/no_blur.png "Canny"

[canny_with_gaussian_smoothing]: ./intermediate_result_images/2_canny.png "Canny With Smoothing"

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
My first step was to create a greyscale image.I did so with the following function:

greyscale_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

![grayscale]

# 2) Apply Canny Edge Detection to Image
My next step was to apply canny edge detection to the image.
This was done in my make_canny_image(image) function

```python

"""
Part 2: make canny images & apply gaussian blur
"""
def make_canny_image(image):
    low_threshold = 255 *  0.3
    high_threshold = 255 * 0.6
    kernel_size = 5
    image = cv2.GaussianBlur(image, (kernel_size, kernel_size), 0)
    image = cv2.Canny(image, low_threshold, high_threshold)
    image = cv2.GaussianBlur(image, (kernel_size, kernel_size), 0)
    return image
```
I chose threshold values of 76.5 and 153.0, which are 255 * .3 and 255 * .6.
These are close to the values of 50 and 150, which are the values recommended in the course.

The Image of my Canny Edge Detection:
![canny]

# 3) Apply Gaussian Smoothing to the Image processed with Canny Edge Detection.
in part 2, I chose to perform Gaussian smoothing before and after Canny Edge Detection.  I tried doing it without smoothing, with smoothing before canny, with smoothing after canny, and with smoothing before and after.  I noticed that applying smoothing after, and before and after offered a smooth image.  The smoothing after vs. smoothing before and after looked the same, so I chose the before and after solution.

I chose a kernel size of 5 for no good reason other than it was used in the course code.

Here is the image after smoothing before and after:
![canny_with_gaussian_smoothing]

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
