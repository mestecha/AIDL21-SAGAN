# EfficientNet mod
## Description
This repository includes everything related to the melanoma classification model, from the model itself to the preprocessing 
of the dataset and the training/evaluation loops. The goal of this part of the project is to fine-tune our model architecture and
to provide the tools to establish wether the synthetic imaging approach through GAN was eficient or not.

## Implementation

### Network architecture
To be able to perform the melanoma classification task, we considered some pretrained networks like VGG and ResNet. However, 
we decided to use a state-of-the-art approach by implementing a fine-tuned EfficientNet model, whose accuracy/number of parameters
ratio is much higher and efficient than the other two. This is achieved by scaling the model, using coefficients so the structure
throughout the convolutional layers is the same except it gets modified by the coefficients. 

<img src="https://user-images.githubusercontent.com/37978771/115120986-b6828680-9fb0-11eb-94d7-94cbc80af7ac.png" width="500">

EfficientNet includes batch normalization by the end of every Convolutional Block. This provides the network with a stabilizing effect for the layers’ internal distributions.
Usually, Batch Normalization allows to train a network with relatively high learning rates. However, in this case we are fine-tuning our network, which requires a low LR in order not to overfit. 
Although the model weights are pretrained and top layers unfrozen, Batch Normalization running mean and variance must keep updating themselves since the input images characteristics may differ from the pretrained ones.

Given all the EfficientNet models available, we have used b3 model for its number of parameters and depth are good enough to obtain a proper accuracy without taking too long.
Using b3 architecture, the model takes 30 minutes for each epoch. In addition, a deeper network may cause overfitting to the model due to the dataset's size.

### Metrics

To be able to evaluate and measure the effectiveness of our network, we have implemented the following metrics:
	
* Loss: We are training a binary classifier, so we are using Binary Cross Entropy to calculate the loss between the output of our model and the ground truth.
	
* Accuracy: To know the ratio of correct predictions.

* AUC score: Since we have an initially imbalanced dataset, accuracy is not enough to now whether our network is performing properly. 

In order to control whether our network overfits and prevent it from keeping training and spending resources in GCP, we have set a patience threshold so the network stops early in case it is not improving.

* Patience: 15 epochs.

### Parameters

To train our network, we used the parameters specified in figure #. Some parameters were restricted to our working environment, such is the case of the batch size 
and other were tuned by trial and error.


| Variable | Description | Value |
|     :---:    |     :---:      |     :---:     |
| batch size   | Batch size     | 64    |
| num_epochs     | Number of epochs       | 50      |
| learning_rate     | Learning rate       | 0.0001      |
| frozen_layers     | Frozen layers       | 18      |
| patience     | Patience       | 15      |

In order to accomplish our objective, the following parameters from both the model and the training section were tuned:

*	Data augmentation.

*	Learning rate.

*	Unfreezing pretrained layers.

* Training the model using synthetic images

#### Data augmentation

Due to the imbalance of the dataset, the model overfits fast. In order to avoid this, we have implemented some augmentations, using the library albumentations.
The following figures show the model implemented without using augmentations, overfitting right at the beginning and the results of using them, where the model doesn’t overfit. 
In addition to the augmentations, all the images were resized so to adapt the network to the images synthetically generated.

<img src="https://user-images.githubusercontent.com/37978771/115122062-3bbc6a00-9fb6-11eb-9c05-519a13fa65cf.png" width="900">

<img src="https://user-images.githubusercontent.com/37978771/115122075-4f67d080-9fb6-11eb-95f8-451a1ac36325.png" width="900">



