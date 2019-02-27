---
layout: page
title: '计算机如何高效识别图像(译)'
categories: [tech]
tags: [translation]
excerpt: 2012年之前，深度神经网络只是机器学习领域中的死水泥潭。但随即Krizhevsky和他的多伦多大学团队向一个备受瞩目的图像识别竞赛提交了作品，准确率突破性地超越了以往任何技术。一夜之间，深度神经网络成为了图像识别的领衔技术。
---

> 原文 [How computers got shockingly good at recognizing images](https://arstechnica.com/science/2018/12/how-computers-got-shockingly-good-at-recognizing-images/)
> 作者 [Timothy B. Lee](https://arstechnica.com/author/timlee/)

> 图片如无标示亦出自原文

> 文末附有[专业词汇对照表](#专业词汇对照表)

![](https://cdn.arstechnica.net/wp-content/uploads/2018/12/neural-net-eyes-800x450.jpg)

如今，我在GooglePhotos里输入“沙滩”，就可以看到过去一段时间里我去过的各个沙滩。我并没有管理过照片，给它们打标签。实际上是Google根据照片内容识别出沙滩。这个看似普通的功能基于深度卷积神经网络技术，此技术允许软件以一种复杂的方式理解图像，这是以前的技术做不到的。

这些年来研究人员发现，随着他们建立更深层的网络，积累更大的数据集来训练它们，软件的准确性越来越高。这导致了对算力的需求几乎无法被满足，让英伟达(Nvidia)和AMD等GPU制造商大赚一笔。Google几年前开发了自己定制的神经网络芯片，其他公司争相效仿。

例如在特斯拉，深度学习专家Andrej Karpathy被任命为负责自动驾驶项目。这家汽车制造商在开发定制的芯片，以加速未来的自动驾驶版本中的神经网络操作。又例如苹果公司，最近几款的iPhone中的A11/A12芯片就包含“神经引擎”，可以加速神经网络运行，以提供更好的图像识别/语音识别应用。

那些我为了写本文而采访的专家们认为，如今深度学习的热潮可以追溯到一篇专业论文——AlexNet(以该文首要作者Alex Krizhevsky的昵称命名)。

“在我看来，AlexNet论文发表的2012年可谓里程碑。”机器学习专家、《智能机器如何思考》(How Smart Machines Think)作者Sean Gerrish如是说。

2012年之前，深度神经网络只是机器学习领域中的死水泥潭。但随即Krizhevsky和他的多伦多大学团队向一个备受瞩目的图像识别竞赛提交了作品，准确率突破性地超越了以往任何技术。一夜之间，深度神经网络成为了图像识别的领衔技术。其他研究人员很快地利用这项技术使图像识别精度飞跃发展。

在这篇文章中，我们将深入探讨深度学习。我会解释神经网络究竟是什么，它如何被训练，以及它为什么需要那么大的算力。然后我将解释一种特殊神经网络——深层卷积网络——为什么它特别擅长理解图像。别担心，我们会有很多配图。

## 简单的单神经元的例子

“神经网络”这个词可能比较含糊，让我们从一个简单例子开始吧。假设你想要一个根据红绿黄交通灯决定车子是否能走的神经网络。此神经网络可以用一个神经元来完成这项任务。

![](https://cdn.arstechnica.net/wp-content/uploads/2018/12/Screen-Shot-2018-12-18-at-8.53.12-AM.png)

这个神经元接收每个输入(1代表打开，0代表关闭)，乘以各自权重(weight)，并且把加权值(weighted value)相加。然后此神经元会加上偏置(bias)，这决定了神经元“激活(activate)”的阈值。此情况下如果输出值为正，我们认为此神经元“激活”了，否则没“激活”。这个神经元相当于不等式“green - red - 0.5 > 0”。如果计算结果为true，就意味着绿灯亮了，红灯灭了，车子可以通过。

在真正的神经网络中，虚拟神经元会有额外的一步。在加权值加上偏置求和后，神经元会调用非线性激活函数(Non-linear activation function)。一种流行的做法是sigmoid函数，一个S型函数，它总是产生一个0~1间的值。

![](https://gss3.bdstatic.com/-Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=33afcd8303f79052fb124f6c6d9abcaf/d009b3de9c82d158dfb4e7218a0a19d8bc3e426f.jpg)

在我们简单的交通灯模型中，使用激活函数并不会改变结果(但不能用0，例子中我们用0.5)。但激活函数的非线性是让神经网络模拟更复杂函数的关键。没有了激活函数，无论多复杂的神经网络都可以简化为它的输入的线性组合。而且线性函数不可能模拟复杂的真实现象。非线性激活函数可以让神经网络更接近任何数学函数。

## 一个神经网络样例

当然，有很多办法可以近似函数。神经网络之所以特别，是因为我们知道如何用一些算式，一堆数据，以及海量算力来“训练”它们。我们可以建造一个合适的通用神经网络来替代用人类程序员专门写来执行特定任务的神经网络。刷过一堆打过标签的数据后，它会修改神经网络本身，让自己能尽可能打出正确的标签。我们希望得出的神经网络具有通用性，能为不在训练数据中的样本也正确地打标签。

在AlexNet发表之前，这个努力已经开始了。1986年，三位研究人员发表了一篇关于反向传播(backpropagation)的里程碑式论文。该技术是一种有助于训练复杂神经网络的数学方法。

为了直观地了解反向传播如何工作，让我们来看看Michael Nielsen在他杰出的在线深度学习教程中描述的一个简单神经网络。目标是让神经网络从28X28像素的图片上正确地识别一个手写的数字(例如0，1，2)。

每个图片有28x28=784个输入值，每个都用0或1代表这个像素是深色还是浅色。Nielsen 构建了一个这样的神经网络:

![](https://cdn.arstechnica.net/wp-content/uploads/2018/08/tikz12-1.png)

插图里中层和右层每个圆圈代表我们上一段说的那种神经元。每个神经元为输入值加权求平均，然后加上偏置，然后调用一个激活函数。值得注意的是左层的圆圈不是神经元，它们代表的是输入值。虽然只画了8个输入圈，但实际上有784个输入——每个代表一个像素。

右层的10个神经元代表“亮灯”猜测不同的数字：假如是0，首个神经元应该被激活(其他神经元不激活)，假如是1则是第二个神经元被激活(其余的不激活)，如此类推。

每个神经元接收上一层所有神经元的输入。所以中层的15个神经元都是收784个输入值。它们每个都为784个输入值设置权重。这意味着在此层有15X784=11760个权重参数。同理，输出层有10个神经元，每个都从中层的15个神经元接收输入值，所以又多了15*10个权重参数。与此同时，整个网络25个神经元共有25个偏置。

## 训练神经网络

我们目标是调整这11935个参数，尽可能正确地得让代表对应手写数字的输出神经元亮灯。我们可以用一个叫MNIST的著名数据集来训练，它提供了60000个已经正确标注的28X28像素图像：

![](https://cdn.arstechnica.net/wp-content/uploads/2018/08/MnistExamples.png)
此图是 MNIST 数据集里60000个图像种的160个

Nielsen 示范了如何不依赖任何机器学习库，仅用74行正常Python代码来训练这个神经网络。训练从选取11935个随机的权重/偏置参数开始。软件将遍历每张图片样本，完成两步过程:

1. 根据输入图像和网络的当前参数，前馈步骤计算网络的输出值。

2. 反向传播步骤将计算结果与正确输出值的偏差，然后修改网络种的参数，以略微提高其在特定输入图像上的性能。

下例中，假设网络收到了这张图片：

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/seven.png)

如果网络经过良好调校，网络中“7”的输出值应该接近1，其他9个输出值应该都接近0。但假如收到此图时“0”值是0.8。那就太离谱了！训练算法会改变输入“0”输出神经元的各个权重，好让下次输入这张图片时，输出值会更接近0。

为此，反向传播算法会为各个权重参数算出一个误差梯度。这是一个衡量输出错误有多大程度被输入权重所被改变的维度。算法会依照这个梯度决定多大程度改变每个输入值的权重——梯度越大，这个参数修改越多。

换句话说，这个训练过程“教会”输出神经元更少地注意导向错误的输入方(即中层的神经元)，更多地注意正确方向的输入方。

算法一直为每个输出神经元重复此步骤。它将降低 "1," "2," "3," "4," "5," "6," "8," 和 "9" 神经元(除了神经元“7”)的输入方的权重，使输出神经元的值降低。输入值越大，反映此输入值的权重参数的误差梯度就越大，因此这个权重会减少越多。

同理，训练算法会增加输出神经元“7”的输入值的权重。令下次遇到这个图像时，此神经元会产生更大的输出值。同样的，输入值越大，加权后的增加值也越大，这使得输出神经元“7”在以后的几轮学习中更加关注这些输入。

接下来，算法需要对中层执行相同的计算：改变每个输入值的权重以减少网络的错误——同样，使“7”输出更接近1，其他输出更接近0。但每个中层神经元都会对向全部10个输出层神经元进行输出，这会导致两个方向的矛盾。

首先，中层的误差梯度，不仅仅取决于输入值，也同时取决于下一层的误差梯度。

这个算法之所以被称为反向传播，是因为网络里后层的误差梯度会反向传到前面层级(沿着算式中的关系链)并用来计算梯度。

同时，每个中层神经元也是全部10个输出层神经元的输入源。因此，训练算法计算的误差梯度，必须反映输入权重的变化如何影响*所有输出*的平均误差。

反向传播是一种爬坡算法:该算法的每一轮都会使输出结果更接近训练图像的正确结果，但只会更接近一点。随着算法经历越来越多的样本，它会“爬坡”到一个最优参数集，该参数集能够正确地分类尽可能多的训练样本。要获得较高的准确率，需要成千上万的训练样本，算法可能需要对训练集中的每个图像进行数十次循环，才能达到最优。

Nielsen展示了如何用74行Python代码实现上述所言。值得注意的是，使用这个简单程序训练的神经网络能够识别MNIST数据库中95%以上的手写数字。通过一些额外的改进，像这样一个简单的两层神经网络能够识别98%以上的数字。

## AlexNet的突破

你可能以为，上世纪80年代反向传播的发展会开启一段基于神经网络的机器学习的快速发展时期，但事实并非如此。在90年代和21世纪初，有一些人在研究这项技术。但对神经网络的热潮直到2010年代初才真正兴起。

我们可以在由斯坦福大学计算机科学家李飞飞组织的年度机器学习竞赛《ImageNet》的结果中看到这一点。在每年的比赛中，参赛者都会得到一组由100多万张训练图片组成的通用图片，每张图片上都手工标注着1000种可能的分类，比如“消防车”、“蘑菇”或“猎豹”。参赛者的软件根据其对未被纳入训练集的其他图像进行分类的能力进行评判。程序可以进行多次猜测，如果前五次对图像的猜测中有一次与人类选择的标签相匹配，则该软件被认为是成功的。

比赛从2010年开始举办，在前两届比赛中，深度神经网络并没有发挥主要作用。顶级团队使用了各种其他机器学习技术，但结果平平无奇。2010年获胜的那支球队，top5错误率(即五轮皆猜错)为28%。2011年度则是25%。

然后就到了2012年。这个来自多伦多大学的团队提交了一份参赛作品，不啻于将选手们从死水泥潭中拖出。该参赛作品即后来以主要作者Alex krizhevsky名字命名的AlexNet。通过使用深度神经网络，该研究小组获得了16%的top5错误率。当年最接近的竞对手错误率达到26%。

前文所说的手写识别网络有两层，25个神经元，以及将近12000个参数。AlexNet更大更复杂:8个可训练的层、65万个神经元和6000万个参数。训练这样规模的网络需要大量的算力，而AlexNet的设计就是利用现代GPU提供的大量并行算力。

研究人员想出了如何在两个GPU之间分配网络训练的工作，从而使他们的算力提高了一倍。尽管进行了积极的优化，使用2012年水平的硬件(两个Nvidia GTX 580 GPU，每个有3GB内存)对网络训练依然耗费了5到6天。

看看AlexNet结果的一些例子，对我们理解这是一个多么引人瞩目的突破很有帮助。以下是那篇论文的插图，展示了一些图片样本和AlexNet对应的top5分类猜测:

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/Screen-Shot-2018-10-12-at-3.08.44-PM-980x798.png)

AlexNet能够识别出第一张图片中包含了一只螨虫，尽管这只螨虫只是图片边缘的一小块。该软件不仅能正确识别美洲豹，它的其他先行猜测——美洲虎、猎豹、雪豹和埃及猫——都长得很像。AlexNet将蘑菇的图片标注为“伞菌”——蘑菇的一种。官方正确的标签“蘑菇”是AlexNet的第二选择。

AlexNet的“错误结果”同样令人惊讶。照片上，一只斑点狗站在樱桃后面，旁边写着“斑点狗”，而官方的标签是“樱桃”。AlexNet意识到这张照片上有某种水果——“葡萄”和“接骨木果”是它的前五种选择——但它没有完全意识到它们是樱桃。

给AlexNet展示一张马达加斯加猫在树上的照片，它会猜一系列的小型爬树哺乳动物。很多人(包括我)都会同样地猜错。这确实是一个引人瞩目的表现，表明软件可以识别各种方向和排布中的常见对象。深度神经网络迅速成为图像识别任务中最受欢迎的技术，机器学习的世界从此迈上快车道不回头。

ImageNet的发起人写道:“随着基于深度学习的方法在2012年取得成功，2013年的绝大多数参赛作品都使用了深度卷积神经网络。”这种模式持续了好几年，后来的优胜者都是建立在AlexNet团队开创的基本技术之上。到2017年，使用深度神经网络的参赛者将top5错误率降至3%以下。考虑到这项任务的复杂度，可以说这使得计算机比许多人更擅长这项任务。

这张来自ImageNet团队的条形图显示了每年top-5分类比赛中获胜团队的错误率。从2010年到2017年，错误率稳步下降。

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/Screen-Shot-2018-10-12-at-4.24.35-PM-980x577.png)

## 卷积网络的概念

从技术上讲，AlexNet是一个卷积神经网络。在本节中，我将解释卷积网络的作用，以及为什么这项技术对现代图像识别算法至关重要。

我们之前研究的简单手写识别网络是全连接的:第一层的每个神经元都是第二层神经元的输入源。此结构比较适合相对简单的任务，例如识别28×28像素图像中数字。但进一步扩展就很难了。

在MNIST手写数字数据集中，字符总是居中的。这大大简化了训练，因为这意味7在图像的顶部和右侧总是有一些黑色像素，而左下方总是白色的。0总是中间白块，四周的像素则较暗。一个简单的、全连接的网络可以相当容易地检测这些类型的模式。

但是假如你想建立一个神经网络来识别可能位于大图中的任何位置的数字。一个全连接的网络不擅长于此，因为它没有一种有效的方法来识别位于图像不同部分的形状之间的相似性。如果你的训练集的左上角恰好有大部分的7，那么你会得到一个网络，它比图像中其他地方更善于识别左上角的7。

从理论上讲，你可以通过确保你的训练集在每个可能的像素位置上都有很多数字的样本来解决这个问题。但在实践中，这将是巨大的浪费。随着图像的大小和网络的深度的增加，连接的数量——也就是输入权重参数的数量——将会激增。你需要更多的训练图像和算力来达到足够的准确性。

当神经网络学习识别图像中某个位置的形状时，它应该能够将这种学习应用于识别图像中其他位置的类似形状。卷积神经网络为这个问题提供了一个优雅的解决方案。

“这就像拿一个模板或模式，然后将其与图像上的每一个点进行匹配，”人工智能研究员唐杰(音)说。“你有一个狗的模板轮廓，然后发现右上角和你的模板基本匹配——那里有条狗吧?”如果没有，可以稍微移动一下模板，遍历整张图片。狗出现在在图像哪个位置并不重要。模板会匹配到它。你不会想让网络的每个子部分都学习自己单独的狗识别器。

试想一下，如果我们把一幅大图像分成28*28像素的小块。然后，我们可以将每个小块输入到之前所述的全连接的手写识别网络中。如果这些小块中的“7”输出至少有一个被点亮，就意味着整个大图中可能有7的存在。这就是卷积网络的本质。

## 卷积网络在AlexNet中如何工作?

在卷积网络中，这些“模板”被称为特征检测器(feature detector)，它们所观察的区域被称为接受域(receptive field)。真正的特征检测器的接受域往往比边长28像素小得多。在AlexNet中，第一个卷积层具有接受域为11×11像素的特征检测器。它随后的卷积层有3到5个单位宽的接受域。

当特征检测器扫过输入图像时，它会生成一个特征图(feature map):一个二维网格，表示该检测器被图像的不同部分激活的强度。卷积层通常有多个特征检测器，每个检测器扫描输入图像以寻找不同的模式。在AlexNet中，第一层有96个特征检测器，生成了96个特征图。

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/Screen-Shot-2018-10-12-at-4.45.46-PM-980x384.png)

为了更具体地说明这一点，以下是经过网络训练的AlexNet第一层中的96个特征检测器所学习的视觉模式的可视化表示。特征检测器可以寻找水平或垂直的线条、从亮到暗的渐变、棋盘图形和其他形状。

彩色图像用像素分布格式表示，每个像素有三个值：红值、绿值和蓝值。AlexNet的第一层会将红绿蓝三位表示的像素归纳为96个类型值之一。图像中的每个“像素”(指第一层分割的单元)有96种可能的值，对应96种特征检测器。

在本例中，这96个值中的第一个值指示图像中的某个特定点是否与此模式匹配:

![](https://cdn.arstechnica.net/wp-content/uploads/2018/11/Screen-Shot-2018-11-12-at-5.15.46-PM.png)

第二个值指示某个特定点是否与此模式匹配:

![](https://cdn.arstechnica.net/wp-content/uploads/2018/11/Screen-Shot-2018-11-12-at-5.15.52-PM.png)

第三个值表示某个特定点是否与此模式匹配:

![](https://cdn.arstechnica.net/wp-content/uploads/2018/11/Screen-Shot-2018-11-12-at-5.16.01-PM.png)

如此类推，在AlexNet的第一层中，其他93个特征检测器也是如此。第一层将图像的每个“像素”用一种新表现形式输出——表示为96个值(类似96进制？)的向量(我稍后将解释，这个新表示形式也被按比例缩小四倍)。

这是AlexNet的第一层。接下来是另外四个卷积层，每个层的输入都是前一层的输出。

正如我们所看到的，第一层检测基本的模式，如水平和垂直的线，光到暗的变化，以及曲线。然后，第二层使用第一层的结果作为构建块来检测稍微复杂一些的形状。例如，第二层可能有一个特征检测器，它通过组合第一层特征检测器的输出来查找曲线，从而查找圆。第三层通过结合第二层的特征找到更复杂的形状。第4层和第5层可以找到更复杂的模式。

研究人员Matthew Zeiler和Rob Fergus在2014年发表了一篇出色的论文，为可视化类似ImageNet的五层神经网络所识别的模式提供了一些有用的方法。

在下列插图中，每张图片(第一张除外)都有两个部分。在右边，你将看到一些缩略图图像，它们高度匹配了特定的特征检测器。它们9个分为一组，每组对应一个不同的特征检测器。在左边是一个地图，它显示了缩略图图像中的哪些特定像素产生了高度匹配。你可以在第5层中最显著地看到这一点，因为有一些特性检测器可以高度匹配为狗、企业标识、独轮车轮子等。

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/Screen-Shot-2018-10-12-at-11.13.38-PM.png)

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/Screen-Shot-2018-10-12-at-11.03.55-PM.png)

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/Screen-Shot-2018-10-12-at-11.05.23-PM.png)

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/Screen-Shot-2018-10-12-at-11.05.55-PM.png)

![](https://cdn.arstechnica.net/wp-content/uploads/2018/10/Screen-Shot-2018-10-12-at-11.06.04-PM.png)

浏览这些图像，你可以看到每一层都能够识别比前一层更复杂的模式。第一层识别简单的像素模式，看起来也并不十分相像。第二层识别纹理和简单的形状。在第三层，我们可以看到可识别的形状，如汽车轮子和红橙色的球体(可能是西红柿、瓢虫或其他东西)。

第一层的感受域为11X11单位大小，而后面的层的感受域大小则为3X3单位到5X5单位不等。但是请记住，后面的这些层接收的是前面的层生成的特征图，这些特征图中的每个“像素”表示原始图像中的n个像素。所以每一层的感受域都包含了原始图像中比前一层更大的部分。这就是为什么后期图层的缩略图看起来比早期图层更复杂的原因之一。

![](https://pic3.zhimg.com/80/v2-5378f1dfba3e73dedafdc879bbc4c71e_hd.png)
感受域示意图，from[深度神经网络中的感受野](https://zhuanlan.zhihu.com/p/28492837)

网络的第五层也是最后一层能够识别这些图像中最显眼的大尺度元素。例如这张图片，我从上面第5层图片的右上角截取的:

![](https://cdn.arstechnica.net/wp-content/uploads/2018/11/Screen-Shot-2018-11-12-at-3.40.36-PM-980x494.png)

右边的9张图片看起来可能不太相似。但是如果你看一下左边的9张热图，你会发现这个特定的特征检测器并没有聚焦于每张图像前景中的物体。相反，它关注的是每个图像背景中的绿色部分!

显然，如果你要识别的类别之一是“草”，那么用“草检测器”就合适了，但是它同时也可能检测其他东西。在五个卷积层之后，AlexNet有三个层是全连接的，就像我们手写识别网络中的层一样。这些层会利用第五卷积层生成的每个特征图，尝试将图像分类为1000个类别中的一个。

所以如果一幅画的背景是草，那么它更有可能是野生动物。另一方面，如果这幅画的背景是草，那么它不太可能是一幅室内家具的画。其他位于第五层的特征探测器提供了丰富的信息可用于猜测照片内容。网络的最后几层将这些信息综合起来，对整个图片所描绘的内容进行有理据的猜测。

## 卷积层的独到之处:共享输入权重

我们已经看到卷积层中的特征检测器执行了牛逼的模式识别，但是到目前为止，我还没有解释卷积网络实际上是如何工作的。

卷积层是一层神经元。像任何神经元一样，这些神经元取其输入的加权平均值，然后应用一个激活函数。用我们讨论过的反向传播技术来训练当中的参数。

但与上面的神经网络不同，卷积层没有全连接。每个神经元只接受前一层神经元的一小部分输入。更重要的是，卷积网络中的神经元共享输入权重。

让我们放大AlexNet的第一个卷积层中的第一个神经元。这一层的接受域大小是11×11像素,所以第一个神经元接收某角落的11×11像素。这个神经元接受这121个像素的输入，每个像素有三个值——红、绿、蓝。神经元总共有363个输入。和任何神经元一样，这个神经元取363个输入值的加权平均值，然后应用一个激活函数。因为它有363个输入值，它也需要363个输入权重参数。

AlexNet第一层中的第二个神经元与第一个神经元非常相似。它同样接收11×11像素,但它的接受域是从第一个神经元的接受域平移4个像素开始。这就在两个接受域之间产生了7个像素的重叠，这就避免了遗漏两个神经元之间的有趣模式。第二个神经元同样接收描述11×11像素的363个值,每个值乘以权重参数,求和,并调用一个激活函数。

特殊的是，第二个神经元没有自己的一组363个输入权重，而是使用与第一个神经元相同的输入权重。第一个神经元的左上角像素使用与第二个神经元的左上角像素相同的输入权重。所以这两个神经元在寻找完全相同的模式。它们的接受域之间有4个像素的偏移。

当然实际远不止两个神经元——图片分为55×55格共3025个神经元。这3025个神经元中的每一个都使用与开始两个神经元相同的363个输入权重。所有这些神经元共同“扫描”图像中可能存在的特定模式，组成一个特征检测器。

请记住，AlexNet的第一层有96个特性检测器。我刚才提到的3025个神经元组成了96个特征探测器中的一个。其他95个特征探测器则是由各自的一组3025个神经元组成。每组3025个神经元与组内其他神经元共享其363个输入权重，但不是和其他95个特征检测器的神经元共享。

卷积网络的训练使用与训练全连接网络相同的基础反向传播算法，但是卷积结构使得训练过程更加高效。

“使用卷积非常有用，因为可以重用参数，”机器学习专家和作家Sean Gerrish告诉Ars(本文原作者机构)。这大大减少了网络需要学习的输入权重的数量，这使得网络可以用更少的训练样本产生更好的结果。

从图像的一个部分学到的识别模式，可以转化用于识别其他图像的其他位置的相同模式。这使得网络可以通过更少的训练实例来实现高性能。

## 深度卷积网络威名远扬

AlexNet的论文在学术机器学习社区引起了轰动，它的重要性也很快得到了业界的认可。谷歌对这项技术特别感兴趣。

2013年，谷歌收购了AlexNet论文作者创办的一家初创公司。他们利用这项技术为谷歌照片添加了一个新的图像搜索功能。谷歌的Chuck Rosenberg写道:“我们直接从一个学术研究实验室进行了前沿研究，并在短短6个多月的时间里启动了它。”

与此同时，谷歌2013年的一篇论文描述了它如何使用深度卷积网络从谷歌街景(Google Street View )的照片中读取地址码。“我们的系统帮助我们从街景图像中提取了近1亿个实体街道编号，”作者写道。

研究人员发现，随着神经网络的发展，其性能不断提高。谷歌街景团队写道:“我们发现这种方法的性能随着卷积网络的深度而提高，在我们训练层级最多的架构中，性能最好。我们的实验表明，更深层次的架构可能获得更好的准确性，但效率会逐渐降低。”

所以在AlexNet之后，神经网络变得越来越深。一个谷歌的团队在2014年ImageNet竞赛中提交的作品获得了优胜——就在2012年AlexNet获胜的两年后。和AlexNet一样，它也是基于一个深度卷积神经网络，但是谷歌使用了一个更深层的22层网络，从而获得了6.7 %的top5错误率——这比AlexNet的16%错误率有了很大的提高。

不过，更深层的网络只对大型训练集有用。基于这个原因，Gerrish认为ImageNet数据集和竞争对深度卷积网络的成功起到了关键作用。大家还记得吗?ImageNet竞赛给了参赛者一百万张图片，要求他们从1000个不同的类别中选择一个。

Gerrish说:“拥有100万张图片来训练你的神经网络，意味着每个类别有1000张图片(1m/1000 = 1000)。”他说，如果没有这么大的数据集，“你训练网络的参数就多得多。”

近年来，人们聚焦于收集更大的数据，以便训练更深入、更准确的网络。这是自动驾驶汽车公司如此专注于在公共道路上积累里程的一大原因——来自测试的图像和视频被送回总部，用于培训公司的图像识别网络。

## 深度学习计算的繁荣

更深层次的网络和更大的训练集可以提供更好的性能，这一发现对更多的算力产生了无法满足的需求。AlexNet成功的一个重要原因是意识到，神经网络训练涉及矩阵运算，可以使用显卡高度并行的算力高效地执行这些运算。

“神经网络是可并行的，”机器学习研究员唐杰说。显卡本来是为视频游戏提供大规模并行算力的，结果非常适合神经网络使用。

“GPU的核心运算是极快的矩阵乘法，最终成为神经网络的核心运算，”唐说。

这为英伟达(Nvidia)和AMD这两家行业领先的GPU制造商带来了滚滚财源。这两家公司都致力于开发适应机器学习应用程序独特需求的新芯片，而人工智能应用程序目前在这些公司的GPU销售中占据了相当大的比例。

2016年，谷歌公开了一个定制芯片，称为张量处理单元(TPU)，专门用于神经网络操作。谷歌2017年写道，虽然他们早在2006年就考虑过为神经网络构建专用集成电路(ASIC)，但2013年这种情况变得紧迫起来。

“就在那时，我们意识到神经网络快速增长的计算需求可能要求我们将运营的数据中心数量增加一倍。”最初，TPU仅允许谷歌自己的专有服务使用，但后来，谷歌开始允许任何人通过它的云计算平台使用这项技术。

当然，谷歌并不是唯一一家致力于人工智能芯片的公司。一个小例子是，iPhone最新版本的芯片就包括一个为神经网络操作优化的“神经引擎”。英特尔正在开发自己的针对深度学习的优化芯片系列。特斯拉最近宣布放弃英伟达的芯片，转而采用自制神经网络芯片。亚马逊据说也在开发自己的AI芯片。

## 为何深层神经网络难以理解

我已经解释了深度神经网络是如何工作的，但我还没有真正解释它们的工作何以那么出色。大量的矩阵运算可以让计算机区分美洲虎和猎豹，接骨木和醋栗，这真令人惊讶。

但也许神经网络最值得注意的地方是它们做不到的事。卷积使神经网络能够理解平移——它们可以判断一幅图像的右上角的模式是否与另一幅图像的左上角的模式相似。

但除此之外，卷积网络并没有真正的理解几何图形。如果图像旋转45度或放大2倍，他们就无法识别出这两幅图像是否相似。卷积网络并没有尝试去理解三维物体的结构，而且它们也不能识别不同的光照条件。

然而，深度神经网络能够识别狗的图片，无论它们是正面还是侧面拍摄的，无论它们是占据了图片的一小部分还是很大一部分。

神经网络是怎么做到的?事实证明，有了足够的数据，暴力统计方法就足以完成这项工作。卷积网络的设计初衷并不是“想象”如果从不同的角度或在不同的环境下，一张特定的图像会是什么样子，但是有了足够多的标记样本，它可以通过纯粹的重复来学习所有可能的排列。

有证据表明，人类的视觉系统实际上是以类似的方式工作的。我们来看这两张图片(在看第二张图片之前，确保你仔细看了第一幅图片):

![](https://cdn.arstechnica.net/wp-content/uploads/2018/11/face_upside_down.png)

![](https://cdn.arstechnica.net/wp-content/uploads/2018/11/face_rightside_up.png)

显然，这张照片的创作者把人像的眼睛和嘴巴颠倒过来，再把整张照片翻过来。当你把图像倒过来看时，它看起来相对正常，因为人类的视觉系统已经习惯了在这个方向上看眼睛和嘴。但是当你看后者时，就会发现人像面容变得畸形。

这表明人类视觉系统依赖于一些与神经网络相同的粗糙模式匹配技术。如果我们看到的东西几乎都是同一个方向的——就如人类的眼睛——我们就能更好地识别出它们通常的方向。

神经网络善于利用图片中的所有环境来理解它所显示的内容。例如，汽车通常出现在道路上。裙子通常要么出现在女性的身体上，要么挂在衣橱里。飞机要么在蓝天的衬托下出现，要么在跑道上滑行。没有人明确地教神经网络这些相关性，但有足够多的标记样本，网络可以自动学习它们。

2015年，谷歌的一些研究人员试图通过“反向操作”来更好地理解神经网络。他们不是用图像来训练神经网络，而是用训练过的神经网络来修改图像。

比如，他们从一幅包含随机噪声的图像开始，然后逐渐修改它，使其强烈地“点亮”神经网络的输出之一——要求神经网络高效地“绘制”一个它被训练识别的类别。在一个有趣的例子中，他们用一个被训练用来识别哑铃的神经网络来画图。

![](https://cdn.arstechnica.net/wp-content/uploads/2018/11/dumbbells.png)

研究人员在研究报告中写道:“这些是哑铃没错，但似乎都画出了肌肉发达的举重运动员来举哑铃。”

乍一看，这似乎很奇怪，但实际上这与人类的行为并没有太大的不同。如果我们在一幅图像中看到一个小的或模糊的物体，我们就会观察周围的环境，寻找图片中可能发生的事情的线索。很明显，人类通过不同的方式对图像进行推理，利用我们对周围世界复杂的概念理解。

总而言之，深层神经网络擅长于图像识别，是因为它们充分利用了图片中显示的所有环境，这与人类的识别方式并没有太大的不同。(完)

## 专业词汇对照表

- neural network 神经网络
- computing power 算力
- convolutional network 卷积网络
- neuron 神经元
- weight 权重
- bias 偏置
- activation function 激活函数
- Non-linear activation function 非线性激活函数
- light up 亮灯(即匹配)
- layer 层
- receptive field 感受域
- backpropagation 反向传播
- top-five error rate top5错误率
- error gradient 误差梯度
- matrix 矩阵
- fully connected layer 全连接层
- pattern 模式
- feature detector 特征检测器
- brute-force statistical 暴力统计