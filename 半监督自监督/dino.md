# Self-Supervised Vision Transformers with DINO

```
@inproceedings{caron2021emerging,
  title={Emerging Properties in Self-Supervised Vision Transformers},
  author={Caron, Mathilde and Touvron, Hugo and Misra, Ishan and J\'egou, Herv\'e  and Mairal, Julien and Bojanowski, Piotr and Joulin, Armand},
  booktitle={Proceedings of the International Conference on Computer Vision (ICCV)},
  year={2021}
}
```

github: https://github.com/facebookresearch/dino

### 核心算法：

<img src="..\images\2021122701.png" style="zoom:80%;" />

### 要点：

① 利用multi-crop的思想达到局部-全局注意力机制

 <img src="..\images\2021122702.png" style="zoom:100%;" />

②  对老师特征进行centering，避免模型坍塌

### 不懂：

① 为啥momentum teacher的出现会产生模型坍塌？

![](..\images\2021122706.png)

② KL=0，表示坍塌？ entropy h怎么能说明坍塌的不同形式？

![](..\images\2021122705.png)

