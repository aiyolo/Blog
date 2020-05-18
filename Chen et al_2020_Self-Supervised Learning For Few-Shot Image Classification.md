<img src="https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518171156141.png" alt="image-20200518171156141" style="zoom:50%;" />

论文名称：**SELF-SUPERVISED LEARNING FOR FEW-SHOT IMAGE CLASSIFICATION**

论文链接：http://arxiv.org/abs/1911.06045

代码链接：https://github.com/phecy/ssl-few-shot

### Abstract

以元学习的方式处理小样本学习任务时，每个任务都只有少量样本，网络非常容易过拟合。越来越多的工作表明，获得一个鲁棒性强的预训练网络对于元学习阶段来说非常重要。然而许多预训练网络都是在有限样本上以监督学习方式训练，网络结构一般采取ResNet12 或者wide ResNet，而用更深的网络效果就变差了。在本文中作者提出采用自监督方式训练一个更加大型的网络，然后用该预训练网络进行第二阶段的元学习，从实验结果上，本方法能够显著超越现有的方法。

### Method

网络结构

<img src="https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518173020067.png" alt="image-20200518173020067" style="zoom:50%;" />

一共分为两个阶段：

#### Self-supervised learning stage

在本阶段，目标是预训练一个网络，以得到更好的特征表示。在这里作者采用了**AMDIM**（Augmented Multiscale Deep InfoMax）方法，大概意思是增强的多尺度的深度最大化互信息的方法，来得到一个预训练网络。

amdim代码链接：https://github.com/Philip-Bachman/amdim-public

AMDIM的核心思想在于最大化同一张图片的两个尺度$（x_a,x_b）$的全局特征和局部特征的互信息。

互信息是用来衡量两个随机变量的共享信息。

<img src="https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518175059340.png" alt="image-20200518175059340" style="zoom:50%;" />

在这里我们只有样本，没有潜在的分布表示，因此没办法直接计算互信息。但是有一篇文献证明了可以通过最小化噪声对比估计（NCE）来最大化互信息的下界

具体的：我们要最大化<img src="https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518175728914.png" alt="image-20200518175728914" style="zoom:50%;" /><img src="https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518175747691.png" alt="image-20200518175747691" style="zoom:50%;" ><img src="https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518175812877.png" alt="image-20200518175812877" style="zoom:50%;" />

其中$f_g$表示表示全局特征,$f_5$是编码器的5x5特征图，$f_7$是编码器的7x7特征图

NCE损失定义为：

![image-20200518182822278](https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518182822278.png)

$N_x$表示图片x的negative samples, $\phi$是距离度归，这样整体损失为：

<img src="https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518182729901.png" alt="image-20200518182729901" style="zoom:50%;" />

![image-20200518183047549](https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518183047549.png)

#### Meta learning Stage

元学习阶段就跟原型网络的过程一样。

### 实验

在miniimagenet上的表现

![image-20200518183316154](https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518183316154.png)

Mini80-SSL:指在80个类（training + validation）上self-supervised trainning，没有用到标签

Mini80-SL:指使用相同的AmdimNet，以交叉熵损失进行supervised learning,有用到标签，应该也是80个类

Image900-SSL:指在ImageNet1K中除去MiniImageet的100各类后剩下的900个类上训练

其他同理

实验结论：

- 在5-way 1-shot: mini80-ssl超过protonet+(预训练+protonet)7.53%,超过leo 2.27%（sota）
- Image900_SSL最高，因为预训练的图片更多
- mini80_sl 不采用自监督训练，效果非常差
- mini80_SSL- 预训练之后不进行元学习，直接使用NN分类，结果也很差，说明元学习在fine-tune网络的时候有效

![image-20200518184444649](https://raw.githubusercontent.com/aiyolo/blog/master/assets/image-20200518184444649.png)