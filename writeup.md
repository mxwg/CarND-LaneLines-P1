# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

[//]: # (Image References)

[pipeline]: ./results/solid_white_curve.png "My pipeline"

---

### Reflection

### 1. My pipeline

The image shows examples for all steps of the pipeline, which are explained in further detail below.

![Pipeline overview][pipeline]

1. **Preprocessing**

The preprocessing consists of converting the original image to grayscale and blurring it with a Gaussian filter, making the image suitable for line extraction.

2. **Canny filtering**

A Canny filter with experimentally chosen parameters is run on the preprocessed image. 
The aim here is to extract as much as possible of the actual line markings, but not too much other clutter.

3. **Masking**

The Canny edges are overlaid with a mask, excluding more than half of the image -- the top portion and everything to the sides of where the line markings are expected to appear.

4. **Hough transform**

A Hough transform is then applied to the masked image.
Here, as with the Canny filter, the parameters were chosen experimentally to retain as much structure as possible from the actual line markings while removing other clutter.
This was achieved by requiring somewhat long minimal line lengths while also allowing some line gaps.

5. **Single line fitting**

The last step of the pipeline is implemented as a modification to the `draw_lines()` function.
The basic idea is to collect all Hough lines corresponding to the left and right lane markings, respectively.
Then, a single line representing these collections should be drawn onto the image.
To be able to draw such a line, we need to find the parameters of the line equation `y = m*x + b`, where `m` is the slope of the line and `b` its intercept.

For each line in the left and right line lists, we compute its slope and intercept.
In a next step, we find the average of all lines of one side by weighting the slopes and intercepts by the squared lenght of each line segment.
This gives proportionally more weight to longer lines, as we expect "true lane markings" to result in longer line segments than clutter not belonging to a lane marking.

These two "average lines" are finally drawn onto the image instead of the raw Hough lines.

### 2. Potential shortcomings

The pipeline as it stands works reasonably well on the first two video sequences, but has more trouble with the challenge video.
Right now, it excludes some spurious lines based on extreme slope.
Because it only estimates a single line, it is not well suited for rather steeper curves in the challenge video.

The pipeline currently also does no tracking of the lines it estimates.
If there are single images where not enough Hough lines are found, the pipeline is not able to output any estimate for this frame.

Through the missing tracking, there is also no smoothing of the successive estimates.
This is especially apparent in the challenge video, where in some cases, there are extreme changes in slope from one frame to the next, which cannot happen during a normal drive.

### 3. Possible improvements

Right now, every single line segment is used for estimation, only weighted by its length.
This should already favour lines towards the bottom of the picture, but we could also explicitly assign higher weight to these lines, as they are probably more accurately detectable than lines towards the middle of the frame that are far away from the car.

Another improvent could be reached by employing an algorithm like RANSAC to reject outliers and thus base the estimation only on the subset of the most accurate Hough lines.

Estimating a curve instead of a straight line should improve the fit especially in curves.

