[YOLOv3: An Incremental Improvement](https://arxiv.org/pdf/1804.02767)
### Abstact

Little design changes
Bit bigger
Still fast

### Evolutions

**Bonding Box Prediction**. No longer assign on bounding box to each ground truth.
Assign every prediction with on overlap over .5.

**Class prediction**. Each box predict the class of the bounding box. Use binary cross entropy for the multi-label classification. (No softmax witch imposes the assumption that each box has exactly one label ??)

**Predict Across Scales**. [Feature Pyramid Networks for Object Detection](https://arxiv.org/pdf/1612.03144)
Predict boxes at 3 different scales.
First scale : Base feature extractor -> few conv layer -> predict 3D tensor
	Other scales: Upsampling of the feature map of 2 previous layer + earlier feature map -> conv layers -> predict 
Each scale predict 3 box 
Post process: K-means create clusters ??
![[ArchitectureYOLOv3.png]]

**Feature extractor**:  convolution + residual connection + max pool

**Backbone**: Darknet-19 less accuracy than ResNet models but faster


## Le banger
![[Le banger.png]]
