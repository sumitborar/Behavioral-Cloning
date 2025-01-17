# Behaviorial Cloning Project

[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

Overview
---
**Behavioral Cloning Project**

The goals / steps of this project are the following:
* Use the simulator to collect data of good driving behavior
* Build, a convolution neural network in Keras that predicts steering angles from images
* Train and validate the model with a training and validation set
* Test that the model successfully drives around track one without leaving the road
* Summarize the results with a written report


#### 1. Submission includes all required files and can be used to run the simulator in autonomous mode

My project includes the following files:
* Behaviour Cloning.ipynb containing the code to create and train the model
* drive.py for driving the car in autonomous mode
* model.h5 containing a trained convolution neural network
* challenge_transfer_learning.ipynb - Code for challenge track
* README.md - summarizing the results
* output.mp4 - Video of autonomous run across track 1
* output-challenge.mp4 - Video of autonomous run across challenge track

#### 2. Submission includes functional code
Using the Udacity provided simulator and my drive.py file, the car can be driven autonomously around the track by executing
```sh
python drive.py model.h5
```

#### 3. Submission code is usable and readable

 Behaviour Cloning.ipynb contains the code for training and saving the convolution neural network. The file shows the pipeline I used for training and validating the model, and it contains comments to explain how the code works.

### Model Architecture and Training Strategy

#### 1. An appropriate model architecture has been employed

My model consists of a convolution neural network with 3x3 filter sizes and dense layers.  

The model includes ELU activation function to introduce nonlinearity instead of 'relu' as it is faster. Each image is cropped to remove horizon and car areas and resized to 80X160. Image is then normalized with 0 mean.

#### 2. Attempts to reduce overfitting in the model

As shown model contains dropout layers in order to reduce overfitting  

The model was trained and validated on different data sets to ensure that the model was not overfitting. (code line 58 ). The model was tested by running it through the simulator and ensuring that the vehicle could stay on the track.

#### 3. Model parameter tuning

I used adam optimizer with learning rate of 1e-4 and decay of 0. Code is located on line 63.

#### 4. Appropriate training data

I combined training data from muliple sources.
1. Udacity data
2. generated_data - Data generated by driving in center of the lane, along with a lap of revcovery.
3. x_box_training - Data generated by Patrick Kerns using xbox.

Center, Left and right images from above mentioned data sets were combined to form roughly 23k samples. This data was further subsampled based on steering angles to balance the training set.

Angles for left and right lane images were calculated assuming that the cameras are 1.2 meters away from the center and we are looking at distance of 25 meters in front. Also I utilize the fact that tan(very small angle) = very small angle and avoid finding arctan.

For details about how I created the training data, see the next section.

### Model Architecture and Training Strategy

#### 1. Solution Design Approach

The overall strategy for deriving a model architecture was to make sure model will learn to recognize and generalize different car positions on lane and predict correct steering angle.

My first step was to use a convolution neural network model similar to the one suggested by Vivek Yadav in his blog. I later arrived at slightly different architecture to reduce number of parameters that needed to be trained. This helped in speeding up the training process. I used a 2013 2Ghz core i7 cpu on my laptop to train my model.

In order to gauge how well the model was working, I split my image and steering angle data into a training and validation set. I used image augmentation and generators to generate more variations of training and validation data. This helped is better generalization and avoided overfitting.

Initially I made an error while randomly flipping images. Due to this I found that my car was hugging the right lane marker and would fell off the road on right after some time. After fixing the error car was running fine on the track.

At the end of the process, the vehicle is able to drive autonomously around the track without leaving the road.

#### 2. Final Model Architecture

The final model architecture (line 62) consisted of a convolution neural network with the following layers and layer sizes

Details:

```
____________________________________________________________________________________________________
Layer (type)                     Output Shape          Param #     Connected to
====================================================================================================
lambda_11 (Lambda)               (None, 80, 160, 3)    0           lambda_input_11[0][0]
____________________________________________________________________________________________________
convolution2d_41 (Convolution2D) (None, 80, 160, 3)    12          lambda_11[0][0]
____________________________________________________________________________________________________
convolution2d_42 (Convolution2D) (None, 78, 158, 32)   896         convolution2d_41[0][0]
____________________________________________________________________________________________________
maxpooling2d_30 (MaxPooling2D)   (None, 19, 39, 32)    0           convolution2d_42[0][0]
____________________________________________________________________________________________________
dropout_48 (Dropout)             (None, 19, 39, 32)    0           maxpooling2d_30[0][0]
____________________________________________________________________________________________________
convolution2d_43 (Convolution2D) (None, 17, 37, 64)    18496       dropout_48[0][0]
____________________________________________________________________________________________________
maxpooling2d_31 (MaxPooling2D)   (None, 8, 18, 64)     0           convolution2d_43[0][0]
____________________________________________________________________________________________________
dropout_49 (Dropout)             (None, 8, 18, 64)     0           maxpooling2d_31[0][0]
____________________________________________________________________________________________________
convolution2d_44 (Convolution2D) (None, 6, 16, 128)    73856       dropout_49[0][0]
____________________________________________________________________________________________________
maxpooling2d_32 (MaxPooling2D)   (None, 3, 8, 128)     0           convolution2d_44[0][0]
____________________________________________________________________________________________________
dropout_50 (Dropout)             (None, 3, 8, 128)     0           maxpooling2d_32[0][0]
____________________________________________________________________________________________________
flatten_10 (Flatten)             (None, 3072)          0           dropout_50[0][0]
____________________________________________________________________________________________________
dense_28 (Dense)                 (None, 128)           393344      flatten_10[0][0]
____________________________________________________________________________________________________
dropout_51 (Dropout)             (None, 128)           0           dense_28[0][0]
____________________________________________________________________________________________________
dense_29 (Dense)                 (None, 64)            8256        dropout_51[0][0]
____________________________________________________________________________________________________
dropout_52 (Dropout)             (None, 64)            0           dense_29[0][0]
____________________________________________________________________________________________________
dense_30 (Dense)                 (None, 1)             65          dropout_52[0][0]
====================================================================================================

```

#### 3. Creation of the Training Set & Training Process


##### Training dataset

I utilize three different training data sets to build my model.

1. generated data - I recorded two laps with driving in center of the track and then part of a lap with recovery driving.
2. udacity data
3. xbox data - Data generated by Patrick Kerns using xbox which also has both center track driving and recovery data.

All three camera images are used as part of the training data.

Example
![Training data example ](images/training_data_clr_images.png)

For the challenge part of the assignment, I took one lap of the challenge track to train the fully connected layers of the existing model.

##### Data Filtering

Analysis of the training data showed lot of samples with 0 degree ( i.e. straight driving ). In order to balance training set I limit number of examples selected from each angle range. This helps in getting a balanced training data. Histogram below shows pre and post filtering distribution of training dataset.

![Histogram of training data](images/training_data_pre_post_distribution.png)

Validation data is loaded separately and is an independent ideal run of the whole track.

##### Data Augmentation

I utilize a data generator during the training process which augments the training data. For augmenting training data I utilize following process.

1. Image is changed from RGB to YUV colorspace. Nvidia paper suggested that this gives better results.
2. Random Shear - Each image is randomly shifted horizontally using Affine transformations to create artificial turning angles. This further helps in generating more turning samples.
3. Random flip - Each image is randomly flipped by a probability of 50%. This helps in training for both left and right hand turns.
4. Random Brightness - V channel of each image is modified with random distribution to help model generalize for different light conditions and shadows.
5. Crop - Finally image is cropped and uninteresting sections from top and bottom of the frame are removed. Also image is resized to 80X160.

Example of augmented training data

![Augmented training samples ](images/data_augumentation.png)


Batch generator runs multiple times for each epoch and generates a batch at runtime by calling batch_generator function which inturn randomly selects images from the orignal training set and passes them through augmentation pipe to generate a training batch.
Below histogram shows distribution of steering angles in a batch.

![Histogram of training batch](images/batch_distribution.png)


##### Training Process

Adams optimizer with mean squared loss is used for training. Early stopping with loss delta of 0.001 is used during training. Model is saved at each epoch. I observed that typically model generated by 9th epoch was able to drive the car around the track.
