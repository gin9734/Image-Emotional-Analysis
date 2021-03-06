Image-Emotional-Analysis
====

基于深度学习的图像情感分类系统<br>
作者：侯逊<br>
----

# 项目背景
深度学习作为目前图像情感分类的一个重要手段，比传统机器学习方法具有明显优势。由于情感图像分类是一个相对复杂抽象的研究领域，本文采用Inception网络提取的特征与传统特征结合的方法构建分类模型，并将图像分为15个情感极性，分别为：`Happy`、`Calm`、`Sad`、`Scared`、`Bored`、`Angry`、`Annoyed`、`Love`、`Excited`、`Surprised`、`Optimistic`、`Amazed`、`Ashamed`、`Disgusted`、`Pensive`等。该方法比仅使用Inception网络和仅使用传统特征进行图像情感分类的准确率都要高，证明了本课题创新方法的可行性。本课题基于Tumblr社交平台提供的原始数据，完成以下的研究工作：<br>

(1) 对原始数据预处理，即筛选无关图像、不一致标签图像和特征不明显图像。<br>

(2) 提出了传统特征与深度学习特征结合的方法，即使用`预训练的Inception网络`提取图像的情感层特征。提取图像的`HSV局部直方图`、`GLCM`、`HOG`、`LBP`特征，并将其与高层次特征拼接起来，作为该图像的整体特征。<br>

(3) 提出了使用神经网络作为分类器的方法。网络结构从下到上分别为：全连接层、Dropout层、全连接层，网络最顶层的softmax输出层给出图像的15类情感概率分布。最后用训练集和验证集训练出良好模型并保存。<br>

(4) 完成对本系统的需求分析、系统设计、系统实现、系统测试等工作。本课题采用Django实现可视化界面，用户可在本系统进行注册、登录、登出、上传图片、预测图像情感、管理用户等操作。<br>

## 系统框架
![系统框架如图所示](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/structure.png)

## 数据获取
社交平台：[Tumblr](https://tumblr.zendesk.com/hc/zh-cn)<br>
API: [PyTumblr](https://pypi.org/project/PyTumblr/)<br>

## 图像预处理
* 人工排除无关图像和不一致标签图像。无关图像包括广告、无效图像等。不一致标签图像反映出用户发表的图像和文本所表达的情感存在不一致现象，通常情况下文本所表达的情感是用户的实时发帖情感，而图像往往不能准确的表达出该情感。如图所示，该图展现了一位正在哭泣的女人，然而文本部分表明这位女人见到了奥巴马喜极而泣；下图描述的是一位微笑的男孩，但根据文本，事实上是这位小男孩不幸去世。对于这种不一致标签的图像，也需要进行人工检查筛选。<br>
![不一致标签图像](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/noise1.png)<br>

* 特定算法移除特征不明显图像。特征不明显图像的特点是图像颜色过于单一、在纯色背景下附有大量的文字。如图所示，虽然从图片中的文字能看出用户发帖时的情感是积极、乐观的，但从图像本身分析，该图像并不具备表达积极情感的特征。<br>
![特征不明显图像](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/noise2.jpg)<br>
![特征不明显图像筛选算法](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/algorithm1.png)<br>
## 图像特征提取
### 传统图像特征
* HSV颜色直方图。HSV颜色空间由色调、饱和度和亮度组成。色调H表示颜色种类，饱和度表示颜色深浅，亮度V表示颜色明暗程度。HSV颜色空间是最符合人眼视觉特性的颜色模型，在图像分类、检索中，应用这种颜色模型会更适合用户的视觉判断。<br>
以全局颜色直方图作为图像颜色特征，仅包含了图像的整体颜色分布，缺少了图像的空间信息。因此，为反映出颜色的空间分布关系，本系统引入了HSV局部特征直方图，即将整幅图片四等分块，分别计算每一块的HSV颜色直方图，最后将四个直方图横向拼接成局部颜色直方图，作为该图像的颜色特征。<br>
![hsv](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/hsv.png)<br>
* 灰度共生矩阵。灰度共生矩阵的定义为：任取图像中的一组点对(x，y)和(x+a，y+b)，设该点对的灰度值对为(p，q)，若灰度值有n种取值，则(p，q)的组合共有 n2种。对于整个图像像素矩阵，统计出每一种(p，q)值出现的次数并建立概率矩阵，这样的矩阵称为灰度共生矩阵。当a=1，b=0时，以0度方向统计像素对；当a=0，b=1时，以90度方向统计像素对；当a=1，b=1时，以45度方向统计像素对。<br>
本系统定义最大灰度级数为16，并分别计算出图像0度、45度、90度扫描的灰度共生矩阵，并计算每个矩阵的角二阶矩、熵、对比度、均匀度这四个参数，三组参数拼接成一组12维的特征向量，作为该图像的纹理特征。<br>

* HOG和LBP特征。HOG（方向梯度直方图）计算统计图像的局部梯度方向直方图作为特征。LBP（局部二值模式）提取图像的局部纹理特征，其具有旋转不变性和灰度不变性等优点。<br>

### 预训练Inception网络特征
本系统使用预训练的InceptionV3网络提取图像高层次特征。以往的神经网络大多是通过增加网络层数来获得更好的训练效果，但网络深度的增加会带来过拟合、计算开销庞大、梯度消失、梯度爆炸等问题。InceptionV3则通过更改网络宽度、深度、卷积核等提高模型效果，可以更高效地利用计算资源，提取到更多的特征，从而提升训练结果。<br>

实现库：[Keras](https://keras.io/)<br>

### 特征拼接
本模块提出了传统特征与预训练网络特征结合的创新方法。即将四种传统特征和Inception网络提取的高层次特征拼接在一起，作为图像的整体特征。特征拼接后，每个图像对应一个4394维的特征向量。<br>

## 模型训练
本系统本搭建了一个三层神经网络，网络结构如图所示。该网络由两个全连接层和一个Dropout层组成，接收形状为(samples，4394)的numpy向量，返回形状为(samples，15)的numpy向量，代表每个图像对应的15类情感概率分布。模型搭建后需要进行编译、训练，由于本系统处理的是多分类问题，需要用rmsprop优化器和交叉熵损失函数配置模型，在训练过程中记录验证精度。最后反复训练得到最优模型，并保存到本地。<br>
![模型网络结构](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/model.png)<br>

## 数据库设计
### 用户表
![用户表](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/user_table.png)<br>
### 图像表
![图像表](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/img_table.png)<br>
## Django框架
Django是一个基于python的Web框架。Django框架能自动处理用户和控制器的交互部分，其核心是模型(Model)、模板(Template)和视图(Views)，因此Django采用的是MTV模式。<br>
Django的优势在于它能简单、快速实现一个具有前端、后端、数据库的网站，通过Django内置函数可以很容易地实现用户与系统交互、业务逻辑、数据库搭建等模块。同时Django内置许多功能齐全的插件，如验证码等，具有很强的可扩展性。<br>
![Django](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/django.png)<br>

## 预测结果
![test1](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/test1.png)<br>
![test2](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/test2.png)<br>
![test3](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/test3.png)<br>
![test4](https://github.com/HouXun/Image-Emotional-Analysis/raw/master/pics/test4.png)<br>




