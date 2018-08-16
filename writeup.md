# **Finding Lane Lines on the Road** 

## Writeup

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"
[region_of_interest]: ./writeup/region_of_interest.png "Regions of interest"
[canny_left]: ./writeup/canny_left.png "Edges on left side"
[hough_all_lines]: ./writeup/hough_all_lines.png "Hough lines"
[final_lines]: ./writeup/final_lines.png "Final lines"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

##### Step 1 - Gaussian blur
This step is pretty straightforward, I'm using kernel_size=7. This is needed to reduce noise for edge detection.

##### Step 2 - Region of interest & Canny
There are 2 regions of interest - area around the left line and area around the right line as shown on the image below:
![alt text][region_of_interest]
I run edge detection twice with different parameters - parameters for edge detection on each side(left/right) depend on the average intensity on that side. That potentially can help if one side of the road is in a shade, or has some objects on it. After edge detection I get 2 full-size edge images - 1 for each side and then apply masks to them accordingly, so my output on this step is 2 full-sized images - one contains edges within the left region of interest, second - within right region. Below is an example of an output for left side:

![alt text][canny_left]

##### Step 3 - Hough lines
I'm using the same idea of separating left and right regions for finding hough lines. Additionally, I'm adjusting the parameters for HoughLinesP using the number of edge-pixels on an image. The idea is that the more edges there are on an image, the higher the threshold should be and the lower the minLinesGap can be used. That helps to avoid finding too many lines in the third video on frames with tree shadow on the road.

![alt text][hough_all_lines]

##### Step 4 - Filtering and averaging lines
The last step is to get one final line for each side of the road. To do this, I'm calculating additional information from the lines - their length, and "a" and "b" parameters for the y=ax+b definition. I filter the lines by their slopes, leaving only lines that have their slope within a range of \[25, 50\](or  \[-25, -50\]) degrees, to reduce the noise. That gives an array of lines of different lengths. 
Then I average them using the lengths as weights, which gives me just one line(for each side). The last step is to extrapolate the lines from the middle to the bottom - I'm using the "a" and "b" parameters to get the edge points.

![alt text][final_lines]

### 2. Identify potential shortcomings with your current pipeline

1. There are quite a few parameters in the pipeline, so it might be overfitting the test videos.
2. Since I'm using very specific regions of interests and filtering the slopes of the lines, the pipeline will not detect lines in the middle (e.g. when the car changes lanes), sharp turns probably will not be handled well, also it's sensitive to the position of camera in the car.
3. I'm averaging the lines, so if there are 4 yellow lines on the left, they will be averaged to their middle, though it would make more sense to detect the right-most line
4. Some parameters are hardcoded, so the pipeline can break if the resolution of an input video changes
5. Untested cases: nighttime, snow, rain, heavy traffic, fog, and probably many more
6. There are some parts in the pipeline that can be optimized, e.g. Canny edge detection part is doing some extra work


### 3. Suggest possible improvements to your pipeline

1. I guess, using a CNN for that can give a boost in accuracy compared to manually set rules.
2. The parameters can probably be fine-tuned to improve accuracy.
3. We know that the lines are usually white or yellow - using this information should help to prioritize the actual lines over the noise.
4. Haven't tried converting to grey-scale image before doing edge detection - might be worth to look into it
5. Extend the region of interest to make the pipeline more general, but that requires improving the accuracy first
