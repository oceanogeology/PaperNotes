# **ViLT: Vision-and-Language Transformer Without Convolution or Region Supervision**

> 第一篇整合多模态领域（patch embedding, transfomer, mask language model, image aug）的工作，提出了非常简单有效的基线模型。

****

**Wonjae Kim** * 1 † **Bokyung Son** * 1 **Ildoo Kim** 2

https://github.com/dandelin/vilt

****



## **Abstract**

- 视觉-文本预训练（VLP）目前的计算时长主要耗时在视觉特征提取，ViLT提出了卷积free的视觉编码方式。10倍提速于现有的VLP模型

## Introduction

![image-20220905134723180](..\images\image-20220905134723180.png)

- 现有的多种VLP可以分为以上4种。其中本文首次提出了不用深度视觉编码器来编码视觉特征；首次使用了whole word mask 和image augmentations的优势
- VSE对应的是图a，CLIP对应的是图b，VilBERT对应的是图c，ViLT对应的是图d。后续系类研究表明模态融合的关键应该放在MI上面。

## Method

![image-20220905153054819](..\images\image-20220905153054819.png)

![image-20220905153122115](..\images\image-20220905153122115.png)

- 图像embedding（B，197，768）和文本embedding（B，512，768）cat起来送入Transformer Encoder

- **Image Text Matching**： 就是图文对比损失

- **Masked Language Modeling** 和  **Whole Word Masking** 连用，文本以0.15的概率随机mask掉一整个单词，然后预测出这个mask掉的类别，其中MLM Head用了2层MLP，输出vocab size的logits，计算MLMLoss

- **Image Augmentation**：图片增强上使用了RandAugment（N=2，M=9）,因为之前的工作的多模态匹配为了避免mask掉图文中的关键信息，研究者没使用过强的Aug，这篇文章为aug打了强心针。

  