## 端到端ASR论文小结

### 语音识别发展路径

https://developer.aliyun.com/article/704173

1、纯传统算法 GMM-HMM 

2、声学模型被深度学习模型替代：DNN-HMM （不完全端到端）

3、声学模型引入循环网络：RNN-HMM（不完全端到端）

4、纯神经网络算法，主流模型变多：RNN-CTC、CLDNN、Deep CNN、FSMN、DFCNN、Deep Speech（完全端到端）

*NN-HMM方式的hybrid声学建模被证明比目前学界前沿的e2e建模在低资源场景更加有效

### 声学模型

声学模型这里指NN-HMM中的NN部分，也可以理解为端到端ASR中encoder部分。

在端到端ASR的encoder-decoder结构中，encoder作用是对连续的信号特征进行建模，从原始语音特征（Fbank/MFCC/TF...）开始，逐层抽象特征，最后学习到连续信号的高维特征，送入decoder解码，encoder可对标NN-HMM系统中的声学模型。这里我们希望找到一个结构，能够很好地学习到信号特征，例如信号本身特征，长时特征，局部特征，时序特征等等...

#### 1) RNN

说明：时序建模曾经最主流的模型，Deep Speech就是用的rnn

#### 2) CNN

说明：Deep Speech 2由RNN进化到RNN+CNN

#### 3) TDNN

论文：A time delay neural network architecture for efficient modeling of long temporal contexts  (TDNN)

			Semi-Orthogonal Low-Rank Matrix Factorization for Deep Neural Networks  (TDNN-F)

相关：https://zhuanlan.zhihu.com/p/64148454 TDNN-F和DFSMN的关系

			https://blog.csdn.net/richard2357/article/details/16896837 TDNN
	
			https://www.jianshu.com/p/ddef79012db5   TDNN-F

说明：TDNN-F对TDNN的权重矩阵M进行半正交低秩矩阵分解，降低中间层维数

#### 4) FSMN

论文：Feedforward Sequential Memory Networks: A New Structure to Learn Long-term Dependency	(FSMN)

			Deep-FSMN for Large Vocabulary Continuous Speech Recognition	(DFSMN)
	
			A novel pyramidal-FSMN architecture with lattice-free MMI for speech recognition (pyramidal-FSMN)
	
			DFSMN-SAN with Persistent Memory Model for Automatic Speech Recognition	(DFSMN-SAN)

相关：https://www.cnblogs.com/machine-lyc/p/10572936.html  FSMN快速解读，有一个构造方阵的图示

			https://www.cnblogs.com/machine-lyc/p/10573743.html DFSMN快速解读
	
			https://developer.aliyun.com/article/601146  阿里巴巴开源DFSMN模型	
	
			https://developer.aliyun.com/article/600635  张仕良博士自己写的介绍DFSMN

说明：FSMN是由FNN加了一个记忆模块得来，这个Memory block可以看作是高阶FIR滤波器，对信号的长时相关性进行建模，相比于RNN训练上会更加简单和稳定；

			DFSMN在FSMN基础上增加了skip connection；
	
			pyramidal-FSMN，底层的Memory Block较小，越高层的Memory Block依次变大，只在记忆模块的维度发生变化时才进行skip connection操作；
	
			DFSMN-SAN，增加SAN模块

想法：阿里主推模型，从原理上看可以做到流式识别

#### 5) transformer （当前使用）

论文：Attention is all you need 2017

==Transformer-based Acoustic Modeling for Hybrid Speech Recognition  2019 (transformer for hybrid ASR)==

说明：当前的主流结构，多层自注意，擅长提取长序列依赖，可并行计算，容易忽略位置信息

#### 6) conformer

==论文：Conformer: Convolution-augmented Transformer for Speech Recognition 2020==

相关：https://zhuanlan.zhihu.com/p/319881884

说明：用卷积去加强transformer的encoder部分，加强局部视野，提高在语音识别领域的效果。

			核心是在每个SAN层后面添加Convolution Module，并且前后加上Feedforward module，类似马卡龙结构，实验结果表明SML三个尺寸的conformer都优于transformer。
	
			消融实验表明卷积块在效果上最为重要，Macaron-style的三明治FFN比只有一个FFN要好。Swish激活函数的使用使得模型收敛更快。

想法：据说conformer模型基本成为继transformer模型后的标配，有待实验验证。



### E2E整体结构

实现端到端ASR的几种网络结构，可以拆分成多个模块去解读，每个模块有特殊的作用，每个模块可以用不同算法来实现。

#### 1) encoder+ctc

论文：==Deep Speech: Scaling up end-to-end speech recognition 2014==

==Deep Speech 2: End-to-End Speech Recognition in English and Mandarin 2015==  ----->  encoder(CNN+RNN+FC) + CTC  

相关：https://zhuanlan.zhihu.com/p/36488476  知乎语音识别CTC详解

			https://www.jianshu.com/p/68eb6c377bda  CTC-based AM for ASR总结

说明：ctc的训练替代了人工标签对齐，实现端到端训练；解码相当于替代了HMM，缺点是解码有独立性，捕捉不到上下文信息，一般要引入语言模型来辅助修正。

CTC介绍博客 ： https://xiaodu.io/ctc-explained/

#### 2) encoder+rnnt

论文：Exploring Architectures, Data and Units For Streaming End-to-End Speech Recognition with RNN-Transducer

#### 3) encoder+attn+decoder

论文：Listen, Attend and Spell 2015  

#### 4) encoder+cif+decoder

论文：CIF: Continuous Integrate-and-Fire for End-to-End Speech Recognition

#### 5) hybird ctc + attn（当前使用）

论文：==Hybrid CTC/Attention Architecture for End-to-End Speech Recognition==

ESPNet https://github.com/espnet/espnet

#### 6) endocer + ctc + decoder

论文： ==Mask CTC==

非自回归的实现ASR，效率比较高，利用decoder来进行ctc输出的纠错。

论文： ==JOINT CTC-ATTENTION BASED END-TO-END SPEECH RECOGNITION USING MULTI-TASK LEARNING==

multi-task任务，利用CTC loss，增强e2e attention进行asr识别的对齐能力，同时增加其收敛速度，提升效果。



#### 小结

Deep Speech：结构encoder+ctc，骨干RNN

LAS：结构encoder+attn+decoder，骨干RNN

RNNT：结构encoder+rnnt，骨干RNN



### 训练目标

#### 1) ctc loss

论文：Connectionist Temporal Classification: Labelling Unsegmented Sequence Data with Recurrent Neural Networks 2006

#### 2) CE loss

#### 3) LST loss

论文：Learn Spelling from Teachers: Transferring Knowledge from Language Models to Sequence-to-Sequence Speech Recognition 2019

说明：知识蒸馏，需要预训练一个RNN-LM，AM的训练阶段LM同时生成soft label + LST loss去指导AM训练，预测可不用LM，也可做shallow fusion

框架：encoder+xxx+decoder

想法：怎么保证RNNLM预测的准确性？可以先用kenlm打分或预训练BERT来做指导？

			语言模型和声学模型对特征抓取的相似度不同，声学模型提取发声特征，语言模型提取文本频次，一个词的竞争词是不一样的，怎么融合？

#### 4) SD loss

论文：Self-Distillation for Improving CTC-Transformer-based ASR Systems

说明：自蒸馏，在CTC-Transformer的 joint 训练中添加多任务： ① Transformer ② CTC ③ 辅助任务（Self-Distillation）

			用attention weights和Transformer output来制作伪目标，伪目标携带了语言和时间信息

#### 5) MBR training

论文：Minimum Bayes Risk Training of RNN-Transducer for End-to-End Speech Recognition

说明：考虑Nbest路径和相应概率

Youtube咖喱味解读：https://www.youtube.com/watch?v=DUVC_4LBHf4

## 6）MWERloss

### 预训练

MPC training