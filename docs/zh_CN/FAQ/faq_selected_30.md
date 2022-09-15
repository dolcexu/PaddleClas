# FAQ

## 写在前面

* 我们收集整理了开源以来在 issues 和用户群中的常见问题并且给出了简要解答，旨在为图像分类的开发者提供一些参考，也希望帮助大家少走一些弯路。

* 图像分类领域大佬众多，模型和论文更新速度也很快，本文档回答主要依赖有限的项目实践，难免挂一漏万，如有遗漏和不足，也希望有识之士帮忙补充和修正，万分感谢。


## PaddleClas 常见问题汇总

* [1. 图像分类 30 个问题](#1)
    * [1.1 基础知识](#1.1)
    * [1.2 模型训练相关](#1.2)
    * [1.3 数据相关](#1.3)
    * [1.4 模型推理与预测相关](#1.4)
* [2. PaddleClas 使用问题](#2)


<a name="1"></a>
## 1. 图像分类 30 个问题

<a name="1.1"></a>
### 1.1 基础知识

>>
* Q: 图像分类领域常用的分类指标有几种
* A:
    * 对于单个标签的图像分类问题（仅包含 1 个类别与背景），评估指标主要有 Accuracy，Precision，Recall，F-score 等，令 TP(True Positive)表示将正类预测为正类，FP(False Positive)表示将负类预测为正类，TN(True Negative)表示将负类预测为负类，FN(False Negative)表示将正类预测为负类。那么 Accuracy=(TP + TN) / NUM，Precision=TP /(TP + FP)，Recall=TP /(TP + FN)。
    * 对于类别数大于 1 的图像分类问题，评估指标主要有 Accuary 和 Class-wise Accuracy，Accuary 表示所有类别预测正确的图像数量占总图像数量的百分比；Class-wise Accuracy 是对每个类别的图像计算 Accuracy，然后再对所有类别的 Accuracy 取平均得到。

>>
* Q: 怎样根据自己的任务选择合适的模型进行训练？
* A: 如果希望在服务器部署，或者希望精度尽可能地高，对模型存储大小或者预测速度的要求不是很高，那么推荐使用 ResNet_vd、Res2Net_vd、DenseNet、Xception 等适合于服务器端的系列模型；如果希望在移动端侧部署，则推荐使用 MobileNetV3、GhostNet
    等适合于移动端的系列模型。同时，我们推荐在选择模型的时候可以参考[模型库](../algorithm_introduction/ImageNet_models.md)中的速度-精度指标图。

>>
* Q: 如何进行参数初始化，什么样的初始化可以加快模型收敛？
* A: 众所周知，参数的初始化可以影响模型的最终性能。一般来说，如果目标数据集不是很大，建议使用 ImageNet-1k 训练得到的预训练模型进行初始化。如果是自己手动设计的网络或者暂时没有基于 ImageNet-1k 训练得到的预训练权重，可以使用 Xavier 初始化或者 MSRA 初始化，其中 Xavier 初始化是针对 Sigmoid 函数提出的，对 RELU 函数不太友好，网络越深，各层输入的方差越小，网络越难训练，所以当神经网络中使用较多 RELU 激活函数时，推荐使用 MSRA 初始化。

>>
* Q: 针对深度神经网络参数冗余的问题，目前有哪些比较好的解决办法？
* A: 目前有几种主要的方法对模型进行压缩，减少模型参数冗余的问题，如剪枝、量化、知识蒸馏等。模型剪枝指的是将权重矩阵中相对不重要的权值剔除，然后再重新对网络进行微调；模型量化指的是一种将浮点计算转成低比特定点计算的技术，如 8 比特、4 比特等，可以有效的降低模型计算强度、参数大小和内存消耗。知识蒸馏是指使用教师模型(teacher model)去指导学生模型(student model)学习特定任务，保证小模型在参数量不变的情况下，性能有较大的提升，甚至获得与大模型相似的精度指标。

>>
* Q: 怎样在其他任务，如目标检测、图像分割、关键点检测等任务中选择比较合适的分类模型作为骨干网络？
* A: 在不考虑速度的情况下，在大部分的任务中，推荐使用精度更高的预训练模型和骨干网络，PaddleClas 中开源了一系列的 SSLD 知识蒸馏预训练模型，如 ResNet50_vd_ssld, Res2Net200_vd_26w_4s_ssld 等，在模型精度和速度方面都是非常有优势的，推荐大家使用。对于一些特定的任务，如图像分割或者关键点检测等任务，对图像分辨率的要求比较高，那么更推荐使用 HRNet 等能够同时兼顾网络深度和分辨率的神经网络模型，PaddleClas 也提供了 HRNet_W18_C_ssld、HRNet_W48_C_ssld 等精度非常高的 HRNet SSLD 蒸馏系列预训练模型，大家可以使用这些精度更高的预训练模型与骨干网络，提升自己在其他任务上的模型精度。

>>
* Q: 注意力机制是什么？目前有哪些比较常用的注意力机制方法？
* A: 注意力机制(Attention Mechanism)源于对人类视觉的研究。将注意力机制用在计算机视觉任务上，可以有效捕捉图片中有用的区域，从而提升整体网络性能。目前比较常用的有 [SE block](https://arxiv.org/abs/1709.01507)、[SK-block](https://arxiv.org/abs/1903.06586)、[Non-local block](https://arxiv.org/abs/1711.07971)、[GC block](https://arxiv.org/abs/1904.11492)、[CBAM](https://arxiv.org/abs/1807.06521) 等，核心思想就是去学习特征图在不同区域或者不同通道中的重要性，从而让网络更加注意显著性的区域。

<a name="1.2"></a>
### 1.2 模型训练相关

>>
* Q: 使用深度卷积网络做图像分类，如果训练一个拥有 1000 万个类的模型会碰到什么问题？
* A: 因为 FC 层参数很多，内存/显存/模型的存储占用都会大幅增大；模型收敛速度也会变慢一些。建议在这种情况下，再最后的 FC 层前加一层维度较小的 FC，这样可以大幅减少模型的存储大小。

>>
* Q: 训练过程中，如果模型收敛效果很差，可能的原因有哪些呢？
* A: 主要有以下几个可以排查的地方：（1）应该检查数据标注，确保训练集和验证集的数据标注没有问题。（2）可以试着调整一下学习率（初期可以以 10 倍为单位进行调节），过大（训练震荡）或者过小（收敛太慢）的学习率都可能导致收敛效果差。（3）数据量太大，选择的模型太小，难以学习所有数据的特征。（4）可以看下数据预处理的过程中是否使用了归一化，如果没有使用归一化操作，收敛速度可能会比较慢。（5）如果数据量比较小，可以试着加载 PaddleClas 中提供的基于 ImageNet-1k 数据集的预训练模型，这可以大大提升训练收敛速度。（6）数据集存在长尾问题，可以参考[数据长尾问题解决方案](#long_tail)。

>>
* Q: 训练图像分类任务时，该怎么选择合适的优化器？
* A: 优化器的目的是为了让损失函数尽可能的小，从而找到合适的参数来完成某项任务。目前业界主要用到的优化器有 SGD、RMSProp、Adam、AdaDelt 等，其中由于带 momentum 的 SGD 优化器广泛应用于学术界和工业界，所以我们发布的模型也大都使用该优化器来实现损失函数的梯度下降。带 momentum 的 SGD 优化器有两个劣势，其一是收敛速度慢，其二是初始学习率的设置需要依靠大量的经验，然而如果初始学习率设置得当并且迭代轮数充足，该优化器也会在众多的优化器中脱颖而出，使得其在验证集上获得更高的准确率。一些自适应学习率的优化器如 Adam、RMSProp 等，收敛速度往往比较快，但是最终的收敛精度会稍差一些。如果追求更快的收敛速度，我们推荐使用这些自适应学习率的优化器，如果追求更高的收敛精度，我们推荐使用带 momentum 的 SGD 优化器。

>>
* Q: 当前主流的学习率下降策略有哪些？一般需要怎么选择呢？
* A: 学习率是通过损失函数的梯度调整网络权重的超参数的速度。学习率越低，损失函数的变化速度就越慢。虽然使用低学习率可以确保不会错过任何局部极小值，但也意味着将花费更长的时间来进行收敛，特别是在被困在高原区域的情况下。在整个训练过程中，我们不能使用同样的学习率来更新权重，否则无法到达最优点，所以需要在训练过程中调整学习率的大小。在训练初始阶段，由于权重处于随机初始化的状态，损失函数相对容易进行梯度下降，所以可以设置一个较大的学习率。在训练后期，由于权重参数已经接近最优值，较大的学习率无法进一步寻找最优值，所以需要设置一个较小的学习率。在训练整个过程中，很多研究者使用的学习率下降方式是 piecewise_decay，即阶梯式下降学习率，如在 ResNet50 标准的训练中，我们设置的初始学习率是 0.1，每 30 epoch 学习率下降到原来的 1/10，一共迭代 120 epoch。除了 piecewise_decay，很多研究者也提出了学习率的其他下降方式，如 polynomial_decay（多项式下降）、exponential_decay（指数下降）, cosine_decay（余弦下降）等，其中 cosine_decay 无需调整超参数，鲁棒性也比较高，所以成为现在提高模型精度首选的学习率下降方式。Cosine_decay 和 piecewise_decay 的学习率变化曲线如下图所示，容易观察到，在整个训练过程中，cosine_decay 都保持着较大的学习率，所以其收敛较为缓慢，但是最终的收敛效果较 peicewise_decay 更好一些。
![](../../images/models/lr_decay.jpeg)
>>
* Q: Warmup 学习率策略是什么？一般用在什么样的场景中？
* A: Warmup 策略顾名思义就是让学习率先预热一下，在训练初期我们不直接使用最大的学习率，而是用一个逐渐增大的学习率去训练网络，当学习率增大到最高点时，再使用学习率下降策略中提到的学习率下降方式衰减学习率的值。如果使用较大的 batch_size 训练神经网络时，我们建议您使用 warmup 策略。实验表明，在 batch_size 较大时，warmup 可以稳定提升模型的精度。在训练 MobileNetV3 等 batch_size 较大的实验中，我们默认将 warmup 中的 epoch 设置为 5，即先用 5 epoch 将学习率从 0 增加到最大值，再去做相应的学习率衰减。

>>
* Q: 什么是 `batch size`？在模型训练中，怎么选择合适的 `batch size`？
* A: `batch size` 是训练神经网络中的一个重要的超参数，该值决定了一次将多少数据送入神经网络参与训练。论文 [Accurate, Large Minibatch SGD: Training ImageNet in 1 Hour](https://arxiv.org/abs/1706.02677)，当 `batch size` 的值与学习率的值呈线性关系时，收敛精度几乎不受影响。在训练 ImageNet 数据时，大部分的神经网络选择的初始学习率为 0.1，`batch size` 是 256，所以根据实际的模型大小和显存情况，可以将学习率设置为 0.1*k, batch_size 设置为 256*k。在实际任务中，也可以将该设置作为初始参数，进一步调节学习率参数并获得更优的性能。
>>
* Q: weight_decay 是什么？怎么选择合适的 weight_decay 呢？
* A: 过拟合是机器学习中常见的一个名词，简单理解即为模型在训练数据上表现很好，但在测试数据上表现较差，在卷积神经网络中，同样存在过拟合的问题，为了避免过拟合，很多正则方式被提出，其中，weight_decay 是其中一个广泛使用的避免过拟合的方式。在使用 SGD 优化器时，weight_decay 等价于在最终的损失函数后添加 L2 正则化，L2 正则化使得网络的权重倾向于选择更小的值，最终整个网络中的参数值更趋向于 0，模型的泛化性能相应提高。在各大深度学习框架的实现中，该值表达的含义是 L2 正则前的系数，在 paddle 框架中，该值的名称是 l2_decay，所以以下都称其为 l2_decay。该系数越大，表示加入的正则越强，模型越趋于欠拟合状态。在训练 ImageNet 的任务中，大多数的网络将该参数值设置为 1e-4，在一些小的网络如 MobileNet 系列网络中，为了避免网络欠拟合，该值设置为 1e-5~4e-5 之间。当然，该值的设置也和具体的数据集有关系，当任务的数据集较大时，网络本身趋向于欠拟合状态，可以将该值适当减小，当任务的数据集较小时，网络本身趋向于过拟合状态，可以将该值适当增大。下表展示了 MobileNetV1_x0_25 在 ImageNet-1k 上使用不同 l2_decay 的精度情况。由于 MobileNetV1_x0_25 是一个比较小的网络，所以 l2_decay 过大会使网络趋向于欠拟合状态，所以在该网络中，相对 1e-4，3e-5 是更好的选择。

| 模型                | L2_decay | Train acc1/acc5 | Test acc1/acc5 |
|:--:|:--:|:--:|:--:|
| MobileNetV1_x0_25 | 1e-4     | 43.79%/67.61%   | 50.41%/74.70%  |
| MobileNetV1_x0_25 | 3e-5     | 47.38%/70.83%   | 51.45%/75.45%  |


>>
* Q: 标签平滑(label_smoothing)指的是什么？有什么效果呢？一般适用于什么样的场景中？
* A: Label_smoothing 是深度学习中的一种正则化方法，其全称是 Label Smoothing Regularization(LSR)，即标签平滑正则化。在传统的分类任务计算损失函数时，是将真实的 one hot 标签与神经网络的输出做相应的交叉熵计算，而 label_smoothing 是将真实的 one hot 标签做一个标签平滑的处理，使得网络学习的标签不再是一个 hard label，而是一个有概率值的 soft label，其中在类别对应的位置的概率最大，其他位置概率是一个非常小的数。具体的计算方式参见论文[2]。在 label_smoothing 里，有一个 epsilon 的参数值，该值描述了将标签软化的程度，该值越大，经过 label smoothing 后的标签向量的标签概率值越小，标签越平滑，反之，标签越趋向于 hard label，在训练 ImageNet-1k 的实验里通常将该值设置为 0.1。
在训练 ImageNet-1k 的实验中，我们发现，ResNet50 大小级别及其以上的模型在使用 label_smooting 后，精度有稳定的提升。下表展示了 ResNet50_vd 在使用 label_smoothing 前后的精度指标。同时，由于 label_smoohing 相当于一种正则方式，在相对较小的模型上，精度提升不明显甚至会有所下降，下表展示了 ResNet18 在 ImageNet-1k 上使用 label_smoothing 前后的精度指标。可以明显看到，在使用 label_smoothing 后，精度有所下降。

| 模型   | Use_label_smoothing | Test acc1 |
|:--:|:--:|:--:|
| ResNet50_vd | 0    | 77.9%  |
| ResNet50_vd | 1    | 78.4%  |
| ResNet18    | 0    | 71.0%  |
| ResNet18    | 1    | 70.8%  |


>>
* Q: 在训练的时候怎么通过训练集和验证集的准确率或者 loss 确定进一步的调优策略呢？
* A: 在训练网络的过程中，通常会打印每一个 epoch 的训练集准确率和验证集准确率，二者刻画了该模型在两个数据集上的表现。通常来说，训练集的准确率比验证集准确率微高或者二者相当是比较不错的状态。如果发现训练集的准确率比验证集高很多，说明在这个任务上已经过拟合，需要在训练过程中加入更多的正则，如增大 l2_decay 的值，加入更多的数据增广策略，加入 label_smoothing 策略等；如果发现训练集的准确率比验证集低一些，说明在这个任务上可能欠拟合，需要在训练过程中减弱正则效果，如减小 l2_decay 的值，减少数据增广方式，增大图片 crop 区域面积，减弱图片拉伸变换，去除 label_smoothing 等。

>>
* Q: 怎么使用已有的预训练模型提升自己的数据集的精度呢？
* A: 在现阶段计算机视觉领域中，加载预训练模型来训练自己的任务已成为普遍的做法，相比从随机初始化开始训练，加载预训练模型往往可以提升特定任务的精度。一般来说，业界广泛使用的预训练模型是通过训练 128 万张图片 1000 类的 ImageNet-1k 数据集得到的，该预训练模型的 fc 层权重是是一个 k\*1000 的矩阵，其中 k 是 fc 层以前的神经元数，在加载预训练权重时，无需加载 fc 层的权重。在学习率方面，如果您的任务训练的数据集特别小（如小于 1 千张），我们建议你使用较小的初始学习率，如 0.001（batch_size:256,下同），以免较大的学习率破坏预训练权重。如果您的训练数据集规模相对较大（大于 10 万），我们建议你尝试更大的初始学习率，如 0.01 或者更大。

<a name="1.3"></a>
### 1.3 数据相关

>>
* Q: 图像分类的数据预处理过程一般包括哪些步骤？
* A: 以在 ImageNet-1k 数据集上训练 ResNet50 为例，一张图片被输入进网络，主要有图像解码、随机裁剪、随机水平翻转、标准化、数据重排，组 batch 并送进网络这几个步骤。图像解码指的是将图片文件读入到内存中，随机裁剪指的是将读入的图像随机拉伸并裁剪到长宽均为 224 的图像，随机水平翻转指的是对裁剪后的图片以 0.5 的概率进行水平翻转，标准化指的是将图片每个通道的数据通过去均值实现中心化的处理，使得数据尽可能符合 `N(0,1)` 的正态分布，数据重排指的是将数据由 `[224,224,3]` 的格式变为 `[3,224,224]` 的格式，组 batch 指的是将多幅图像组成一个批数据，送进网络进行训练。

>>
* Q: 随机裁剪是怎么影响小模型训练的性能的？
* A: 在 ImageNet-1k 数据的标准预处理中，随机裁剪函数中定义了 scale 和 ratio 两个值，两个值分别确定了图片 crop 的大小和图片的拉伸程度，其中 scale 的默认取值范围是 0.08-1(lower_scale-upper_scale),ratio 的默认取值范围是 3/4-4/3(lower_ratio-upper_ratio)。在非常小的网络训练中，此类数据增强会使得网络欠拟合，导致精度有所下降。为了提升网络的精度，可以使其数据增强变的更弱，即增大图片的 crop 区域或者减弱图片的拉伸变换程度。我们可以分别通过增大 lower_scale 的值或缩小 lower_ratio 与 upper_scale 的差距来实现更弱的图片变换。下表列出了使用不同 lower_scale 训练 MobileNetV2_x0_25 的精度，可以看到，增大图片的 crop 区域面积后训练精度和验证精度均有提升。

| 模型                | Scale 取值范围 | Train_acc1/acc5 | Test_acc1/acc5 |
|:--:|:--:|:--:|:--:|
| MobileNetV2_x0_25 | [0.08,1]  | 50.36%/72.98%   | 52.35%/75.65%  |
| MobileNetV2_x0_25 | [0.2,1]   | 54.39%/77.08%   | 53.18%/76.14%  |


>>
* Q: 数据量不足的情况下，目前有哪些常见的数据增广方法来增加训练样本的丰富度呢？
* A: PaddleClas 中将目前比较常见的数据增广方法分为了三大类，分别是图像变换类、图像裁剪类和图像混叠类，图像变换类主要包括 AutoAugment 和 RandAugment，图像裁剪类主要包括 CutOut、RandErasing、HideAndSeek 和 GridMask，图像混叠类主要包括 Mixup 和 Cutmix，更详细的关于数据增广的介绍可以参考：[数据增广章节](../algorithm_introduction/DataAugmentation.md)。
>>
* Q: 对于遮挡情况比较常见的图像分类场景，该使用什么数据增广方法去提升模型的精度呢？
* A: 在训练的过程中可以尝试对训练集使用 CutOut、RandErasing、HideAndSeek 和 GridMask 等裁剪类数据增广方法，让模型也能够不止学习到显著区域，也能关注到非显著性区域，从而在遮挡的情况下，也能较好地完成识别任务。

>>
* Q: 对于色彩变换情况比较复杂的情况下，应该使用哪些数据增广方法提升模型精度呢？
* A: 可以考虑使用 AutoAugment 或者 RandAugment 的数据增广策略，这两种策略中都包括了锐化、直方图均衡化等丰富的颜色变换，可以让模型在训练的过程中对这些变换更加鲁棒。
>>
* Q: Mixup 和 Cutmix 的工作原理是什么？为什么它们也是非常有效的数据增广方法？
* A: Mixup 通过线性叠加两张图片生成新的图片，对应 label 也进行线性叠加用以训练，Cutmix 则是从一幅图中随机裁剪出一个 感兴趣区域(ROI)，然后覆盖当前图像中对应的区域，label 也按照图像面积比例进行线性叠加。它们其实也是生成了和训练集不同的样本和 label 并让网络去学习，从而扩充了样本的丰富度。
>>
* Q: 对于精度要求不是那么高的图像分类任务，大概需要准备多大的训练数据集呢？
* A: 训练数据的数量和需要解决问题的复杂度有关系。难度越大，精度要求越高，则数据集需求越大，而且一般情况实际中的训练数据越多效果越好。当然，一般情况下，在加载预训练模型的情况下，每个类别包括 10-20 张图像即可保证基本的分类效果；不加载预训练模型的情况下，每个类别需要至少包含 100-200 张图像以保证基本的分类效果。

<a name="long_tail"></a>
>>
* Q: 对于长尾分布的数据集，目前有哪些比较常用的方法？
* A:（1）可以对数据量比较少的类别进行重采样，增加其出现的概率；（2）可以修改 loss，增加图像较少对应的类别的图片的 loss 权重；（3）可以借鉴迁移学习的方法，从常见类别中学习通用知识，然后迁移到少样本的类别中。

<a name="1.4"></a>
### 1.4 模型推理与预测相关

>>
* Q: 有时候图像中只有小部分区域是所关注的前景物体，直接拿原图来进行分类的话，识别效果很差，这种情况要怎么做呢？
* A: 可以在分类之前先加一个主体检测的模型，将前景物体检测出来之后再进行分类，可以大大提升最终的识别效果。如果不考虑时间成本，也可以使用 multi-crop 的方式对所有的预测做融合来决定最终的类别。
>>
* Q: 目前推荐的，模型预测方式有哪些？
* A: 在模型训练完成之后，推荐使用导出的固化模型（inference model），基于 Paddle 预测引擎进行预测，目前支持 python inference 与 cpp inference。如果希望基于服务化部署预测模型，那么推荐使用 PaddleServing 的部署方式。
>>
* Q: 模型训练完成之后，有哪些比较合适的预测方法进一步提升模型精度呢？
* A:（1）可以使用更大的预测尺度，比如说训练的时候使用的是 224，那么预测的时候可以考虑使用 288 或者 320，这会直接带来 0.5% 左右的精度提升。（2）可以使用测试时增广的策略（Test Time Augmentation, TTA)，将测试集通过旋转、翻转、颜色变换等策略，创建多个副本，并分别预测，最后将所有的预测结果进行融合，这可以大大提升预测结果的精度和鲁棒性。（3）当然，也可以使用多模型融合的策略，将多个模型针对相同图片的预测结果进行融合。
>>
* Q: 多模型融合的时候，该怎么选择合适的模型进行融合呢？
* A: 在不考虑预测速度的情况下，建议选择精度尽量高的模型；同时建议选择不同结构或者系列的模型进行融合，比如在精度相似的情况下，ResNet50_vd 与 Xception65 的模型融合结果往往比 ResNet50_vd 与 ResNet101_vd 的模型融合结果要好一些。

>>
* Q: 使用固定的模型进行预测时有哪些比较常用的加速方法？
* A:（1）使用性能更优的 GPU 进行预测；（2）增大预测的 batch size；（3）使用 TenorRT 以及 FP16 半精度浮点数等方法进行预测。


<a name="2"></a>
## 2. PaddleClas 使用问题

>>
* Q: 评估和预测时，已经指定了预训练模型所在文件夹的地址，但是仍然无法导入参数，这么为什么呢？
* A: 加载预训练模型时，需要指定预训练模型的前缀，例如预训练模型参数所在的文件夹为 `output/ResNet50_vd/19`，预训练模型参数的名称为 `output/ResNet50_vd/19/ppcls.pdparams`，则 `pretrained_model` 参数需要指定为 `output/ResNet50_vd/19/ppcls`，PaddleClas 会自动补齐`.pdparams` 的后缀。


>>
* Q: 在评测 `EfficientNetB0_small` 模型时，为什么最终的精度始终比官网的低 0.3% 左右？
* A: `EfficientNet` 系列的网络在进行 resize 的时候，是使用 `cubic 插值方式`(resize 参数的 interpolation 值设置为 2)，而其他模型默认情况下为 None，因此在训练和评估的时候需要显式地指定 resize 的 interpolation 值。具体地，可以参考以下配置中预处理过程中 ResizeImage 的参数。
```
  Eval:
    dataset:
      name: ImageNetDataset
      image_root: ./dataset/ILSVRC2012/
      cls_label_path: ./dataset/ILSVRC2012/val_list.txt
      transform_ops:
        - DecodeImage:
            to_rgb: True
            channel_first: False
        - ResizeImage:
            resize_short: 256
            interpolation: 2
        - CropImage:
            size: 224
        - NormalizeImage:
            scale: 1.0/255.0
            mean: [0.485, 0.456, 0.406]
            std: [0.229, 0.224, 0.225]
            order: ''
```

>>
* Q: python2 下，使用 visualdl 的时候，报出以下错误，`TypeError: __init__() missing 1 required positional argument: 'sync_cycle'`，这是为什么呢？
* A: 目前 visualdl 仅支持在 python3 下运行，visualdl 需要是 2.0 以上的版本，如果 visualdl 版本不对的话，可以通过以下方式进行安装：`pip3 install visualdl -i https://mirror.baidu.com/pypi/simple`

>>
* Q: 自己在测 ResNet50_vd 预测单张图片速度的时候发现比官网提供的速度 benchmark 慢了很多，而且 CPU 速度比 GPU 速度快很多，这个是为什么呢？
* A: 模型预测需要初始化，初始化的过程比较耗时，因此在统计预测速度的时候，需要批量跑一批图片，去除前若干张图片的预测耗时，再统计下平均的时间。GPU 比 CPU 速度测试单张图片速度慢是因为 GPU 的初始化并 CPU 要慢很多。

>>
* Q: 灰度图可以用于模型训练吗？
* A: 灰度图也可以用于模型训练，不过需要修改模型的输入 shape 为 `[1, 224, 224]`，此外数据增广部分也需要注意适配一下。不过为了更好地使用 PaddleClas 代码的话，即使是灰度图，也建议调整为 3 通道的图片进行训练（RGB 通道的像素值相等）。

>>
* Q: 怎么在 windows 上或者 cpu 上面模型训练呢？
* A: 可以参考[开始使用教程](../models_training/classification.md)，详细介绍了在 Linux、Windows、CPU 等环境中进行模型训练、评估与预测的教程。
>>
* Q: 怎样在模型训练的时候使用 label smoothing 呢？
* A: 可以在配置文件中的 `Loss` 字段下进行设置，如下所示，`epsilon=0.1` 表示设置该值为 0.1，若不设置 `epsilon` 字段，则不使用 `label smoothing`。
```yaml
Loss:
  Train:
    - CELoss:
        weight: 1.0
        epsilon: 0.1
```
>>
* Q: PaddleClas 提供的 10W 类图像分类预训练模型能否用于模型推断呢？
* A: 该 10W 类图像分类预训练模型没有提供 fc 全连接层的参数，无法用于模型推断，目前可以用于模型微调。
>>
* Q: 在使用 `deploy/python/predict_cls.py` 进行模型预测的时候，报了这个问题: `Error: Pass tensorrt_subgraph_pass has not been registered`，这是为什么呢？
* A: 如果希望使用 TensorRT 进行模型预测推理的话，需要安装或是自己编译带 TensorRT 的 PaddlePaddle，Linux、Windows、macOS 系统的用户下载安装可以参考参考[下载预测库](https://paddleinference.paddlepaddle.org.cn/user_guides/download_lib.html)，如果没有符合您所需要的版本，则需要本地编译安装，编译方法可以参考[源码编译](https://paddleinference.paddlepaddle.org.cn/user_guides/source_compile.html)。
>>
* Q: 怎样在训练的时候使用自动混合精度(Automatic Mixed Precision, AMP)训练呢？
* A: 可以参考 [ResNet50_amp_O1.yaml](../../../ppcls/configs/ImageNet/ResNet/ResNet50_amp_O1.yaml) 这个配置文件；具体地，如果希望自己的配置文件在模型训练的时候也支持自动混合精度，可以在配置文件中添加下面的配置信息。

```yaml
# mixed precision training
AMP:
  scale_loss: 128.0
  use_dynamic_loss_scaling: True
  # O1: mixed fp16
  level: O1
```