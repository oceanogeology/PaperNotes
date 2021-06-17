# SpecAugment : A Simple Data Augmentation Method for Automatic Speech Recogniton

## Abstract

* applied to feature inputs of a neural network,(ie FBank)
* ==warp the feature;  mask blocks of frequency channel ; masking blocks of time steps.==

## Augmentation Policy

* warping: 在时间维度上，当成图像，进行一定的扭曲
* Frequency masking: 
* Time masking:
* 

<img src="..\images\image-20210610100337514.png" alt="image-20210610100337514" style="zoom:50%;" />

* 三种方法中Time warping耗时最多，但是收益最小。

* ![image-20210610103040874](..\images\image-20210610103040874.png)

* Augment可以解决过拟合问题，使得训练效果更好。

  ![image-20210610102933458](..\images\image-20210610102933458.png)

* 

