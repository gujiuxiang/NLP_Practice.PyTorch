## [HyperNet: Towards Accurate Region Proposal Generation and Joint Object Detection](https://arxiv.org/abs/1604.00600)

论文出处是CVPR2016的Spotlight，目前只有arxiv版本：http://arxiv.org/pdf/1604.00600.pdf
## 概述
文章做的问题是通用物体检测，基本框架如下图所示：

<p align="center"><img src="http://7j1yz3.com1.z0.glb.clouddn.com/小Q截图-20160405210704.png" width="800" ></p>


基本可以理解成基于Faster R-CNN做了一些改进。

这里先简述Faster R-CNN的基本要素（因为之前没有空写Faster R-CNN就在这里简单review好了= =）。如下图所示，Faster RCNN可以理解成RPN + Fast R-CNN。即用RPN（Region Proposal Network）来提取proposal，然后用Fast R-CNN的分类器（即3个全连接层加上softmax分类器）作为detector，对这些proposal进行类别上的识别。另外，RPN跟Fast R-CNN能够通过分步训练达到sharing卷积部分，能够在inference阶段一次forward就直接得到proposal和detection的结果。

<p align="center"><img src="http://7j1yz3.com1.z0.glb.clouddn.com/小Q截图-20160405212339.png" width="400" ></p>

回到这篇文章，文章提出的框架称为HyperNet，比起Faster R-CNN，它的最主要改进是使用了多层特征融合（作者称之为Hyper feature extraction，也就是hyperNet的由来）。除了之外的trick还有，增加额外的Conv层来提取信息（增加网络深度），以及提到了如何做加速。下面分段叙述。

### 多层特征融合

作者在文章中提到，使用多层特征融合的动机是用于处理小尺度物体。具体来说，在CNN中，一般分类器都是直接接在最顶层的Conv上的，而这个Conv经过多次pooling后，其实“缩水”得很厉害。比如在VGG-16中，Conv5_3的长和宽都缩水到原图片的1/16。这样就会导致对于某些小物体很难提取到精细的特征（比如，对于32x32的物体，Conv5_3的feature只剩下2x2大小，就很不精细了）。

要解决的办法其实也很直观，既然顶层的Conv层缩水了，那么就把它扩大就可以了，所以这里使用了Deconv来对高层特征进行上采样。同时，还可以接上浅层的Conv层来进一步增加判别性。这样就可以得到更加精细，判别性更强的特征了。

另外，在concat不同层特征的时候，作者做了一个LRN的正则化。正则化的步骤应该是必须的（但不限于使用LRN）。这是因为不同层的特征有着不一样的norm，如果直接concat到一起，norm大的特征就会抑制住norm小的特征，这样很容易就会学不好。这个观察是在ParseNet中提出的，具体可以参考：http://www.cs.unc.edu/~wliu/papers/parsenet.pdf

### 增加额外的Conv层

在HyperNet中，在很多地方都添加了额外的Conv层，比如在多层特征融合的时候，对于每层特征后面都接一个Conv层，在ROI pooling之后，也接了额外的Conv层。注意到接上的这些Conv层其实也都起到了压缩特征长度减少网络参数的作用，对于防止过拟合应该也有一定作用。

### 加速
文章提到他们使用VGG16也能达到5fps，除了使用了Titan X显卡之外，最主要的trick是交换了13x13x126的ROI pooling层和3x3x4的Conv层。
但其实本人觉得这样固然比原来快，但为何不在训练前就直接交换，而不是训练后在测试阶段才交换位置？感觉这一步蛮奇怪的。

<p align="center"><img src="http://7j1yz3.com1.z0.glb.clouddn.com/小Q截图-20160405214318.png" width="800" ></p>
