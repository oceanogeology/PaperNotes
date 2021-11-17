# Zero-shot  text-to-image  Generation

author: openai

---

## Abstract

- text-to-image 过去的努力在于针对特定的数据，找到更好的模型假设，这些假设可以包括：复杂的模型，辅助loss,或者物体的label以及分割的mask等
- based on transformer , autoregressively

## introduction

* in this work, we demonstrate that training a 12-billion param-eter autoregressive transformer on 250 million image-textpairs collected from the internet results in a flexible, highfidelity generative model of images controllable throughnatural language.

## Method

- Stage 1:  train a discrete variational autoencoder(dVAE) to compress 256x256 to 32x32 grid of image tokens,就是说将256x256的通过vae方法压缩到32x32，这样可以减少192倍的信息。（==具体而言，图像分词器首先通过Encoder 将H*W*3的图像转换为h*w*d特征向量，之后特征向量中每一个长度为d的向量都会通过“查表操作”转化为一个embedding的index，最终变为 h*w的量化向量。而Decoder 的作用是将量化后的特征重构为较为模糊的原始图像。整个Image Tokenizer的训练采用了VQ-VAE中的模式，这里不做过多介绍。==）
- 这个介绍讲的比较详细：https://spaces.ac.cn/archives/6760， 结合这个代码： https://nbviewer.org/github/zalandoresearch/pytorch-vq-vae/blob/master/vq-vae.ipynb
- stage 2: 将文字tokens与图像的tokens concat到一起，训练一个自回归的transformer，来联合文字和图像的分布。
- gumbel-softmax relaxation
- 