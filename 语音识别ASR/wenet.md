# WeNet

> author : Zhouyuan Yao
>
> Year: 2021
>
> Github :  https://github.com/wenet-e2e/wenet

## Abstract

* wenet是一个开源的语音识别工具（We in WeChat, Net in EspNet）
* 利用一个新的two-pass 方法U2实现了在单个模型中联合流式与非流式的语音识别
* **CTC解码，利用transformer作为encoder，利用attention decoder进行重打分。**
* **为了实现流式与非流式的动态解码，使用chunk策略，focus on 随机长度的上下文**
* aishell-1 dataset，5.03%的CER

## Introduction

* AED 的缩写是 attention based encoder decoder，。。。

* end2end存在的问题,也是WeNet想主要解决的问题：

  > streaming problems, such as  LAS and Transformer
  >
  > unifying streaming and non-streaming modes,一个模型同时满足两个条件是比较难的
  >
  > **production problem**落地的问题最大

* wenet做了哪些努力

  > exported to JIT
  >
  > U2 framework to unified solution for streaming and non-streaming ASR
  >
  > portable runtime,可以部署各种环境
  >
  > light weight: no dependencies on Kaldi



## WeNet

### Model Architecture

* shared encoder + ctc decoder + attention decoder, attention decoder 对ctc decoder进行rescore

![image-20210707194535188](..\images\image-20210707194535188.png)

* training loss:

  

![image-20210707194645193](..\images\image-20210707194645193.png)

* use chunk technique to unify the non-streaming and streaming model

  > * input is split into several chunks by a fixed chunk size C, every chunk attends on itself and all the previous chunks ,当chunk被限制在比较小的时候，就是流式的了
  >* secondly, 训练的时候，chunk的尺寸是从1-max随机的去选择，这样模型可以学习各种尺寸的chunk
  > * 一般来说，大的chunk结果会更好，延时会更高
  > * **chunk这个东西怎么理解，我觉得就是在原有的基于非自回归的方法上，可以看到一点点未来的东西进行解码，进而比只能看到过去的东西要好一点点**

* Decoding

  > ctc prefix beam search
  >
  > attention rescoring


### 系统结构：

![image-20210707200806038](..\images\image-20210707200806038.png)

* Torchaudio 直接提取特征，不需要kaldi先保存好
* Joint CTC  / AED training
* DDP
* 支持多端部署

## 实验

* 数据集： AISHELL-1, 150小时training set
* 融合top-k的模型
* 下表，M1采用非流式训练， M2采用chunk训练，4,8,16代表chunk的尺寸。attention_rescoring基本可以涨点。ctc_greedy_search和ctc_prefix_beam_search结果基本相近，并且随着chunk size的减少，性能下降明显。

![image-20210707202405261](..\images\image-20210707202405261.png)

* 实时率 RTF ，chunk越小实时率越高，并且量化可以带来一定的提升，当量化的效果并没有明显的降低。

  ![image-20210708094924667](..\images\image-20210708094924667.png)

  ![image-20210708095029009](..\images\image-20210708095029009.png)

  

  * Latency，就放个图吧，反正挺快的

    ![image-20210708103120454](..\images\image-20210708103120454.png)


## 代码理解

1. 针对attention decoder rescore的代码阅读记录

   代码位置：https://github.com/wenet-e2e/wenet/blob/main/runtime/core/decoder/torch_asr_decoder.cc：AttentionRescoring( )

   * 拿到ctc beam seach 的 n-best的结果，并将其变成矩阵输入后面的attention decoder
   * 将上面nbest以及encoder_output作为输入，forward  decoder
   * 计算每一句的decoder score
   * 同时根据选择，计算reverser的decoder score
   * 最后根据权重进行加权融合

   

   

   

论文待看：

《Unified Streaming and Non-streaming Two-pass End-to-end Model for Speech Recognition》

