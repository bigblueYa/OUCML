# 联合子带学习的CliqueNet在小波域上的图像超分辨复原

[论文地址](https://arxiv.org/abs/1809.04508E)
这篇论文是北京大学的[林宙辰](
http://www.cis.pku.edu.cn/faculty/vision/zlin/zlin.htm)老师组的工作。

#### 引言

卷积神经网络在单幅图像超分辨率上取得了很大的成功。但是作者认为这些方法普遍有个问题就是产生过于平滑的图像而失掉了纹理细节。为了解决这个问题，作者提出了超分辨团结构网络以及在小波域上的学习方法。

#### 整体框架

![image-20190413152420009](https://ws4.sinaimg.cn/large/006tNc79ly1g211bf0zz2j30go04uq50.jpg)
这张图就是作者提出的srcliquenet。

#### 特征嵌入网络

首先，特征嵌入网络会在低分辨率图像上提取一系列的特征。这个部分成为特征嵌入网络，在这一部分中，不会改变输入图像的大小。然后，将这些特征图送入到图像重建网络。这里的团结构的上采样网络里面包括四个小波子带网络。最后通过离散小波逆变换将四个子网络的输出分别合成对应的图像，然后通过一个卷积层输出最终的三通道的高清图像。
下面我们来详细看一下特征嵌入网络和小波域的上采样方法。

![image-20190413152531213](https://ws3.sinaimg.cn/large/006tNc79ly1g211cmygglj30ds05f74p.jpg)


在特征嵌入网络中，首先是两层卷积网络，第一层卷积的作用是扩充输入图像的通道。F1的输出通道为nlg，n是block的数量，l是每一个block的卷积层数量，g每一个block的增长率。F2的作用是将F1的输出通道数重新整合成它的输入block的通道数，F2的输出通道数为lg。

![image-20190413152615686](https://ws4.sinaimg.cn/large/006tNc79ly1g211df4nfwj30go057jsy.jpg)

这里的团网络模块，是多个团网络的堆叠。

![image-20190413152635097](https://ws4.sinaimg.cn/large/006tNc79ly1g211drulksj30dy0bc452.jpg)

团结构网络是作者之前在CVPR2018上的工作。cliquenet的每一个模块可分为多个阶段。下图中示意的cliquenet包含两个阶段，第一个阶段和dense net一样。在第二个阶段中，每一个卷积运算的输入不仅包括前面所有层的输出特征图，同样包括后面层级特征图。
$$
X_{i}^{(k)}=g\left(\sum_{l<i} W_{l i} * X_{l}^{(k)}+\sum_{m>i} W_{m i} * X_{m}^{(k-1)}\right)
$$
可以看到更新公式中，i为第i层，k为第k个stage,它包括在这个阶段中，低于i层的feature和上一阶段中高于i层的feature。作者认为这种结构重复使用了不同层级的特征图，并且相较于densenet较少了参数量。在本文中的特征嵌入网络中，作者就采用了这种结构提取特征。同时，因为在超分辨的问题中，输入特征一般包含着大量的有用信息。作者也加入了跳跃连接。最后将n个block的特征图进行拼接得到输出特征图。

#### 图像重建网络

接下来是图像重建网络模块，在讲网络结构之前，先来看下小波子带的概念。

![image-20190413152740410](https://ws4.sinaimg.cn/large/006tNc79ly1g211ew08d9j30d707xdhv.jpg)

这张图是2维快速小波变换的示意图。出自中科院赫然老师组的[WaveletSRNet](
http://openaccess.thecvf.com/content_ICCV_2017/papers/Huang_Wavelet-SRNet_A_Wavelet-Based_ICCV_2017_paper.pdf)这里的h high和h low分别代表着高通滤波器和低通滤波器，由小波基函数生成。图示里的小波基函数是选用的haar小波基函数。给定一张特征图x，沿列做高通滤波并下采样，再沿行分别做高通和低通滤波并下采样，就得到了子带D和H，分别代表着对角细节高频系数和水平细节高频系数。同样的可以得到V和A，分别代表垂直细节高频系数和近似低频细节系数。这样就得到了四个2倍下采样的小波子带图像。这就是离散的小波变换，同时，如果我们知道了子带系数，和滤波器系数，就可以执行小波逆变换操作，可以将四个子带恢复到X。

![image-20190413152759917](https://ws1.sinaimg.cn/large/006tNc79ly1g211f83oaej30go06vdiq.jpg)

利用这些性质，作者设计了小波域的上采样网络。上一阶段的Fen作为输入。首先使用低通滤波器得到特征图的近似细节分量。作者认为近似细节分量中包含了对于垂直和水平细节分量有用的信息，所以在计算这两个细节分量的时候，分别加入了近似细节分量。同样的，在计算对角细节分量的时候，前三个细节分量都有作用，所以，加入了前三个细节分量的信息。在第二和第三子带中没有连接，因为他们分别代表着垂直和水平细节，所以无法互相得到信息。
然后将这些小波系数分量分别输入到残差块中，在论文里作者将这部分称为非线性函数，就没有细讲，但是从公式来看，这应该也是一个小波滤波操作。这个阶段被称为自残差学习阶段。最后，是子带精炼阶段。在这个阶段中，相反，作者使用高频块去精炼低频块。对四个子带网络的输出作小波逆变换，就得到了2倍的上采样特征。

#### Architecture for magnification factor $2^j$X

![image-20190413152858094](https://ws2.sinaimg.cn/large/006tNc79ly1g211g89zemj30d809240i.jpg)
到目前为止，介绍的结构是2倍率的超分辨，为了得到更高倍数的超分辨，则通过**级联图像复原网络**的方法完成。损失函数则设计成每个阶段的损失的和。首先将一个高清图像通过三插值下采样分别得到j级的相应大小的图像。然后，通过对应元素的差的绝对值计算j级的损失。

#### 实验

![image-20190413153026425](https://ws1.sinaimg.cn/large/006tNc79ly1g211hrd5czj30go033dh4.jpg)
首先，作者将特征嵌入模块中的团结构模块替换成resnet 和 densenet，并保持上采样模块不变，可以看到在峰值噪声比和结构相似性两个衡量指标上均获得了优异的的结果。同样的，将特征嵌入模块保持不变，而将联合子带上采样替换成解卷积、子像素卷积或者是去掉原文中的小波子带联合学习，得到结果就是表三。证明了子带联合学习的有效性。

![image-20190413153043125](https://ws4.sinaimg.cn/large/006tNc79ly1g211i1p9bwj30dc07ctas.jpg)

下面是和其他两种小波卷积超分辨网络[wavelet-srnet](
http://openaccess.thecvf.com/content_ICCV_2017/papers/Huang_Wavelet-SRNet_A_Wavelet-Based_ICCV_2017_paper.pdf) 和 [waveletCNN](
https://www.sciencedirect.com/science/article/pii/S0167865517300879)。的比较以及峰值噪声比的值。上图是wavelet-srnet的结构，它同样主要的是包括一个特征嵌入模块和上采样模块，但最主要的差别就是，它没有将小波子带的信息联合学习。下图是CNNWSR的结构，它是一种直接用卷积网络将图像学习到小波系数，在恢复成高清图像的结构。

![image-20190413153106865](https://ws4.sinaimg.cn/large/006tNc79ly1g211igsc4hj30es05wgnh.jpg)

可以看出作者的结果是最好的。

![image-20190413153148795](https://ws3.sinaimg.cn/large/006tNc79ly1g211j6x2jbj30go02gaba.jpg)

最后和其他的一些sota的结构做对比的结果。这里作者又加入了srcliquenet+的对比结果，作者称为self-ensemble，我查了一下，应该是多种数据增强方法的集合。可以看出无论是否使用数据增强，作者的结果都非常优秀。

![image-20190413153419880](https://ws4.sinaimg.cn/large/006tNc79ly1g211lu2eh5j30ev0n2n7h.jpg)