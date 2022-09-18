# To visualize how a CNN works.  
For Chinese reader, there is a [Traditional Mandarin translation version](https://github.com/sleepingjun/How-a-CNN-work/blob/main/Traditional%20Mandarin%20doc.md).  
This was my freelance research assignment in January, 2021.  
Visualizing Convolutional Neural Networks using weights and heatmap(grad-CAM and grad-CAM++).

## Intorduction
One of the most debated topics in deep learning is how to interpret and understand a trained model. This project is to research the methods of how to visualize a CNN model.

## Importance of Visualizing a model
It is absolutely crucial that we know what our model is doing and how it’s making decisions on its predictions.  
To visualize the model can help us to know the following things:
  1. Understanding training process of the model.
  2. Assistance in hyperparameter tuning.
  3. Finding out the predicted location of the model.

## Technologies
  ‧ tensorflow 2.4.0
  ‧ keras 2.4.3
  ‧ opencv 4.1.2

## Reference
  1. Grad-CAM: https://arxiv.org/abs/1610.02391
  2. Grad-CAM++: https://arxiv.org/abs/1710.11063
