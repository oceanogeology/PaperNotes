# Improved mask-ctc for non-autoregressive end-to-end asr

> author : Yosuke Higuchi
>
> Year : 2021

## Abstract

* Mask CTC 速度很快，但是其表现不如自回归系统。
* 增强encoder的能力，引入conformer
* 提出了新的训练和解码方法，通过引入辅助目标来预测部分目标序列的长度，可以实现删除或插入
* WER on WSJ: 15.5% -> 9.1%

## Introduction

* mask ctc的问题：1. mask-ctc受到ctc输出结果的严重限制，只有ctc输出的小错误才会被纠正。2. 不能实现插入和删除
* 引入conformer，引入新的训练和解码策略，解决插入删除操作。

## Non-autoregressive end to end asr

### CTC

### Mask-CTC

![image-20210617153529698](..\images\image-20210617153529698.png)

* inference, greedy decoding , low score are masked ,and then fed into MLM decoder

## Proposed improvemnets of mask-ctc

* enhance the encoder architecture by adopting the state-of-the-art encoder architecture, ==**Conformer**==
*  introduce new training and decoding methods for the MLM decoder to handle the insertion and deletion
  errors during inference.



### Dynamic length prediction(DLP)

* 训练mlm decoder实现自动长度的预测功能

#### Deletion-simulated task

* 主要预测被删除的字符

* Y = [Y1, Y2, Y3, Y4] => [ Y1, MASK, Y4]
* random sample <MASK>, and merge consecutive mask to one mask
* 预测每个mask的数量，如下公式所示，$Y^{del}_{obs}$代表的是观察到的字符，$D_{del}$代表的是预测的数量
* ![image-20210617194112994](..\images\image-20210617194112994.png)

#### Insertion-simulated task

* 主要预测多插入的字符，通过预测mask的数量为0，进而删除mask
* Y = [Y1, Y2, Y3, Y4] => [ Y1, Y2, Y3, MASK, Y4]
* random insert <MASK>
* $Y_{obs}^{ins}$ 代表的是插入mask后，观察到的字符串  $D_{ins}$代表的是预测的数量
* ![image-20210617194531176](..\images\image-20210617194531176.png)

---

* 如何实现数量预测呢？网络结构上，加入了一个linear layer 然后接了一个softmax进行预测，==这里作者设置的预测最大值是50==
* DLP的loss就是如下公式：
* ![image-20210617194935667](..\images\image-20210617194935667.png)
* 整个e2e的mask ctc的loss如下：
* ![image-20210617195045226](..\images\image-20210617195045226.png)

#### Inference

* 算法流程图
* ==需要两次前向计算==，第一次计算mask的length，第二次计算出预测的结果，进而实现插入和删除操作。**==这里还有个疑问，就是训练的时候如何同时进行数量统计和mask预测，貌似论文里没提==**

![image-20210617192237848](..\images\image-20210617192237848.png)



## Experiments

* WSJ数据集，TEDLIUM2数据集，Voxforge 数据集
* 80fb + 3 pitch 
* data augment : speed perturbation + specaugment

### Result

* 引入DLP后，可以得到更好的performance，比較B2和B3
* 引入conformer后，可以得到更好的performance,比較C1-C3
* 非自回归会比自回归快5.7倍
* 使用DLP之后，由于增加了计算量，所以耗时会增加，但是由于使用了DLP之后，可以减少迭代次数，所以整体时间并没有延迟，反而更快。

![image-20210621153822541](..\images\image-20210621153822541.png)

