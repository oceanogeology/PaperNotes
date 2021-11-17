# MoCo : Momentum Contrast for Unsupervised Visual Representation Learning

>在进行对比学习的时候，查询字典可以越大越好，可是在增大查询字典的同时，如何保证字典内的key的连续性就比较关键，比如query encoder，key encoder 同时反向传播，由于key encoder更新较快，就只能利用mini-batch的字典，而另一种方法将key全部存起来，每次sample一定的量进行学习，虽然字典的量上来了，但是由于每次只能更新sample出来的key，依然不具备连续性，基于此，本文提出的momentum更新方式，由于momentum很大基本上是0.999，这就导致key encoder其实更新的很慢，这个时候维护一个队列比如65536，bs=128的时候，512个batch就完成了一次队列的更新，这就保证了队列里面的数据不会太older，具备一定的连续性。还有一个小点是，contrastive loss的时候，其实是点积然后算cross_entropy(),做了一个k+1维的softmax

---

Kaiming He

https://github.com/facebookresearch/moco

---

## Abstract

- 对比学习，可以看做是字典查找，本文通过队列建立了一个动态的字典

## Introduction

- 相比较语言任务，视觉任务的更关注与字典的建立，因为原始信号是连续的高维空间，是没有结构的。
- contrastive loss
- 以前的方法，主要是建立动态的字典，字典中的key是从数据（image or patch）中采样的,然后用一个encoder来表示。 encoder用来完成字典查找的过程，query encoder应该匹配相同的key，不匹配其他，学习的过程就是最小化contrastive loss的过程。
- 所以，这个字典得大，并且在训练过程中是具有连续性的
- 本文提出方法如下图

![image-20211109162238989](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211109162238989.png)

## Related Work

- unsupervised self-supervised learning，主要有两个方面，一个是pertext tasks (不是完成一个具体的任务，而是学习以及更好的特征表达),另一个是 loss function

## Method

### contrastive learning as dictionary look-up

* 相似度的度量用点积，所以本文的对正负样本相似度的度量的loss函数如下：t是温度系数。简单点看，就是在k+1维的softmax-based classifier中，把q预测成k+的概率

  ![image-20211109164140087](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211109164140087.png)

- query representation 可以是$q=f_q(x^q)$ 输入的可以是image 也可以是patch 也可以是 patches， 两个encoder可以是相同的，可以是共享的，也可以是不同的。

- 

### Momentum contrast 

- 我们的假设是，好的表达可以从大的字典中学到，因为其有足够的负样本。
- 弄了个队列，来扩大字典的容量，新数据进队列，outdate数据出队列。
- 用队列可以扩大字典量，但是却限制了key encoder的更新,那怎么办？一个简单的解决方式是，直接copy query encoder to key encoder，但实验显示效果不好，可能是key encoder更新太快了，破坏了连续性。
- 本文是怎么干的呢？主要问题还是让key encoder更新的慢一点，所以就采用了动量的方式，每次梯度只更新query encoder，key encoder通过query的更新进行计算，公式如下，并且实验证明，动量越大，效果越好，e.g. m=0.999比m=0.9要好，说明更新的越慢越好。
- ![image-20211109174727800](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211109174727800.png)
- 该方法的好处，1.可以使用大量的数据，亿级别也不在话下，2.保证了更新的连续性。

### Pretext Task

- 实例辨别任务，a query and a key, 如果来自同一张图像，就是一个正例，如果是不同的就是一个反例。

### Technical details

- resnet as encoder
- last fc层的输出(128维)，经过L2-norm 作为key 和query的特征表示， t=0.07

### shuffling BN（小trick）

 - 传统的BN效果不是很好。
 - 因为是采用分布式进行的训练，所以数据会被分配到多块卡上，shuffling bn计算bn只在每块卡上单独计算。同时对于一批数据，对于key encoder，moco先把数据shuffle掉，然后在放到gpu过key encoder,但是query encoder并没有shuffle，这就导致key和query是跟不同的数据做的bn，解决了bn导致的数据内部通信减少了数据分辨信息的问题。

### 伪代码

![image-20211109194215614](C:\Users\wanglichun\Desktop\Typera\TyporaPapers\images\image-20211109194215614.png)

## Exp

- bs:256, 8 gpus
- init lr 0.03, 200 epochs
- linear classification protocol, 先unsepervise训练，然后固定参数，训练分类head,  ==initial learning rate:30 , weight decay is 0==这个设置有点猛啊。
- 在finetune的时候，bn是全同步的，而不是冻住的，并且在新加的层中也加入了BN