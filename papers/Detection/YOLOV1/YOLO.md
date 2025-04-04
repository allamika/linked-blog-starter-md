 [You Only Look Once Unified, Real-Time Object Detection](https://arxiv.org/pdf/1506.02640) May 2016
### Abstract
Frame object detection as a regression problem to spatially separated bounding boxes and associated class probabilities. Vs current use classification system for detection task.
Way faster but bit less performant than DPM and R-CNN.
-  Fast: First real time detection ?
- See "entire" image (better dans sliding window)
-  Better generalization
-  Bit behind state of the art in accuracy

### Unified Detection
#### 1.End to end:
- Cut image in S×S tiles
- For each cell predict B boxes (x,y,w,h,conf) : confidence reflect $Pr(Object) * IOU_{pred}^{truth}$
- Each cell predict C conditional class probabilities $Pr(Class_i|Object) * Pr(Object) = Pr(Class_i)$ 
  ![[End-to-end.png]]
#### 2.Network Design

![[Architecture_fig.png]]
Prediction shape $S*S*(B*5+C) = 7*7*(2*5+20) =7*7*30$ 
So for each cells 2 boxes and 20 class predictions are produced 

### Training

Convolutional layers are pretrained on ImageNet. 20 first convolutional follow by an average pooling layer and a full connected layer trained to achieve 88% on ImageNet.

For the predicted boxes, h and w are normalized by the width and height of the image and between 0 and 1. For x and y the normalization is in the cell and between 0 and 1.

Use a rectified linear activation $\phi(x) = \left\{ \begin{array}{cl}x, & if \ x > 0 \\ 0.1x, & othewise \end{array} \right.$

Loss: sum-squared error (mid because same weights for localization and classification)
Problems:
- empty cells over power gradient from cells with object pushing confidence scores toward 0. This cause instability and divergence -> 
	- decrease the confidence loss for boxes without object $\lambda_{coord} = 5$
	- increase coordinate loss $\lambda_{coord} = 0.5$
- penalize deviation the same way in small and large boxes:
	- predicting square root for width and height
- multiple box per grid cell
	- assign each predictor to the highest current IUO with the ground truth, hoping for specialization between predictors.
![[Loss.png]]

Hyper-parameters: batch size 64, momentum of 0.9 and decay of 0.0005. For the learning rate: start low $10^{-3}$, slowly raise $10^{-2}$ and stay a $10^{-2}$ (75 epoch), go down to $10^{-3}$ for 30 epochs, then finish at $10^{-4}$ (30 epoch). When starting too high the model diverge.

Use of dropout layer 0.5 and data augmentation

###  Limitations

Spacial constraint: each cells can produce 2 boxes => problems with small objects in group
Class spacial constraint: only one class per cell
The loss do not perfectly align with IOU maximisation