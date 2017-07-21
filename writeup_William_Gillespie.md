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

[extrapolated_lines_on_black]: ./intermediate_result_images/6_extrapolated_lines_on_canvas.png "estrap_black"

[extrapolated_lines_on_road]: ./intermediate_result_images/7_extrapolated_lines_on_road.png "estrap_black"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline is in my process_image2() method in my P1 jupyter notebook.
There are 7 steps in my pipeline:
1) Convert to greyscale
2) Apply Canny Edge detection to image.
3) Apply Gaussian smoothing to the image (not explicitly listed in the process_image2() method; applied in step 2's method).
4) Define a region of interest, and bitwise and the image with that region to exclude non-lane-line-items.
5) Calculate the hough lines.
6) Draw lines on image by using the hould lines as input.
7) Create an image with the lines on the road.

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

### 6) Draw lines on image by using the hough lines as input.

In this step, I use the hough lines to create two lines, each representing a lane line.  The attempt is to cut out the noise of all of the hough lines.

I modified the function draw_lines() that was provided by the course.

To take out the noise, I wanted to filter the hough lines.  I checked the number of hough lines in an image, and there were over 300.  I thought that many of these were probably noise, so I developed two methods of filtering:
#### a) Filter by Slope
I disregarded lines that were less than a slope of 1 (45 degrees) and greater than a slope of -1.  I though that these would be noisy lines because lane lines would not have these slopes if one were driving straight in a lane.
#### b) Filter by the location of the two x points of the line
If the slope was greater than zero, I accepted a line if either of the x coordinates were greater than the x midpoint.  Basically, the slope of the line should be consistent with which side of the screen the line is on.  If it is not consistent, I considered it as noise and filtered out that point.

NOTE: please note that my filter considered BOTH criteria; the line had to pass both to be considered.

In one run, I cut down over 300 lines to about 8.

After filtering the lines, I fit a line to each set of points (left lane x y coords and right lane x y coords).
I determined where the y coordinates should be and calculated the position of the x coordinates using the line fit parameters (slope and intercept).  All of this was to calculate two lines, and then draw them on a canvas using the cv2.line() method.

```python
def draw_lines(img, lines, color=[255, 0, 0], thickness=5):
    slope_threshold = .5
    
    y_size = img.shape[0]
    x_size = img.shape[1]
    x_midpoint = x_size / 2
    
    left_xs = []
    left_ys = []
    right_xs = []
    right_ys = []
    
    for line in lines:
        unwrapped_line = line[0]
        x1,y1,x2,y2 = unwrapped_line # unpack tuple
        slope = get_slope_of_line(unwrapped_line)

        # if the slope is greater than 45 degrees (slope of 1) or less than negative 45 degrees, 
        # we will use those points to fit the line
        if slope > slope_threshold or slope < -slope_threshold: 
            if slope > 0 and (x1 > x_midpoint or x2 > x_midpoint): # right line
                right_xs.extend((x1, x2))
                right_ys.extend((y1, y2))
            elif slope < 0 and (x1 < x_midpoint or x2 < x_midpoint): # left line
                left_xs.extend((x1, x2))
                left_ys.extend((y1, y2))
        
    right_lane_slope, right_lane_intercept = np.polyfit(np.array(right_xs), np.array(right_ys), 1)
    left_lane_slope, left_lane_intercept = np.polyfit(np.array(left_xs), np.array(left_ys), 1)
    
    # left lane points
    top_y_left_lane = int(y_size * 0.6)
    top_x_left_lane = int( (top_y_left_lane - left_lane_intercept) / left_lane_slope )
    
    bottom_y_left_lane = int( y_size )# max(left_ys)
    bottom_x_left_lane = int( (bottom_y_left_lane - left_lane_intercept) / left_lane_slope )
    
    # right lane points
    top_y_right_lane = int( y_size * 0.6 )
    top_x_right_lane = int( (top_y_right_lane - right_lane_intercept) / right_lane_slope )
    
    bottom_y_right_lane = int( y_size )
    bottom_x_right_lane = int( (bottom_y_right_lane - right_lane_intercept) / right_lane_slope )
    
    # make lane line tuples
    left_lane_top_point = (top_x_left_lane, top_y_left_lane)
    left_lane_bottom_point = (bottom_x_left_lane, bottom_y_left_lane)
    
    right_lane_top_point = (top_x_right_lane, top_y_right_lane)
    right_lane_bottom_point = (bottom_x_right_lane, bottom_y_right_lane)
    
    cv2.line(img, left_lane_top_point, left_lane_bottom_point, color, thickness)
    cv2.line(img, right_lane_top_point, right_lane_bottom_point, color, thickness)
```

The lines on a black canvas:

![extrapolated_lines_on_black]

The lines on the road:

![extrapolated_lines_on_road]


### 7) Create an image with the lines on the road.

The last step was to overlay the lienes on the road image.  I did so with the weighted_img() method provided by the course.

```python
def weighted_img(img, initial_img, α=0.8, β=1., λ=0.):
    return cv2.addWeighted(initial_img, α, img, β, λ)
```

That concludes the presentation of my pipeline.  The entire pipeline put together is shown below:

```python
def process_image2(image):
    # NOTE: The output you return should be a color image (3 channel) for processing video below
    # TODO: put your pipeline here,
    # you should return the final output (image where lines are drawn on lanes)
    greyscale_image = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    canny_image = make_canny_image(greyscale_image)
    region_filtered_image = make_region_of_interest(canny_image)
    hough_lines = make_hough_lines(region_filtered_image)
    line_img = np.zeros((image.shape[0], image.shape[1], 3), dtype=np.uint8)
    draw_lines(line_img, hough_lines)
    result = weighted_img(line_img, image)
    return result
```

### 2. Identify potential shortcomings with your current pipeline

One shortcomming is that in the video, the line sometimes changes slope suddently for a frame.  I believe this is due to some noisy hough lines not being filtered out by the draw_lines method.

Another problem is on the optional challenge.  My process_image2 method drew very erratic lines, although they were roughly in the ball park.  I belive some of the problem was the the region of interest was shorter because the car was turning, and the relevant information was not perfectly in the region of interest.  I also think some of the additional line adornments, especially on the yellow lines, may have been causing some issues.

### 3. Suggest possible improvements to your pipeline

One way to solve the problem of the noisy houghlines causing the slope to change rapidly is to try to tweak the filter the eliminates the noisy hough lines.  I could try adjusting the slope limit at which lines are rejected, or tweak the x point requirements for filtering.

Another method of solving the line's slope rapid changes would be to keep track of that lane's slope n frames back.  If the slope jumps too suddently, we can just use the previous frame's slope, or an average of the last n frames' slope.  N could be anywhere from 1 to 10, depending on how fast frames are coming in.

To fix the region of interest issue in the optional challenge, we could check to see if the slope is totally bogus (lets say close to zero or very high (9+).  If this is the case we can lower the top y coordinates of the region of interest and recompute the slope.  We could get several images of a car going down a curvy lane and see how far we might have to bring the y coordinates down.
