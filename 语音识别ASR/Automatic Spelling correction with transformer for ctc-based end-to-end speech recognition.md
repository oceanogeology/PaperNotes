# Automatic Spelling correction with transformer for ctc-based end-to-end speech recognition

> Author: Shiliang Zhang   Alibaba
>
> Year : 2019

## Abstract

* propose a transofrmer based spelling correction model to automatically correct errors
* ctc-based as input and gt as output to train a transformer with encoder-decoder architecture 
* CER 7.28%-> 4.89% 自己的数据集

## Introduction

* 在没有额外lm的情况下，一般基于attention的模型会比ctc模型更好，然而，attention的语言模型只在转录的音频文本对上进行训练。
* 因为中文中会存在很多同音字，所以额外的lm对于基于ctc的中文识别是很有必要的。
* 

## Related Works

* 《ASR context-sensitive error correction based on microsoft n-gram dataset》
* 《Comparison of decoding strategies for ctc acoustic models》
* 《A spelling correction model for end-to-end speech recognition》

## Our Approach



* 总体结构如下图，包含：Listenr,Decoder,Speller

![image-20210623193953984](C:\Users\wanglichun\Desktop\TyporaPapers\images\image-20210623193953984.png)



### Listener

#### DFSMN-CTC-sMBR

* DFSMN的论文《Deep-FSMN for large vocabulary continuous speech recognition》

* state-level minimum Bayes risk criterion

#### Acoustic modeling units

* 实验证明hybrid Character-Syllable modeling units, which mixed the high frequency Chinese characters and syllables, is the best choice for Mandarin speech recognition

### Decoder

* Greedy search
* WFST search
* N-best data expansion

###  Speller

* OpenNMT toolkit to train

## Experiments

