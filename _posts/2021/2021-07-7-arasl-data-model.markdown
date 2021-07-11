---
layout: post
title: ArASL: Data Preparation & Model Training
date: '2021-07-7 10:42'
excerpt: >-
Preparing the Dataset & Building ArASL Model.
comments: true
published: true
---

Welcome back to my AI blog! In my last post, I gave a brief about Arabic Sign Language Recognition. Today, I’ll help you continue your journey with prepare your dataset and build your ArASL model.

## In-depth look at Dataset
[KArSL](https://dl.acm.org/doi/10.1145/3423420) database for ArSL, consisting of 502 signs that cover 11 chapters of ArSL dictionary. Signs in KArSL database are performed by three professional signers, and each sign is repeated 50 times by each signer. The database is recorded using state-of-art multi-modal Microsoft Kinect V2.



<div class="fig figcenter fighighlight">
  <img width="300" height="300" src="../../img/chars.png">
  <div class="figcaption" style="text-align: center;"><br>
  </div>
</div>

We will use an alphabet sub-dataset with only one hand for now. We have `5789` videos as total number, for trainset we have `4600`, testset `917` and validationset `272`, for `39` characters/labels. (As shown in above fig.)

## Extract features from videos
In this part, I used MediaPipe Hand, a machine-learning employed high-fidelity hand and finger tracking solution. It detects 21 Landmark points as shown in Fig. are recorded from a hand in a single frame with the help of multiple models which are working simultaneously.

<div class="fig figcenter fighighlight">
  <img src="https://miro.medium.com/max/1400/1*fMBLvkdLbg0MEfv7KbJZjQ.png">
  <div class="figcaption" style="text-align: center;"><br><a style="text-align: center;" href="https://miro.medium.com/max/1400/1*fMBLvkdLbg0MEfv7KbJZjQ.png">Hand Coordinates</a><br>
  </div>
</div>

Mediapipe Hands consists of two different models working together namely Palm Detection Model in which a full image is identified and it draws a box around the hand, and Hand Landmark Model operates on this boxed image formed by Palm Detector and provides high fidelity 3D `(X, Y, Z)` hand keypoint coordinates. (As shown in above fig.)

After feeding our dataset videos into Mediapipe Hands solution we can see the results below

<div class="fig figcenter fighighlight">
  <img width="300"  src="../../img/data.png">
  <div class="figcaption" style="text-align: center;"><br>
  </div>
</div>

# Data Pre-processing
Before feeding the data to the neural network I decided to work on pre-processing step, which helps me to improve model results. I chose to work with `Z-score` method, which takes each frame, Then, takes all x coordinates and converts them into z scores and then takes all y coordinates and convert them into z scores.

<div class="fig figcenter fighighlight">
  <img src="https://www.statisticshowto.com/wp-content/uploads/2016/11/alternate-z-score-150x76.png">
  <div class="figcaption" style="text-align: center;"><br><a style="text-align: center;" href="https://www.statisticshowto.com/probability-and-statistics/z-score">Source</a><br>
  </div>
</div>

Z-score gives you an idea of how far from the mean a data point is. More technically it’s a measure of how many standard deviations below or above the population mean a raw score is. In our case, Z-score help us to `reduce variability between samples (though they were already normalized by Mediapipe itself) but didn't know which would be more effective.

Also, I rearranged the number of frames in a single video to `45` frames this will help us to have a static number of frames for each character.

<div class="fig figcenter fighighlight">
  <img src="../../img/hist.png">
  <div class="figcaption" style="text-align: center;"><br><h5 style="text-align: center;">Frame-Count Histogram</h5><br>
  </div>
</div>

## Model Workflow
At first, the Mediapipe solution captures the Palm using Palm Detector Model and draws a bounding box around the hand.
Next, the hand landmark model locates 21 keypoint 3D hand coordinates `(X, Y, Z)`.
Then, these hand landmarks are captured by the model which is preprocessed further and sent to the classifier model to classify the hand gestures.

# Model Training 
For the model structure for training, I used the `BiLSTM` model. `Bidirectional recurrent neural networks(RNN)` are really just putting two independent RNNs together. This structure allows the networks to have both backward and forward information about the sequence at every time step
Using bidirectional will run your inputs in two ways, one from past to future and one from future to past and what differs this approach from unidirectional is that in the LSTM that runs backward you preserve information from the future and using the two hidden states combined you are able in any point in time to preserve information from both past and future.


<div class="fig figcenter fighighlight">
  <img src="
https://miro.medium.com/max/1400/1*B5NHtY8_Y4we0DE4Y-acBA.png">
  <div class="figcaption" style="text-align: center;"><br><a style="text-align: center;" href="https://medium.com/@raghavaggarwal0089/bi-lstm-bc3d68da8bd0z-score">Source</a><br>
  </div>
</div>

The Tensorflow model trained using the following architecture (velow fig. Model Architecture) is saved in the HDF5 file, converted to the TensorFlow Lite model. This Tensorflow Lite model that stores the model architecture and weights will be used to classify hand gestures in the final app.


<div class="fig figcenter fighighlight">
  <img width="300"  src="../../img/nn_graph.png">
  <div class="figcaption" style="text-align: center;"><br>
  </div>
</div>


## Results

After running the model, we achieved an accuracy of `83.30%` and loss of `0.0999` for training. For Validation, accuracy of `80.15%` and loss of `0.1992`.

<div class="fig figcenter fighighlight">
  <img src="../../img/acc.png">
  <div class="figcaption" style="text-align: center;"><br><h5 style="text-align: center;">Accuracy  &  Validation Accuracy</h5><br>
  </div>
</div>


<div class="fig figcenter fighighlight">
  <img src="../../img/loss.png">
  <div class="figcaption" style="text-align: center;"><br><h5 style="text-align: center;">Loss  &  Validation Loss</h5><br>
  </div>
</div>


## Model Testing
On testingset, we achieved an accuracy of `81%` and F1-score Macro of `91%`,  F1-score weighted of `81%`.

<div class="fig figcenter fighighlight">
  <img src="../../img/report.png">
  <div class="figcaption" style="text-align: center;"><br><h5 style="text-align: center;">Classification Report</h5><br>
  </div>
</div>


## Conclusion

In the upcoming articles, we will discuss how to integrate our trained model with the Mediapipe framework to build our IOs/Android App.