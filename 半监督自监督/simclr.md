# SimCLR

>contrastive learning，简单来说就是通过unlabel data,构建相似图像与非相似图像集，然后判断模型输出向量，相似图像比较接近，非相似图像比较远。本文的主要方法也比较简单，相比较与moco采用的query encoder 和key encoder,本文采用相同的encoder，通过大的batch（8192）来构建相似与非相似样本集，通过不同的augment来产生positive pair以及negative pair, 通过positive pair和nagative pair 的相似度来计算contrastive loss，得到一个比较好的特征表达的模型。

---

Ting Chen, Hinton

Github: https://github.com/google-research/simclr.

---

## Abstract

- data augment 在定义高效的预测任务中扮演着重要的角色，并且这个角色比在监督训练中还要大。
- 在特征表达以及contrastive loss中，建立一个可学习的非线性转换，对于特征表达的学习具有重大意义。
- 对embedding进行normalize以及适当的调整温度系数，利用contrastive celoss 特征表达的学习
- ==contrastive learning需要更大的batchsize以及更多的训练steps。==

![image-20211110105603900](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211110105603900.png)

![image-20211111154404267](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211111154404267.png)

## Introduction

- 学习高效的特征表达主要有两点，1个是generative（生成），另一个是discriminative（鉴别），生成的方法比如pixel的预测，会比较依赖计算资源。鉴别的方法比较依赖于pretext的设计。最近的 discriminative + contrastive learning的方法取得了不错的效果。

## Relation

- handcrafted pretext  tasks: 通过定义代理任务，来完成特征的学习，比如预测，相邻patch,拼图，旋转角度，color变换等，但这种方法太依赖与任务的设定。
- contrastive visual representation learning,这个是通过判断是否是positive pair 还是negative pair来进行特征的学习。

## Method

### contrastive learning framework

- 随机的augment来产生positive pairs
- 网络结构比较随意，这里采用的是resnet，average pooling layer的输出作为特征表达
- 在上面的输出上，加上了一个projection head(2层MLP，中间ReLU)，实验发现，这样做更有利于contrastive loss的计算（==kaiming的moco也是后面加了fc，不过是加1层，这个加了2层==）
- 对于batch_size=N，经过aug，我们会得到2N的数据，其中正样本1对，其余的2（N-1）的数据，均作为负样本。
- $sim(u,v) = u^Tv/||u||*||v||$, 这不就是数学上向量的内积的夹角嘛，在代码中，相当于点积后做了归一化操作，==就是cosine similarity，余弦相似度==， ==这里跟moco也是一样的，只不过moco的故意化操作L2 norm在网络中完成了==
- ![image-20211110112345449](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211110112345449.png)
- 最终的loss是计算的所有的positive pairs，both (i,j) and (j, i)
- 伪代码如下：
- ![image-20211111154921350](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211111154921350.png)

### Training with Large Batch Size

- simclr并没有使用memory bank,而是使用的大的batch size，将训练batch从256调整到8192,8192可以带来16382的负样本。
- 由于batch size 太大了，使用sgd训练可能会不稳定，所以本文使用的是==LARS optimizer==，==这个是什么优化器？没用过==
- 128个TPU cores

### Global BN

- 意思就是如果采用分布式计算的情况下，各个卡分别计算自己的数据，这会导致局部信息优化预测的精度，但是并不会带来特征表达的提升。==啥意思没整明白，是不是说自己训自己的，对自己有益，但是对全局无益==
- 所以，simclr是对全局进行的bn，或者像moco那种采用shuffle计算bn也行，就是麻烦了一点。

Default settng

- resnet50
- 2-layer MLP projection head -> 128-dim
- optimize: LARS, lr:4.8,weight decay:  1e-6

## Data Augmentation for Contrastive Repressentation Learning

- 实验发现，如果是两个组合的话，random crop + color distortion 效果比较好
- 并且实验发现，使用auto augment没有random crop + color distortion的效果好
- 并且对color distortion进行研究发现，随着强度的增强，模型效果会变好，但是sepervise model，会变坏。
- 总而言之，unserpervise learnging 比 sepervise learning 需要更强的augment

## Architectures

- Unsupervised contrastive learning benefits (more) from bigger models
- nonlinear的头还是比较有用的，提升比较大，==kaiming的moco就是没有采用nonlinear，而是采用的linear==。
- ![image-20211110174646352](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211110174646352.png)
- NT-Xent损失函数，下面的XT-Xent 的loss就是对上面的loss ，log提取后的结果。

![image-20211111142903339](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211111142903339.png)

- 作者对比了在loss中，L2 norm, 和温度的对比效果，实验发现，没有了norm之后 contrastive acc很高，但是特征表达不是特别好，比较低。同时合适的温度也有较为明显的影响。

  ![image-20211111144136313](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211111144136313.png)

- ==larger batch is better==, longer training is better,（作者的部分对比实验是训练了1000epoch）

## Comparison with Sota

- Linear evaluation:
- ![image-20211111144524754](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211111144524754.png)
- transfer learning:(可见基本上是有点效果的，相比supervised)

![image-20211111145019009](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211111145019009.png)

## 附录

- randomresizecrop() 通用设置，crop size 0.08-1, aspect 3/4 - 4/3,  horizontal flip

- color_distortion代码如下，

- ![image-20211111151242266](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211111151242266.png)

- 发现针对LARS optimizer , square root learning rate scaling 效果比单纯的scaling效果要好。

- 实验显示，bs=8192的时候 ， 似乎 饱和了，继续增大到32k，并没有显著改善效果了

- finetune 参数

  > bs:4096, lr:0.8, momentum:0.9,
  >
  > 1%数据 finetune 60 epochs, 10%数据再finetune 30 epoch

