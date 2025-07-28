

### Metrics and NMS
-  IOU
![[IoU.png]]
- Precision Recall: you consider a positive if the IoU of the predicted box and the ground truth is above the AP threshold

Different computation of AP on VOC and COCO
- VOC :
	- 1. precision recall curve by varying the confidence threshold of the model  
	- 2. Calculate AP using interpolated 11 point sampling for precision recall curve
	- 3. Mean of APs across all categories
- COCO : AP for multiple IoU threshold then mean across the different IoU
- NMS: Yolo models tends to predict a lot of boxes for a same image object. The goal of the NMS algorithm is to filter overlapping boxes. This algorithm is compute across boxes of the same class
	- input:  IoU threshold 
	- sort boxes by confidence
	- iterate through boxes
		- keep the box if it as not an IoU with an already keep box over the threshold

### Yolov1
### Yolov2
Improvement:
- **Batch Normalization** on convolutional layer to improve convergence and overfitting
- Improvement on the **resolution** of the input 
- Remove dense layer in the backbone making it a **full convolution backbone**
- Use a set of **anchor boxes** in each cell of the grid. This boxes have predefined shape and the system predict the coordinates, the deformation of the box and the class
- Use **K means** to select **anchor boxes** size
- **Prediction relative** to the cell
- **More feature map**
- **Multi-scale training**: because now the backbone is now full convolutional it's indifferent to size so it can be train on multiple input dimensions
### Yolov3

### Yolov4

