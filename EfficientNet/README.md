# EfficientNet model
## Description
This repository includes everything related to the melanoma classification model, from the model itself to the preprocessing 
of the dataset and the training/evaluation loops. The goal of this part of the project is to fine-tune our model architecture and
to provide the tools to establish wether the synthetic imaging approach through GAN was eficient or not.

## Implementation

### Network architecture
To be able to perform the melanoma classification task, we considered some pretrained networks like VGG and ResNet. However, and given how time consuming was training a dataset with so many images, we decided to build a fine-tuned EfficientNet model, a state-of-the-art approach, whose accuracy/number of parameters
ratio is much higher than the other two, making it smaller and faster. This is achieved by scaling the model, using coefficients so the structure
throughout the convolutional layers is the same except it gets modified by the coefficients. 

To implement this architecture we used [lukemelas](https://github.com/lukemelas/EfficientNet-PyTorch) model, which adapts from tensorflow's [original implementation](https://github.com/marketplace).

<img src="https://user-images.githubusercontent.com/37978771/115120986-b6828680-9fb0-11eb-94d7-94cbc80af7ac.png" width="500">

EfficientNet includes batch normalization by the end of every Convolutional Block. This provides the network with a stabilizing effect for the layers’ internal distributions, diminishing the effect of vanishing and exploding gradient.
Usually, Batch Normalization allows to train a network with relatively high learning rates. However, in this case we are fine-tuning our network, which requires a low LR in order not to overfit.

Although the model weights are pretrained and top layers unfrozen, Batch Normalization running mean and variance must keep updating themselves since the input images characteristics may differ from the pretrained ones.

Given all the EfficientNet models available, we have used model b3 for its number of parameters (12M) and depth are good enough to obtain a proper accuracy without taking too long.
Using b3 architecture, the model takes 30 minutes for each epoch. In addition, a deeper network may cause overfitting to the model due to the dataset's size.

### Dependencies

- Python 3.7
- [PyTorch](http://pytorch.org) 1.7.1+cu110
- [NumPy](http://www.numpy.org/) 1.20.1
- [Albumentations](https://github.com/ufoym/imbalanced-dataset-sampler) 0.5.2 
- [scikit-learn](https://scikit-learn.org/stable/index.html) 0.24.1  
- [efficientnet-pytorch](https://github.com/lukemelas/EfficientNet-PyTorch) 0.7.0
- [PIL](http://pillow.readthedocs.io/en/3.1.x/installation.html) 8.1.0
### Execution

To be able to train our model, we created three different scripts:

- classifier.py: Main script, it executes everything related to the training process.
- dataset.py: It contains the MyDataset class, used to access to the images and the csv files, linking each image to its diagnostic. It also includes the tools used for pre-processing the dataset.
- model.py: Where the efficientnet model is.
- utils.py: In this script you can find some other tools for modifying our dataset. This functions are manually executed. 

```
python3 classifier.py
```
Before executing the main script, these are the parameters that must be set, in order for the program to access and store certain files.

```
#File configuration
    root_path = './'
    path_img = root_path + 'images'
    path_test_reduced = root_path + 'test_reduced.csv'
    path_train_reduced = root_path + 'train_m_reduced.csv'
    path_val_reduced = root_path + 'val_m_reduced.csv'
    path_save_model = root_path + 'model' + model_number
    path_train_dloader = root_path + 'Data/train_loader_reduced.pt'
    path_val_dloader = root_path + 'Data/val_loader_reduced.pt'
    path_results = root_path + "results_" + model_number
    
   ```
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
| num_epochs     | Number of epochs       | 30      |
| learning_rate     | Learning rate       | 0.0001      |
| frozen_layers     | Frozen layers       | 18      |
| patience     | Patience       | 6      |

In order to accomplish our objective, the following parameters from both the model and the training section were tuned:

*	Data augmentation.

*	Fine-tuning

* 	Training the model using synthetic images

#### Data augmentation

Due to the imbalance of the dataset, the model overfits fast. In order to avoid this, we have implemented some augmentations, using the library albumentations.
The following figure show the model implemented without using augmentations, overfitting right at the beginning. 

<img src="https://user-images.githubusercontent.com/37978771/115122062-3bbc6a00-9fb6-11eb-9c05-519a13fa65cf.png" width="900">

Once the augmentation were applied, the model doesn't overfit anymore.

<img src="https://user-images.githubusercontent.com/37978771/115122075-4f67d080-9fb6-11eb-95f8-451a1ac36325.png" width="900">

In addition to the augmentations, all the images were resized to 128x128 so to adapt the network to the images synthetically generated.


### Fine tuning

To achieve a lower loss and thus, a higher accuracy, pretrained weights have been loaded to the network. The following figures present how the model was fine-tuned and improved just by unfreezing layers. Out of all the different configurations, these three were the ones that provided better results.

Apart from freezing/unfreezing layers, other parameters were modified, such as Batch Normalization momentum, which is set to 0.1 by default in the EfficientNet, and we changed it to 0.15 to attach more weight to the updated running means an variances. 

| Frozen layers | Train loss | Train acc |  Val loss |  Val acc |
|     :---:    |     :---:      |     :---:     |     :---:     |     :---:     |
| 22   | 0.30     | 0.87    |  0.34   |   0.87    | 
| 19     |0.36       | 0.85      | 0.29 | 0.87  |
| 18     | 0.28       | 0.88      | 0.30 | 0.88 |
| 17     | 0.39      | 0.83      | 0.37 | 0.85 |



22 Convblocks frozen.

<img src="https://user-images.githubusercontent.com/37978771/115137343-e750d300-a025-11eb-897e-809e7fcc7e57.png" width="900">

19 Convblocks frozen

<img src="https://user-images.githubusercontent.com/37978771/115137985-9d69ec00-a029-11eb-8e28-dada80fc546c.png" width="900">

18 Convblocks frozen

<img src="https://user-images.githubusercontent.com/37978771/115138051-110bf900-a02a-11eb-8828-2fb9ee8d68da.png" width="900">

17 Convblocks frozen

<img src="https://user-images.githubusercontent.com/37978771/115138090-40bb0100-a02a-11eb-8446-0b4f886f4012.png" width="900">

The difference of epochs between models is due to the network's early stopping. The moment the model stops improving and patience goes to 0, the network
stops training and saves the best scoring model.
The results obtained show that even when the dataset has been reduced considerably, we have achieved a proper accuracy. Within a bigger dataset, we shall consider
changing the model from b3 to b4 or even b5 and fine tuning again our architecture.

After the fine tuning, the model architecture was the following:

| Description | Value |
|     :---:      |     :---:     |
| Batch size     | 64    |
| Number of epochs       | 50      |
| Learning rate       | 0.0001      |
| Frozen layers       | 18      |
| Patience       | 6      |
| Normalization layer       | Batchnormalization      |
| Resize      | 128     |
| Avg. Time per epoch (min)     | 30     |

### Synthetic images

Now that we have already trained our model successfully, we may feed and train it with the synthetically generated images.  There are different approaches to this.
Firstly, we fed our network with a 40% of GAN images, equalling the number of malignant images to the number of benign ones, thus resulting in a balanced dataset. On the other hand, we may mantain the network imbalanced, by using the same
dataset and adding an extra 10% of synthetic images to study how does the model behaves. For this approach it is important to take into account that Imbalance Dataset Sample does not shuffle the dataset, so this was manually done in the CSV file.

We kept the previously set parameters, and tried both approaches on ACGAN and DCNSGAN.

| Type | Dataset | Train loss | Train acc |  Val loss |  Val acc |
|     :---:    |     :---:    |     :---:      |     :---:     |     :---:     |     :---:     |
| AC Gan   | Balanced     | 0.24     | 0.90    |  0.43   |   0.82    | 
| AC Gan     |Imbalanced       |0.32       | 0.86      | 0.37 | 0.83  |
| DCSN Gan     | Balanced       | 0.33       | 0.86      | 0.43 | 0.80 |
| DCSN Gan     | Imbalanced      | 0.20      | 0.92      | 0.36 | 0.85 |

The graph below show the results for the balanced ACGAN. Within the first ten epocs, the model establishes an accuracy above 80%.

![pic10](https://user-images.githubusercontent.com/37978771/115243296-10a35900-a123-11eb-8ce9-8c0a3c8e479c.png)


Imbalanced DCSN Gan achieves the better result of the four. In spite of training accuracy improving considerably, both validation loss and validation accuracy are a few tenths below. Balancing the dataset with synthetic images means that there are more fake malignant images than benign. From the results from all four trainings, we can conclude that although the performance hasn't improve, it hasn't worsened excesively either, meaning that the generated images may be considered as a first step for developing a GAN and classifier that improves the accuracy with the required resources.
