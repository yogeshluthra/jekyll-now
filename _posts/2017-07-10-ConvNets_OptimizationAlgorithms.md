---
layout: post
title: Neural Nets and Optimization Algorithms (Fully Connected and Convolutional Neural Nets)
markdown: true
---
{% include math.html %}

It was a great learning experience implementing:
- Fully connected Networks
- Convolutional Neural Networks
- [Batch Normalization](https://arxiv.org/pdf/1502.03167.pdf) was recently proposed.
- [Drop Out](https://www.cs.toronto.edu/~hinton/absps/JMLRdropout.pdf)
- A New Optimization Algorithm, I came up with
- Modifications to Batch Normalization to consider Cumulative Moving Averages of Mean and Variances rather than Exponential Moving Averages.
all on [CIFAR-10 dataset](https://www.cs.toronto.edu/~kriz/cifar.html).  
This dataset has 10-classes.  

### Batch Normalizaion
As I will show below, Batch Normalization is a significant improvement. So it deserves a small section of its own.  
Paper (linked above), proposed to center activity of hidden units to zero mean and unit variance, by doing following in forward propagation:  
![alt text]({{site.url}}/images/batchNorm_forward.png "batch norm forward"){:height="300px" width="300px"}  
This is simple. However back propogation through this function does the trick! Basically it proposes following main things:  
- Center and normalize using mini-batch samples.  
- Since mean and variance are based on current mini-batch samples, back propagation must consider them as variables.  
- $$\gamma$$ and $$\beta$$ are proposed, so as to cover identity transform in the solution space. (I find it very interesting that some major breakthough papers (like ResNet) also are designed around covering identity function in solution space)  
So, backward propagation becomes:  
![alt text]({{site.url}}/images/batchNorm_backward.png "batch Norm backward"){:height="200px" width="300px"}  

However, during test (or validation) we simply ignore mini-batch sample mean/variance but rather using mean/variance calculated from whole data.  
This can be done in various ways, like [I describe here](https://yogeshluthra.github.io/MeanVariance_Estimation_from_Online_Data/)

Source code for [Batch Normalization in Python/Numpy can be found here](https://github.com/yogeshluthra/Deep_Learning/blob/master/assignment2/cs231n/layers.py)

### Description of project
This project is completely implemented using Python, Scipy and Matplotlib libraries. Also all evaluations are done on my Macbook, so runs slower compared to GPU based machines.  
Thus networks are not very deep.  

Full Connected Network architecture I tried:  
**[Affine - [batch norm] - ReLU]x5 - Affine - softmax**  
(each hidden layer was 100x units)  

Convolutional Neural Network I tried was shallower than Fully Connected network mentioned above, but achieves much higher accuracy!  
**conv - [spatial batch norm] - relu - 2x2 max pool - affine - [batch norm] - relu - affine - softmax**    
(Input dimension: 3x32x32; Conv net size 32x3x7x7; hidden layer units 100x)  
(basically only 3 layers with tunable parameters rather than 6 in Fully Connected above)  

I will implement all these in TensorFlow soon!  

Since Fully Connected results are not very interesting, I won't show them here but can be viewed by interested reader [here](https://github.com/yogeshluthra/Deep_Learning/blob/master/assignment2/FullyConnectedNets.ipynb)  

I am exited to share a new optimization algorithm I tried, which combines Nestrov Momentum, RMSProp and bias corrections as proposed for Adam algorithm.

Here is a glossary of various famous algorithms used in Neural Network world quite often:
- SGD
- SGD with Momentum
- SGD with Nestrov Momentum
- RMSProp
- Adaptive momentum (Adam)  
[This link](http://cs231n.stanford.edu/slides/2016/winter1516_lecture6.pdf) describes all of above in great detail.

Nestrov Momentum in my studies almost always performs better than simple Momentum.
So, I tried a combination of Nestrov Momentum with RMSProp.
Following picture describes mathematics behind various algorithms and hopefully you can find a justification of what I am proposing.  
![alt text]({{site.url}}/images/first_order_optimization_algos.png "first_order_optimization_algos"){:height="500px" width="800px"}  

All of the implemented algorithms [can be found here](https://github.com/yogeshluthra/Deep_Learning/blob/master/assignment2/cs231n/optim.py)  

### Results
Now lets look at results of my experiments with Batch Normalization and New Algorithm (Nestrov_Adam) I propose.  
![alt text]({{site.url}}/images/results_convnet_batchNorm_optimAlgo.png "results_convnet_batchNorm_optimAlgo"){:height="500px" width="800px"}  

