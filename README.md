# Vehicle Detection and Tracking


[//]: # (Image References)

[image1]: ./carUndistort.png "Undistorted"
[image2]: ./chessboard.png "Undistorted"
[image3]: ./laneBinary.png "Binary"
[image4]: ./result.png "Result"
[image5]: ./birdsEye.png "Top Down View"

## Goals of the project
The goals / steps of this project are the following:

1. Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images
2. Use a color transformation then append color histograms and color features to the HOG feature vector.
3. Create features which are randomized and normalized.
4. Train a classifier (Linear SVM)
5. Use sliding window technique to look for cars using the previously trained classifier.
6. Eliminate false positives by using a heat map which only considers multiple detections as positives.
7. Approximate the size of bounding box for the detected cars.
8. Draw the boxes.

### We will go by the rubric to answer all the points systematically

## HOG
### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

I created a function called as get_hog_features which takes a normal image and then converts it into a HOG image with feature vectors that are useful for detection. The codes for this were provided by udacity. Then I applied this function to different color spaces to see which one has the best information for our work.

For example : I used YUV color space with the following HOG parameters: orientations=8, pixels_per_cell=(8, 8) and cells_per_block=(2, 2).

### 2. Explain how you settled on your final choice of HOG parameters.
I tried different permutations and combinations of parameters but the best accuracy(SVM Training) was found with the ones that I chose to begin with and here are those parameters:(orientations=8, pixels_per_cell=(8, 8), cells_per_block=(2, 2) and YUV colorspace.)



#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).
I normalized both color and HOG features beforehand as a preparatory step for the classification. An example of that is shown below:



I then used the previously normalized features to train on a linear SVM classifier(Udacity model) and the accuracy came out to be more than 98% so it looked like a good choice of model to me.

## Sliding Window Search


### 1. Describe how (and identify where in your code) you implemented a sliding window search. How did you decide what scales to search and how much to overlap windows?

To detect the cars on the lane, I created a find_cars() function which had the following mechanism:
1. Select the regions where chances of cars are there.
2. Convert the entire colorspace to YUV.
3. Get all the HOG features for the entire area.
4. Get HOG features using a sliding window and use that particular feature on the classifier to find if it's a car or not.

Then I tried playing with different scales for my window size to optimize for the best window size. Here are some example of different levels of scale:

### 2. Show some examples of test images to demonstrate how your pipeline is working. What did you do to optimize the performance of your classifier?

In my pipeline I used the optimized scale values ([1, 1.25, 1.5, 1.75, 2, 2.25, 2.5, 3]) for searching. Plus I used YUV 3 channel HOG features, color histograms and spatially binned color to get a good result.
Finally to get rid of false positives and multiple detections, I utilized heat map and then drew a rectangle of appropriate size in that region. Below is an example of the end result without using the heatmap which also contains too many detections for the same car.

## Video Implementation

### 1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I used different functions in a combination to finally implement a heatmap which would serve the purpose of rejecting the false positives. The way this was done was by recording all the positive detections and then creating a heatmap of those detections and then using a threshold to classify that as a positive detection and thus putting a box around it.

An example of heatmap is shown below:


I also utilized class to keep track of bounding boxes or detections in the previous frames(last 5 frames) and then clubbing all these boxes to draw a heatmap would allow me to set a higher threshold limit which would reduce the false positives even further.

## Discussion

### 1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

One common problem I found was that the guardrail area produced a lot of false positives as they were being falsely classified as cars. This problem could be tackled by providing the images of these areas specifically with them being labeled as non-cars.

Apart from that one other issue I faced is that in some of the frames one of my car is not being boxed which means that it was seen as a false positive due to lack of intensity of the heatmap which can be fixed by lowering the threshold value but at the same time it will compromise with the issue of real false positives being ignored so an optimum  value was used.

Lastly, while using class to find enough number of detections to pass as a threshold we need at least 3 or more frames of data as below that the threshold limit won't classify them as a positive thus in the first 3 or 4 frames the cars are not being classified positives as enough data isn't present till that time. This may be fixed by risking false positives for the first few frames by setting up a lesser threshold value for the first 5 frames.
