[YOLOv3: An Incremental Improvement](https://arxiv.org/pdf/1804.02767)
### Abstact

Little design changes
Bit bigger
Still fast

### Evolutions

**Bonding Box Prediction**. No longer assign on bounding box to each ground truth.
Assign every prediction with on overlap over .5.

**Class prediction**. Now use binary cross entropy for the multi-label classification. (No softmax witch imposes the assumption that each box has exactly one label ??)

**Predict Across Scales**. Predict boxes at 3 different scales.
