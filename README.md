# Vehicle Detection
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

---


The goals of this project are the following:

Broad steps:
* Load the training images (cars and non-cars)
* Include functions for spatial binning, color transformation, computing a color histogram, computing HOGs (Histogram of Oriented Gradient)
* Include functions to extract features from an image using functions from the previous steps
* Include a function to give window a list of windows (defined by top left and bottom right coordinates) from an image using a sliding window technique.
* Include a function to iterate through windows output from function in previous step and return windows that are properly classified as cars using a Linear SVM classifier
* Run my pipeline on a video stream (starting with the test_video.mp4 and later implementing on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.

The Jupyter notebook + code comments do a good job of explaining the individual steps.


[//]: # (Image References)
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.png
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.png
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.   

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the code cell X of the Jupyter notebook.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `car` and `noncar` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`pixels_per_cell`).  I grabbed the same random image from the previous step and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here are examples of changing the color space between RGB and YCrCb and the pixels per cell between 8 and 16. As you would expect the color space changes which regions light up, whereas the shapes get 'longer' with a higher pixels per cell value, presumably because you are taking the net from a larger sample of pixels.


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I played around with different HOG channels and color spaces and spatial size until I got a high accuracy. In practice you would want to get more exhaustive with the parameters you try.


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using LinearSVC imported from sklearn.svm. This is on line X. You train this using svc.fit and passing in your extracted features for the first argument and the y values gleaned from the folder you pulled the images from.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I followed the recommendation that we shouldn't have thousands of search windows. I discarded the top half (cars won't be found in the sky or in trees) as suggested in the lessons. Additionally, I looked at typical images and determined the sizes we would expect at different locations within the images.

At first I used the sliding window functions provided in the lessons, but then I used a trick for searching different window sizes. Rather than taking a window (i.e. 128 x 128) and downsizing it, it scales the entire image, applies HOG to said image, then subsample the array. The net effect is to sample in different window sizes.

Here's a cool visualization that I grabbed from the readme template that illustrates just how many windows can be drawn on a picture. The sizes/locations of windows that can be drawn on an image is infinite!

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

I cut out the top region from the sliding window search to remove false positives. The stack of binned colors, histogram of colors and 'ALL' HOG feature extraction proved to be a good fit. To improve performance I limited the window search to the locations we would expect to find cars and given appropriately sized windows.

 Here are some example images:

![alt text][image4]


### Here are six example images and their corresponding heatmaps and resultant boxes:

![alt text][image5]

More images can be seen in the jupyter notebook.

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

To address the false positives problem I limited my sliding window search to specific regions in the bottom half of the images. Next I constructed a heatmap from labels using the `scipy.ndimage.measurements.label()` function. This allows us to combine overlapping bounding boxes.  For video creation I create a heatmap from the 5 most recent frames then do thresholding on that.






---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Where I spent the most time on the project was refactoring the code from the lessons to be usable for both generating the video and for testing. Reducing the false positives posed a bit tricky. Good configuration for the locations of the sliding windows and their sizes can be helpful, but it's difficult to predict what we'll see in the future without looking at more video. Additionally, the thresholding and the number of frames to threshold over involved a lot of trial and error.

I have a sense that more windows may be helpful, but the trade off is that more windows searched means more times to process each frame i.e. we may not be able to get real-time vehicle detection even with powerful hardware.

My pipeline may fail with unknown cars. Augmenting the training data by obtaining more cars or by artificial forms of augmentation may help. Finally, I didn't exhaustively try to tweak all of the combinations of classification parameters. Surely I haven't attained the absolute optimal.
