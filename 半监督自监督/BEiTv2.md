# BEIT V2: Masked Image Modeling with Vector-Quantized Visual Tokenizers

> 

------

机构：Microsoft Research

Github: https://github.com/microsoft/unilm

****

## Abstract

- 提出用semantic-level的视觉tokenizer作为重建target
- 引入向量定量化的知识蒸馏，离散一个连续的语义空间来压缩编码
- imagenet上达到87.3的fine-tuning

![image-20220818112625805](..\images\image-20220818112625805.png)

## Introduction

- 恢复高语义信息更能充分利用模型表达空间
- Vector-Quantized Knowledge Distillation (VQ-KD) algorithm to discretize a semantic space
- 将全局信息添加到所有局部patches

## Methods

![image-20220818114802033](..\images\image-20220818114802033.png)

-  **Image Representations**: 利用patchembedding将image转化为NxD的维度
- **Training Visual Tokenizer**: 
- 