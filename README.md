# 安全AI挑战者计划第一期 - 人脸识别对抗

*队伍：我永远喜欢阿尔托莉雅*

## 环境

+ python3
+ CUDA 9.0
+ Tensorflow 1.7

## 说明

+ securityAI_round1_images内的图片已经移除，请自行下载放入，链接为：https://pan.baidu.com/s/1zr9aiqsZQrIDdSNSsH1o9g ， 提取码：4296
+ models文件夹内的模型已移除，请自行下载放入，链接为：https://pan.baidu.com/s/1du8U1cnvCzO9ZPJ2v4Ky9w ，提取码：ru81
+ lfw文件夹内图片已移除，请自行下载放入（注意确保其子文件夹为new112），链接为：https://pan.baidu.com/s/1Lqmvo40F4tKeaqeIY5CW3w ，提取码：lg16

## 解题及算法思路

本次赛题设计背景为对已在工业界有成熟应用的人脸识别系统发起攻击，按攻击分类是黑盒无目标攻击

### 1.信息收集

了解主流开源人脸识别项目如face ID， insightface，facenet等，由赛题要求可知所有测试数据均使用MTCNN对齐并且缩放到112X112，经过细致观察，发现使用insightface项目中提供的MTCNN align结果和所提供数据非常吻合，故主要关注insightface项目，且考虑使用其中提供的几种网络作为攻击算法的backbone。

### 2.模型准备

insightface中提供有一些网络及其对应的预训练模型，但是由于insightface原项目基于mxnet，而在mxnet中编写攻击算法并不方便，故使用mmdnn将其中的Resnet100，Resnet50，Resnet34以及Mobilenet都转换为Tensorflow格式的模型（转换Mobilenet时会报错，需要手工修改MMdnn的代码），并且生成相应的网络结构代码，在所有网络结构代码中将KitModel的init方法中的data placeholder注释掉，改为从外部传入（方便后期做模型ensemble时求梯度）。

### 3.数据集

采用LFW作为数据集，统一使用insightface中的align_lfw.py对齐到112X112

### 3.Loss定义

对每张人脸图片进行攻击时均根据赛题提供的CSV找到对应的人在LFW数据集中的所有图片，将这些图片作为一个batch作为各模型的输入，得到的输出形如[图片数量,512]或者[图片数量,128] （以下称为victim）。loss直接采用victim与attack_image（当前正被攻击的图片）间的欧氏距离。

### 4.模型Ensemble

选用前面转换的4个模型作为backbone，由于Mobilenet输出的embedding大小为128，与其他三个Resnet模型的输出大小不同，故基于loss做ensemble（即算出4个模型的loss求平均）

### 5.攻击

采用I-FGSM，每步攻击的步长为1，当像素的修改量大于正负25时，将得到的结果clip到25以内，当4个模型的平均loss大于2或达到最大迭代步数时停止，在攻击前，使用MTCNN进行人脸的特征点检测，如果MTCNN未侦测到人脸特征，则实行全图的攻击，如成功检测到人脸特征点，则只取特征点周围一定范围的像素进行攻击，这样取得的攻击效果与全图攻击没有明显差别，但是可以较大幅度地减小攻击成功所需的扰动。

在平均loss达到2的情况下：

![7.44](7.44.jpg)
![3.77](3.77.jpg)

全图攻击扰动为7.44  检测特征点后扰动为3.77

## 参考及引用

+ [MMdnn](https://github.com/microsoft/MMdnn)
+ [insightface](https://github.com/deepinsight/insightface)
+ [facenet](https://github.com/davidsandberg/facenet)
+ [LFW](http://vis-www.cs.umass.edu/lfw/)
