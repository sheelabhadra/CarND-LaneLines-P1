# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Document the steps used in the pipeline in a written report


[//]: # (Image References)

[image1]: ./examples/writeup_images/solidWhiteRight.jpg "Original image"
[image2]: ./examples/writeup_images/grayscale.jpg "Grayscale"
[image3]: ./examples/writeup_images/Gaussian_blur.jpg "Gaussian blur"
[image4]: ./examples/writeup_images/Canny_edges.jpg "Canny edges"
[image5]: ./examples/writeup_images/masked_image.jpg "Canny edges in the region of interest"
[image6]: ./examples/writeup_images/line_edges.jpg "Hough line segments"
[image7]: ./examples/writeup_images/lane_lines.jpg "Lane Lines"


---

### Reflection

### 1. Description of the pipeline

My pipeline consisted of 5 steps. For illustration I have used an example image shown below to show the effect of all the operations in my pipeline.
![alt text][image1]

**1. Color to grayscale conversion**

The color image was converted to grayscale to make the computation easier and to detect lane lines of any color.
![alt text][image2]

**2. Gaussian blur**

Gaussian blur is applied on the grayscaled image to remove noise and spurious gradients. Gradients are calulated by the change in the intensity of gray value between adjacent pixels. More information about OpenCV's Gaussian blur can be found [here](http://docs.opencv.org/2.4/modules/imgproc/doc/filtering.html?highlight=gaussianblur#gaussianblur). I used a kernel size of 3 for Gaussian blur since the Canny edge detector (described next) already includes Gaussian smoothing.
![alt text][image3]

**3. Canny transform**

[Canny edge detection](http://docs.opencv.org/trunk/da/d22/tutorial_py_canny.html) is applied to the Gaussian-blurred image. This allows us to detect edges on the basis of change in the gradient intensity and direction at every pixel. The algorithm will first detect strong edge (strong gradient) pixels above the high\_threshold, and reject pixels below the low\_threshold. Next, pixels with values between the low\_threshold and high\_threshold will be included as long as they are connected to strong edges. The output edges is a binary image with white pixels tracing out the detected edges and black everywhere else. I used a low\_threshold value of 50 and a high\_threshold value of 150.
![alt text][image4]

**4. Region of interest mask**

A mask is drawn to select only the relevant region in which there is a possibility of lane lines being present. This would be generally a quadrilateral shaped region in the bottom half of the image. The mask selects the white colors (edges) from the image obtained after applying Canny transformation using a bitwise and operation.
![alt text][image5]

**5. Hough transform**

[Hough transform](https://alyssaq.github.io/2014/understanding-hough-transform/) is applied on the the image obtained after applying the region of interest mask. The probabilisitc Hough transform is used to construct line segments using the edge points detected in the previous step. More information about using Hough transform can be found [here](http://docs.opencv.org/3.0-beta/doc/py_tutorials/py_imgproc/py_houghlines/py_houghlines.html). The edges are segments colored in red. The coordinates of the end points of the line segments are returned by the Hough transform function. 
![alt text][image6]

**6. Drawing lane lines**

The line segments obtained in the previous step are extrapolated to draw straight red colored lines along the lane lines. This is accomplished by modifying the draw\_lines() functon in the following ways:

* The slope of each line segment is calculated using the formula (y2-y1)/(x2-x1).

* The slope of the lane lines lie within a particular range and this information is helpul in rejecting spurious line segments. I used a range of (0.5, 2.5) for the left lane line and (-2.5,-0.5) for the right lane line. Line segments with slope values outside of these range were ignored. The coordinates belonging to the left lane line were put into arrays left\_lane\_x, left\_lane\_y and the coordinates belonging to the right lane line were put into arrays right\_lane\_x, right\_lane\_y.

* The np.polyfit() function is helpful in constructing a line given a set of points. More information on this function can be found [here](https://docs.scipy.org/doc/numpy/reference/generated/numpy.polyfit.html). The degree was set to 1 and was applied to the arrays left\_lane\_x, left\_lane\_y to find the slope and intercept values of the left lane line and to the arrays right right\_lane\_x, right\_lane\_y to find the slope and intercept values of the right lane line.

* The points of intersection of the 2 lane lines with the region of interest were calculated and the lane lines were drawn only inside the region of interest.

![alt text][image7]

### 2. Potential shortcomings with my current pipeline

* If the intensity of the lane lines varies a lot in the video due to different angles of the sun, then my pipeline does not perform well.
* If there are portions of the entire road that are white in color then the lane lines are not identified correctly.
* The lane lines are not stable at times when a few outlier points are detected.
* Due to the shortcoming mentioned above my pipeline performs awfully on the challenge.mp4 video.
* My pipeline will not work on roads that do not contain lane markings 


### 3. Suggest possible improvements to your pipeline

* A possible improvement would be to track the yellow lane line by using color masking.
* The lane lines can be smoothed by keeping track of the lane lines in the previous video frames.
* I am trying to make my pipeline more robust by trying to make it work on the challenge.mp4 video.
