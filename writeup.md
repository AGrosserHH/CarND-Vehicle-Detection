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
[image1]: ./examples/car_not_car.png
[image2]: ./examples/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png


[image21]: ./output_images/test1.jpg
[image22]: ./output_images/test2.jpg
[image23]: ./output_images/test3.jpg
[image24]: ./output_images/test4.jpg
[image25]: ./output_images/test5.jpg
[image26]: ./output_images/test6.jpg
[video1]: ./project_video.mp4

###Writeup / README

####1. Provide a Writeup / README
You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the 10th code cell of the IPython notebook 'Vehicle Detection final'.  

In the 6th and 7th cell of the IPython notebook I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes :

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like. Moreover, I tried to augment additional data with data from the GTI data base. But as the classifier did well without the data and due to memory restrictions I limited the data base to the original data provided by Udacity.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][image2]

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and the following were chosen as the final values of the HOG parameters:

* orientations: 8. This is the number of orientation bins. I try several paramemters up to 16 but the accuracy of the classifier did not improve much.
* pixels_per_cell: 8. I experimented with this value as well, but mostly by trial and error and looking at the HOG output images.
* cells_per_block: (2, 2)

Besides HOG features, spatial binning is performed to extract features. The function bin_spatial at cell 2 is used to first resize the image to smaller dimensions using the openCV function resize and then return a single dimensional feature vector from this array using the ravel function.

Another method used to extract features is the color histogram technique. The method color_hist in cell 2 is used to extract color histogram features and return it as a single feature vector using the np.concatenate method.

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using

* extract_features method in cell In [5] which extracts all features from a list of images including the HOG features, the spatially binned features and the histogram of colors features.

* In cell 10 features are extracted for both car and non-car images using the extract_features described above.

* In cell 12 the features are scaled to zero mean and unit variance using the sklearn.preprocessing.StandardScaler.

* in cell 14 the data is split into training and test data

+ A Support Vector Classifier is created in cell 15. 

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

* In the function draw_image in cell 26, a sliding window approach has been employed by using windows of different scales. The scales selected were (64,64) and (96,96). These particular sizes were chosen as they allowed the detection of smaller as well as larger cars in the image but also did not adversely affect performance. The overlapping chosen was 0.6.

* The function add_heat in cell 21 is used to add values to a heatmap (used for detecting vehicle boxes).

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:


---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video. From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle. 
In order to have a more robust version of the algoithms I am trying to save the boxes from 10 images. If a false positive appears on one image but not on the next image then it was a false positive and disappears.  

### Here's an example result the bounding boxed from 6 example images:
![alt text][image21]
![alt text][image22]
![alt text][image23]
![alt text][image24]
![alt text][image25]
![alt text][image26]


---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline might fail once
* pedastrians appear in the windows
* the brightness of the video varies (different sunshine/ shadow) since there is no further image augmentation done 

In order to make it more robust it would be necessary to split the non-car class into pedastrians or bikes. For making it more robust with respect to light it would be ncessary to apply some changes of the images.
