# MOCO v3: An Empirical Study of Training Self-Supervised Vision Transformers

> Siamese  network, ViT, 

## Related Work

- 对比学习被广泛应用于Siamese网络，最近一些列研究已经在Siamese网络中移除了negative samples，表明核心就是学习正样本对的不变特征。

## Method

- 同一张图片的2种crop方式做匹配，经过Siamese网络f_q和f_k，利用MOCO里面的InfoNCEloss。

##### 伪代码

![image-20220817141126667](..\images\2022081701.png)
