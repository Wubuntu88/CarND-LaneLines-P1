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

[region_filtered]: ./intermediate_result_images/3_region_filtered_canny.png "Region Filtered"

[hough_on_black]: ./intermediate_result_images/4_hough_lines_drawn_on_canvas.png "Hough On Black"

[hough_on_road]: ./intermediate_result_images/5_hough_lines_drawn_on_road.png "Hough On Road"

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

### 1) Convert to greyscale
My first step was to create a greyscale image.I did so with the following function:

greyscale_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)

![grayscale]

### 2) Apply Canny Edge Detection to Image
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

### 3) Apply Gaussian Smoothing to the Image processed with Canny Edge Detection.
in part 2, I chose to perform Gaussian smoothing before and after Canny Edge Detection.  I tried doing it without smoothing, with smoothing before canny, with smoothing after canny, and with smoothing before and after.  I noticed that applying smoothing after, and before and after offered a smooth image.  The smoothing after vs. smoothing before and after looked the same, so I chose the before and after solution.

I chose a kernel size of 5 for no good reason other than it was used in the course code.

Here is the image after performing smoothing before and after:

![canny_with_gaussian_smoothing]

### 4) Define a region of interest, and bitwise and the image with that region to exclude non-lane-line-items.
I defined a polygon region of interest that would contain the lane lines withing the polygon and exclude everything outside of the lane lines.  To choose the points, I took a ruler and made some rough estimations about what the bounding polygon should be.  The specific values can bee seen in the make_region_of_interest(image) function below.
Once the polygon is formed, the original image is masked to the polygon by a bitwise and method, so that only the region in the mask is kept.
```python
"""
Part4: make region of interest
"""
def make_region_of_interest(image):
    x_size = image.shape[1]
    y_size = image.shape[0]
    bottom_left = (x_size * 0.05, y_size)
    top_left = (x_size * .45, y_size * .6)
    top_right = (x_size * .55, y_size * .6)
    bottom_right = (x_size * 0.95, y_size)
    vertices = np.array([[bottom_left, top_left, top_right, bottom_right]],
                       dtype=np.int32)
    
    mask = np.zeros_like(image)   
    ignore_mask_color = 255
    
    cv2.fillPoly(mask, vertices, ignore_mask_color)
    masked_edges = cv2.bitwise_and(image, mask)
    return masked_edges
```
The result image is shown below:

![region_filtered]

The region filtered image is sent to the make_hough_lines() function

### 5) Calculate the hough lines.
Next I calculated the hough lines from the region filtered image.  This is performed in the make_hough_lines() function.  I was not so sure of what values the parameters of rho, theta, and the other threshold values should be.  I tried several combinations, staying close to what was shown during the course.
```python
"""
Part5: make hough lines
"""
# Define the Hough transform parameters
# Make a blank the same size as our image to draw on
def make_hough_lines(image):
    rho = 1 # distance resolution in pixels of the Hough grid
    theta = np.pi/180 # angular resolution in radians of the Hough grid
    threshold = 3     # minimum number of votes (intersections in Hough grid cell)
    min_line_len = 3 #minimum number of pixels making up a line
    max_line_gap = 3    # maximum gap in pixels between connectable line segments

    hough_lines = cv2.HoughLinesP(image, rho, theta, threshold, np.array([]), minLineLength=min_line_len, maxLineGap=max_line_gap)
    return hough_lines
```
The Image of the Hough lines on a black canvas:

![hough_on_black]

The Image of the Hough lines on the road:

![hough_on_road]




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
