##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./example_images/car_not_car.png
[image2]: ./example_images/HOG_example.png
[image3]: ./example_images/sliding_windows.png
[image4]: ./example_images/sliding_window.png
[image5]: ./example_images/bboxes_and_heat.png
[image7]: ./example_images/output_bboxes.png
[video1]: ./project_video_out.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for extracting the HOG features from the training images is done in cell four of the IPython notebook utilising the `single_img_features()` function. This function is defined in cell 3. First off, the function allows you to use any color space and depending on certain flags, it will add additional features. Ultimately, the function uses the skimage.feature import from hog.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB` color space and HOG parameters of `orientations=6`, `pixels_per_cell=(8, 8)`, and `cells_per_block=(2, 2)`, and `hog_channel = 0`:


![alt text][image2]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters across RGB, HSV, LUV, HLS, and YUV testing the various `hog_channel` values that I felt would be most useful in detecting vehicles. I would try to change the `spatial_size` to various values such as (16, 16), (32, 32), (64, 64) but this felt best at (32, 32). The pixels per cell and cells per block worked best at 8 and 2. Ultimately I decided to use YCrCb as my `color_space` because it allowed me to properly utilize all HOG channels.

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using all the images from the big vehicles and non-vehicles dataset provided by Udacity. I split the dataset using `train_test_split()` in order to have a testing and a validating set. I split the data by 10%. This is all done in cell 5 of the notebook.

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I used the sliding window search in cell 6 but I defined the function in cell 3. I decided to search through every single window position at a constant scale. I used a window size of (96, 96) and an overlap of (0.5, 0.5). I decided on these through experimenting and testing and found good results using them. Originally I searched through every window but it was taking too long so I set a starting y position to ignore everything above the horizon.

![alt text][image3]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

In cell 9 of the notebook, I defined `find_cars()` and along with it I defined a heatmap. The heatmap iterates everytime a car is detected. In order to filter out false positives I set a threshold that the heatmap must reach in order to be determined to be a car. Any less and it is ignored. I also used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap. I assumed each blob corresponded to a certain vehicle and the bounding boxes which overlapped would merge to form one complete bounding box. This is evident in the video where the two cars would get one bounding box.  

Here's an example result showing the heatmap from the test images, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

Although they are not quite perfect, you can already note the bigger bounding boxes.


### Here the resulting bounding boxes are drawn onto the last frame in the video:
![alt text][image7]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I actually had a lot of issues resizing. It would give me runtime errors due to asserts even though I verified my image was loaded. My machine is stronger than most and so I was able to brute force checking my parameters for optimization. The pipeline is likely to fail in darker images. After a shadow was cast onto the project video, false positives surfaced from just that. In heavier weather conditions like a storm, the pipeline could go haywire. In order to make it more robust, I could finetune the parameters and mess around with the heatmap and it's threshold value to find an optimal level.

Also, in the output video, the vehicle detection occasionally brings up false positives or cars on the other side of the road. It is not for long but it comes up. On top of this, the vehicles it does detect, the bounding boxes appear very jittery. This issue could possibly be fixed by running the pipeline differently. Such as requiring a vehicle to appear in multiple frames.

