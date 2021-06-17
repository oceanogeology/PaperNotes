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









