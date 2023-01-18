# Language-Classification-of-Audio
Classifying the language spoken in audio clips of native speakers using audio augmentation techniques and Convolutional Neural Networks. MP3 files are scraped from audio-lingua, and saved onto AWS. Because the audio files are 

# Project Outline
1. Web Crawler
2. AWS Storage
3. Audio Augmentation
4. Data Representation
5. CNN

# Web Crawler/Scraper

I am collecting data from a website called audio-lingua, that has a database of about 8 languages of audio clips of native speakers. I collected data on the most popular languages, English, Russian, French, Spansih, Chinese, and German.

Below is an example of the download pages:

<img width="412" alt="image" src="https://user-images.githubusercontent.com/79114425/213087717-2869d467-683e-4f71-970e-dd6afef0b5a8.png">

Using the python package requests and Beautiful Soup, we can iteratively go through the different webpages, and navigate the HTML code with Beautiful Soup to find the download links. I collect them in a csv of strings and iteratively download them onto AWS

Code Snippet:

<img width="492" alt="image" src="https://user-images.githubusercontent.com/79114425/213092531-0550892c-dfa9-46b6-b7ec-416044547a9b.png">

# AWS storage

For storage we use S3, and make a bucket that contains different file paths for our different audio files...
<img width="607" alt="image" src="https://user-images.githubusercontent.com/79114425/213089223-a0180d9b-ab3c-422b-8ded-2caeafddafe2.png">

I am using the boto3 package for accessing AWS


# Audio Augmentation

Augmentation is one of the most important steps when working with both audio or image data. The reason for this is that the classifications behind different audio images are never static. 
* Audio is dynamic and the same sentence can be expressed on many wavelengths or frequencies. 
* In order to deal with this we have different ways to transform audio including using Time Mask, High Pass Filter, White Noise, Pitch Scaling, Polarity Inversion, Random Gain
   * Using NumPy to apply random transformations of the audio data
   * This increases the number of samples we have to work with, and also helps our model to generalize a little better, and be sensitive to small changes in the audio clips 

## Augmentation Functions
Time Mask - Randomly set values to 0, randomly blocking off some of the time periods to reduce overfitting

High Pass Filter - Only keep some of the values from the high frequencies, removing certain low frequency values of the sound

White Noise - The basic white noise you hear that is sort of a constant signal underneath the other sounds

Pitch Scaling - Change the pitch at which the audio clips are represented

Polarity Inversion - A simple transformation, multiplying all values by -1

## Visual Examples - Random Transformation
-Time Mask
![image](https://user-images.githubusercontent.com/79114425/213090565-2e696f61-e44e-4f0a-963c-413107963417.png)

-Noise and Gain: Add some base white noise, and gain that increases the abs value of frequency levels

Original Signal:

<img width="468" alt="image" src="https://user-images.githubusercontent.com/79114425/213087888-d0d5976c-6433-404f-a04e-6563ef893fac.png">

Signal Augmented:

<img width="464" alt="image" src="https://user-images.githubusercontent.com/79114425/213087773-8c506468-fbe5-4744-bbcc-428d7b05f221.png">

As you can see, the scale of the values has changed, and there is some noise and gain causing slightly larger fluctuations in the audio file.


# Data Representation
We represent data for the CNN as "images"
* The images below represent a Mel Spectrogram and MFCC. The Mel Spectogram is a close representation of the audio that humans hear, one that highlights sound waves at certain frequencies. 
   * The wavelengths we hear and the ones a dog hears are not the same! So the mel spectrogram does a better job of representhing these values
* These arrays that define these spectrograms are going to be put into a Convolutional Neural Network
   * In essence we are using computer vision techniques to classify different audio clips into the spoken language

<img width="227" alt="image" src="https://user-images.githubusercontent.com/79114425/213087804-f04d6d5b-9f97-4564-8acc-c04788b6eb61.png">

<img width="202" alt="image" src="https://user-images.githubusercontent.com/79114425/213087844-c6dcf2be-55f9-4eec-9ace-4e357dc06c26.png">

# Convolutional Neural Network Model

The main considerations are to ensure that the model generalizes well. Because audio is so dynamic, this can be an extremely difficult task for a CNN. 
* Beyond simple words, classifying audio representations into languages based off its spectral qualities can be difficult, if not impossible. 
* As you might be thinking, different speakrs of different languages might have a lot of the same speaking qualities. 
* To do this, we need to enforce the model to generalize
   * Some methods are to reduce parameters and generalize by adding regularization, adding dropout, and with audio augmentations

Here is the model of the convolution network:


* Research has shown that increasing the number of neurons in the convolutional network on each layer improves performance
* We add padding to be the same so that the inputs and outputs of convolution layers are consistent
* We also use MaxPooling and eventually flatten our outputs from the convolutional layers
   * With these outputted values, we run them through some dense layers to actually calculate weights
* We use Categorical Cross Entropy as our error metric which is essentially a softmax objective function applied to cross entropy
