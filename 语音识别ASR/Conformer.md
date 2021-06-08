# Conformer: convolution-augmented Transformer for Speech Recognition

> anmol Gulati
>
> 2020
>
> Google

## Abstract

* Transformer models are good at capturing content-based global interactions, while CNNs exploit local features effectively
*  propose the convolution-augmented transformer for speech recognition, named Conformer
*  WER of 2.1%/4.3%  1.9%/3.9% with an external language model on test/testother.We also
  observe competitive performance of 2.7%/6.3%  10M

## Introduction

* Transformer architecture based on self-attention [6, 7] has enjoyed widespread adoption for modeling  sequences due to its ability to capture long distance interactions and the high training efficiency

* Conformer结构如下图所示

  ![image-20210608115500052](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608115500052.png)

  

## Conformer Encoder

* first convolution subsampling layer + conformer blocks
* 4个module: feed_forward, self-attention, convolution, feed_forward

### Multi-Headed Self-Attention Module

* multi-head attention + ==sinusoidal positional encoding==
* pre-norm 如下图，可帮助训练更深的网络。

![image-20210608142026030](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608142026030.png)

### Convolution Module

* 卷积结构如下：Inspired by 《Lite transformer with long-short range attention》，==为啥这么设计，本论文没说，引用的论文里不知道有没有说明==

  ![image-20210608142447775](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608142447775.png)

### Feed Forward Module

* pre-norm
* layer normal
* Swish activation and dropout

![image-20210608142834380](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608142834380.png)

### Conformer Block

* 整个结构的灵感来源于macaron-Net《Understanding and improving transformer from a multi-particle dynamic system point of view》，proposes replacing the original feed-forward layer in the Transformer block into two half-step feed-forward layers, one before the attention layer and one after
* 数学表达就是如下：

![image-20210608143303270](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608143303270.png)

## Experiments

* 80-dim FBank ;  25ms, stride 10ms  ;  SpecAugment ; Adam; warmup
* Lingvo toolkit

### Conformer Transducer

* 3 scale models,   use a single-LSTM-layer decoder in all our models, as follow

![image-20210608144131715](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608144131715.png)

* 3-layer LSTM language model
* ![image-20210608144907737](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608144907737.png)
* 



### Ablation Studies

#### Conformer Block vs Transformer Block

* 如下图可见，在一点点的减少conformer的组件之后，发现conv是影响比较大的，其次Swish以及ffn也有一定的影响。

![image-20210608145422885](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608145422885.png)

#### Combinations of Convolution and Transformer Modules

* replace depthwise conv to a lightweight conv, significant drop

* place conv before multi-head-attention, degrade 0.1

* conv 与 multi-head atten  并行， worse

  ![image-20210608150049510](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608150049510.png)

  #### Macaronn Feed Forward Modules

  ![image-20210608150241915](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608150241915.png)

  #### Number of Attention Heads

  * 增加head数量到16，在other上面表现会略有提升。

  ![image-20210608151407957](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608151407957.png)

#### convolution Kernel Sizes

* kernel size = 32 ，效果比较好

![image-20210608151716434](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210608151716434.png)

##  总结

本篇论文，主要的思想是将conv引入到transformer中，主要点有下面几个，1、conv在trans后面，2、引入了马卡龙夹心结构，3、ffn的权重采用1/2。证明都是比较有效果的。以及一些其他的trick。并且消融实验还是做得比较充分的，利用消融实验找到一些比较好的参数。