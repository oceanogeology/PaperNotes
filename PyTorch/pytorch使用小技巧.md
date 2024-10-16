# Some Tricks of PyTorch

## changelog

* 2019年11月29日: 更新一些模型设计技巧和推理加速的内容, 补充了下apex的一个介绍链接, ~~另外删了tfrecord, pytorch能用么? 这个我记得是不能, 所以删掉了~~(表示删掉:<)
* 2019年11月30日: 补充MAC的含义, 补充ShuffleNetV2的论文链接
* 2019年12月02日: 之前说的pytorch不能用tfrecord, 今天看到<https://www.zhihu.com/question/358632497>下的一个回答, 涨姿势了
* 2019年12月23日: 补充几篇关于模型压缩量化的科普性文章
* 2020年2月7日: 从文章中摘录了一点注意事项, 补充在了[代码层面](#代码层面)小节
* 2020年4月30日:
    - 添加了一个github的文档备份
    - 补充了卷积层和BN层融合的介绍的链接
    - 另外这里说明下, 对于之前参考的很多朋友的文章和回答, 没有把链接和对应的内容提要关联在一起, 估计会导致一些朋友阅读时相关的内容时的提问, 无法问到原作者, 这里深感抱歉.
    - 调整部分内容, 将内容尽量与参考链接相对应
* 2020年5月18日: 补充一些关于PyTorch节省显存的技巧. 同时简单调整格式. 另外发现一个之前的错误: `non_blocking=False` 的建议应该是 `non_blocking=True` .
* 2021年01月06日：调整下关于读取图片数据的一些介绍.
* 2021年01月13日：补充了一条推理加速的策略. 我觉得我应该先更新github的文档，知乎答案的更新有点麻烦，也没法比较更改信息，就很费劲。

## PyTorch提速

> 原始文档:<https://www.yuque.com/lart/ugkv9f/ugysgn>
>
> 声明: 大部分内容来自知乎和其他博客的分享, 这里只作为一个收集罗列. 欢迎给出更多建议.

知乎回答(欢迎点赞哦):

* pytorch dataloader数据加载占用了大部分时间, 各位大佬都是怎么解决的? - 人民艺术家的回答 - 知乎 <https://www.zhihu.com/question/307282137/answer/907835663>
* 使用pytorch时, 训练集数据太多达到上千万张, Dataloader加载很慢怎么办? - 人民艺术家的回答 - 知乎 <https://www.zhihu.com/question/356829360/answer/907832358>

### 预处理提速

* 尽量减少每次读取数据时的预处理操作, 可以考虑把一些固定的操作, 例如 `resize` , 事先处理好保存下来, 训练的时候直接拿来用
* Linux上将预处理搬到GPU上加速:
    - `NVIDIA/DALI` :<https://github.com/NVIDIA/DALI>

### IO提速

* 推荐大家关注下mmcv，其对数据的读取提供了比较高效且全面的支持：
  - OpenMMLab：MMCV 核心组件分析(三): FileClient <https://zhuanlan.zhihu.com/p/339190576>

***使用更快的图片处理***

* `opencv` 一般要比 `PIL` 要快 （**但是要注意，`PIL`的惰性加载的策略使得其看上去`open`要比`opencv`的`imread`要快，但是实际上那并没有完全加载数据，可以对`open`返回的对象调用其`load()`方法，从而手动加载数据，这时的速度才是合理的**）
* 对于 `jpeg` 读取, 可以尝试 `jpeg4py`
* 存 `bmp` 图(降低解码时间)
* 关于不同图像处理库速度的讨论建议关注下这个：Python的各种imread函数在实现方式和读取速度上有何区别？ - 知乎 <https://www.zhihu.com/question/48762352>

***小图拼起来存放(降低读取次数)***

对于大规模的小文件读取, 建议转成单独的文件, 可以选择的格式可以考虑: `TFRecord（Tensorflow）` , `recordIO（recordIO）` , `hdf5` , `pth` , `n5` , `lmdb` 等等(<https://github.com/Lyken17/Efficient-PyTorch#data-loader>)

* `TFRecord` :<https://github.com/vahidk/tfrecord>
* 借助 `lmdb` 数据库格式:
    - <https://github.com/Fangyh09/Image2LMDB>
    - <https://blog.csdn.net/P_LarT/article/details/103208405>
    - <https://github.com/lartpang/PySODToolBox/blob/master/ForBigDataset/ImageFolder2LMDB.py>

***预读取数据***

* 预读取下一次迭代需要的数据

【参考】

* 如何给你PyTorch里的Dataloader打鸡血 - MKFMIKU的文章 - 知乎 <https://zhuanlan.zhihu.com/p/66145913>
* 给pytorch 读取数据加速 - 体hi的文章 - 知乎 <https://zhuanlan.zhihu.com/p/72956595>

***借助内存***

* 直接载到内存里面
* 把内存映射成磁盘

【参考】

* YoloV5的`--cache-images`的实现，实际上就是将图片读取后存到一个固定的容器对象中。
* 参见 <https://zhuanlan.zhihu.com/p/66145913> 的评论中 @雨宫夏一 的评论（额，似乎那条评论找不到了，不过大家可以搜一搜）

***借助固态***

* 把读取速度慢的机械硬盘换成 NVME 固态吧～

【参考】

* 如何给你PyTorch里的Dataloader打鸡血 - MKFMIKU的文章 - 知乎 <https://zhuanlan.zhihu.com/p/66145913>

### 训练策略

***低精度训练***

* 在训练中使用低精度( `FP16` 甚至 `INT8` 、二值网络、三值网络)表示取代原有精度( `FP32` )表示
  - 使用 Apex 的混合精度或者是PyTorch1.6开始提供的torch.cuda.amp模块来训练. 可以节约一定的显存并提速, 但是要小心一些不安全的操作如mean和sum:
    - `NVIDIA/Apex` :
        - <https://blog.csdn.net/c9Yv2cf9I06K2A9E/article/details/100135729>
        - <https://github.com/nvidia/apex>
        - Pytorch 安装 APEX 疑难杂症解决方案 - 陈瀚可的文章 - 知乎<https://zhuanlan.zhihu.com/p/80386137>
        - <http://kevinlt.top/2018/09/14/mixed_precision_training/>
    - `torch.cuda.amp`:
        - <PyTorch的文档：https://pytorch.org/docs/stable/notes/amp_examples.html>

### 代码层面

* `torch.backends.cudnn.benchmark = True`
* Do numpy-like operations on the GPU wherever you can
* Free up memory using `del`
* Avoid unnecessary transfer of data from the GPU
* Use pinned memory, and use `non_blocking=True` to parallelize data transfer and GPU number crunching
    - 文档：<https://pytorch.org/docs/stable/nn.html#torch.nn.Module.to>
    - 关于 `non_blocking=True` 的设定的一些介绍：Pytorch有什么节省显存的小技巧？ - 陈瀚可的回答 - 知乎 <https://www.zhihu.com/question/274635237/answer/756144739>
* 网络设计很重要, 外加不要初始化任何用不到的变量, 因为 PyTorch 的初始化和 `forward` 是分开的, 他不会因为你不去使用, 而不去初始化
* 合适的 `num_worker` : Pytorch 提速指南 - 云梦的文章 - 知乎 <https://zhuanlan.zhihu.com/p/39752167>(这里也包含了一些其他细节上的讨论)

### 对优化器的优化

* 将模型参数存放到一块连续的内存中，从而减少`optimizer.step()`的时间
    - <https://github.com/PhilJd/contiguous_pytorch_params>

### 模型设计

来自 ShuffleNetV2 的结论:(内存访问消耗时间, `memory access cost` 缩写为 `MAC` )

* 卷积层输入输出通道一致: 卷积层的输入和输出特征通道数相等时 MAC 最小, 此时模型速度最快
* 减少卷积分组: 过多的 group 操作会增大 MAC, 从而使模型速度变慢
* 减少模型分支: 模型中的分支数量越少, 模型速度越快
* 减少 `element-wise` 操作: `element-wise` 操作所带来的时间消耗远比在 FLOPs 上的体现的数值要多, 因此要尽可能减少 `element-wise` 操作( `depthwise convolution` 也具有低 FLOPs 、高 MAC 的特点)

其他:

* 降低复杂度: 例如模型裁剪和剪枝, 减少模型层数和参数规模
* 改模型结构: 例如模型蒸馏, 通过知识蒸馏方法来获取小模型

### 推理加速

***半精度与权重量化***

在推理中使用低精度( `FP16` 甚至 `INT8` 、二值网络、三值网络)表示取代原有精度( `FP32` )表示:

* `TensorRT` 是 NVIDIA 提出的神经网络推理(Inference)引擎, 支持训练后 8BIT 量化, 它使用基于交叉熵的模型量化算法, 通过最小化两个分布的差异程度来实现
* Pytorch1.3 开始已经支持量化功能, 基于 QNNPACK 实现, 支持训练后量化, 动态量化和量化感知训练等技术
* 另外 `Distiller` 是 Intel 基于 Pytorch 开源的模型优化工具, 自然也支持 Pytorch 中的量化技术
* 微软的 `NNI` 集成了多种量化感知的训练算法, 并支持 `PyTorch/TensorFlow/MXNet/Caffe2` 等多个开源框架

【参考】:

* 有三AI:【杂谈】当前模型量化有哪些可用的开源工具?<https://mp.weixin.qq.com/s?__biz=MzA3NDIyMjM1NA==&mid=2649037243&idx=1&sn=db2dc420c4d086fc99c7d8aada767484&chksm=8712a7c6b0652ed020872a97ea426aca1b06adf7571af3da6dac8ce991fd61001245e9bf6e9b&mpshare=1&scene=1&srcid=&sharer_sharetime=1576667804820&sharer_shareid=1d0dbdb37c6b95413d1d4fe7d61ed8f1&exportkey=A6g%2Fj50pMJYVXsedNyDVh9k%3D&pass_ticket=winxjBrzw0kHErbSri5yXS88yBx1a%2BAL9KKTG6Zt1MMS%2FeI2hpx%2BmeaLsrahnlOS#rd>

***网络 inference 阶段 Conv 层和 BN 层融合***

【参考】

* <https://zhuanlan.zhihu.com/p/110552861>
* PyTorch本身提供了类似的功能, 但是我没有使用过, 希望有朋友可以提供一些使用体会:<https://pytorch.org/docs/1.3.0/quantization.html#torch.quantization.fuse_modules>
* 网络inference阶段conv层和BN层的融合 - autocyz的文章 - 知乎 <https://zhuanlan.zhihu.com/p/48005099>

***多分支结构融合成单分支***

* ACNet、RepVGG这种设计策略也很有意思
  - RepVGG|让你的ConVNet一卷到底，plain网络首次超过80%top1精度：<https://mp.weixin.qq.com/s/M4Kspm6hO3W8fXT_JqoEhA>

### 时间分析

* Python 的 `cProfile` 可以用来分析.(Python 自带了几个性能分析的模块: `profile` ,  `cProfile` 和 `hotshot` , 使用方法基本都差不多, 无非模块是纯 Python 还是用 C 写的)

### 项目推荐

* 基于 Pytorch 实现模型压缩(<https://github.com/666DZY666/model-compression>):
    - 量化:8/4/2 bits(dorefa)、三值/二值(twn/bnn/xnor-net)
    - 剪枝: 正常、规整、针对分组卷积结构的通道剪枝
    - 分组卷积结构
    - 针对特征二值量化的BN融合

### 扩展阅读

* pytorch dataloader数据加载占用了大部分时间, 各位大佬都是怎么解决的? - 知乎 <https://www.zhihu.com/question/307282137>
* 使用pytorch时, 训练集数据太多达到上千万张, Dataloader加载很慢怎么办? - 知乎 <https://www.zhihu.com/question/356829360>
* PyTorch 有哪些坑/bug? - 知乎 <https://www.zhihu.com/question/67209417>
* <https://sagivtech.com/2017/09/19/optimizing-pytorch-training-code/>
* 26秒单GPU训练CIFAR10, Jeff Dean也点赞的深度学习优化技巧 - 机器之心的文章 - 知乎 <https://zhuanlan.zhihu.com/p/79020733>
* 线上模型加入几个新特征训练后上线, tensorflow serving预测时间为什么比原来慢20多倍? - TzeSing的回答 - 知乎 <https://www.zhihu.com/question/354086469/answer/894235805>
* 相关资料 · 语雀 <https://www.yuque.com/lart/gw5mta/bl3p3y>
* ShuffleNetV2:<https://arxiv.org/pdf/1807.11164.pdf>
* 今天, 你的模型加速了吗? 这里有5个方法供你参考(附代码解析): <https://mp.weixin.qq.com/s?__biz=MzI0ODcxODk5OA==&mid=2247511633&idx=2&sn=a5ab187c03dfeab4e64c85fc562d7c0d&chksm=e99e9da8dee914be3d713c41d5dedb7fcdc9982c8b027b5e9b84e31789913c5b2dd880210ead&mpshare=1&scene=1&srcid=&sharer_sharetime=1576934236399&sharer_shareid=1d0dbdb37c6b95413d1d4fe7d61ed8f1&exportkey=A%2B3SqYGse83qyFva%2BYSy3Ng%3D&pass_ticket=winxjBrzw0kHErbSri5yXS88yBx1a%2BAL9KKTG6Zt1MMS%2FeI2hpx%2BmeaLsrahnlOS#rd>
* pytorch常见的坑汇总 - 郁振波的文章 - 知乎 <https://zhuanlan.zhihu.com/p/77952356>
* Pytorch 提速指南 - 云梦的文章 - 知乎 <https://zhuanlan.zhihu.com/p/39752167>

## PyTorch节省显存

> 原始文档:<https://www.yuque.com/lart/ugkv9f/nvffyf>
>
> 整理自: Pytorch有什么节省内存(显存)的小技巧? - 知乎 <https://www.zhihu.com/question/274635237>

### 尽量使用 `inplace` 操作

尽可能使用 `inplace` 操作, 比如 `relu` 可以使用 `inplace=True` . 一个简单的使用方法, 如下:

``` python
def inplace_relu(m):
    classname = m.__class__.__name__
    if classname.find('ReLU') != -1:
        m.inplace=True

model.apply(inplace_relu)
```

进一步, 比如ResNet和DenseNet可以将 `batchnorm` 和 `relu` 打包成 `inplace` , 在bp时再重新计算. 使用到了pytorch新的 `checkpoint` 特性, 有以下两个代码. 由于需要重新计算bn后的结果, 所以会慢一些.

* [gpleiss/efficient_densenet_pytorch](https://github.com/gpleiss/efficient_densenet_pytorch)
* [In-Place Activated BatchNorm:mapillary/inplace_abn](https://github.com/mapillary/inplace_abn)

### 删除loss

每次循环结束时删除 loss, 可以节约很少显存, 但聊胜于无. 可见如下issue: [Tensor to Variable and memory freeing best practices](https://discuss.pytorch.org/t/tensor-to-variable-and-memory-freeing-best-practices/6000/2)

### 混合精度

使用 `Apex` 的混合精度或者是PyTorch1.6开始提供的`torch.cuda.amp`模块来训练. 可以节约一定的显存, 但是要小心一些不安全的操作如mean和sum:

* `NVIDIA/Apex` :
    - <https://blog.csdn.net/c9Yv2cf9I06K2A9E/article/details/100135729>
    - <https://github.com/nvidia/apex>
    - Pytorch 安装 APEX 疑难杂症解决方案 - 陈瀚可的文章 - 知乎<https://zhuanlan.zhihu.com/p/80386137>
    - <http://kevinlt.top/2018/09/14/mixed_precision_training/>
* `torch.cuda.amp`:
  - PyTorch的文档：<https://pytorch.org/docs/stable/notes/amp_examples.html>

### 对不需要反向传播的操作进行管理

* 对于不需要bp的forward, 如validation 请使用 `torch.no_grad` , 注意 `model.eval()` 不等于 `torch.no_grad()` , 请看如下讨论: ['model.eval()' vs 'with torch.no_grad()'](https://discuss.pytorch.org/t/model-eval-vs-with-torch-no-grad/19615)
* ~~将不需要更新的层的参数从优化器中排除~~将变量的 `requires_grad`设为 `False`, 让变量不参与梯度的后向传播(主要是为了减少不必要的梯度的显存占用)

### 显存清理

* `torch.cuda.empty_cache()` 这是 `del` 的进阶版, 使用 `nvidia-smi` 会发现显存有明显的变化. 但是训练时最大的显存占用似乎没变. 大家可以试试: [How can we release GPU memory cache?](https://discuss.pytorch.org/t/how-can-we-release-gpu-memory-cache/14530)
  + 具体效果有待斟酌, 亲自行测试
* 可以使用 `del` 删除不必要的中间变量, 或者使用 `replacing variables` 的形式来减少占用.

### 梯度累加

* 把一个 `batchsize=64` 分为两个32的batch, 两次forward以后, backward一次. 但会影响 `batchnorm` 等和 `batchsize` 相关的层. 在PyTorch的文档<https://pytorch.org/docs/stable/notes/amp_examples.html#gradient-accumulation>中提到了梯度累加与混合精度并用的例子. 
* 这篇文章提到使用梯度累加技术实现对于分布式训练的加速：<https://zhuanlan.zhihu.com/p/250471767>

### 使用 `checkpoint` 技术

#### `torch.utils.checkpoint`

**这是更为通用的选择.**

【参考】

* https://blog.csdn.net/one_six_mix/article/details/93937091
* https://pytorch.org/docs/1.3.0/_modules/torch/utils/checkpoint.html#checkpoint

#### Training Deep Nets with Sublinear Memory Cost

方法来自论文: [Training Deep Nets with Sublinear Memory Cost](https://arxiv.org/abs/1604.06174). 训练 CNN 时, Memory 主要的开销来自于储存用于计算 backward 的 activation, 一般的 workflow 是这样的

![](https://cdn.nlark.com/yuque/0/2019/gif/192314/1572487613013-4b7c7c2d-b622-4640-b977-eb804dbeaeba.gif#align=left&display=inline&height=121&originHeight=121&originWidth=541&size=0&status=done&style=none&width=541)

对于一个长度为 N 的 CNN, 需要 O(N) 的内存. 这篇论文给出了一个思路, 每隔 sqrt(N) 个 node 存一个 activation, 中需要的时候再算, 这样显存就从 O(N) 降到了 O(sqrt(N)).

![](https://cdn.nlark.com/yuque/0/2019/gif/192314/1572487613035-b9ceec0b-8a01-41ff-b51e-a3e6a7a36faf.gif#align=left&display=inline&height=121&originHeight=121&originWidth=541&size=0&status=done&style=none&width=541)

对于越深的模型, 这个方法省的显存就越多, 且速度不会明显变慢.

![image.png](https://cdn.nlark.com/yuque/0/2019/png/192314/1572487699952-d4e7d53e-a712-4697-908e-b6ab25fea5b4.png#align=left&display=inline&height=232&name=image.png&originHeight=376&originWidth=720&size=83464&status=done&style=none&width=445)

PyTorch 我实现了一版, 有兴趣的同学可以来试试: <https://github.com/Lyken17/pytorch-memonger>

本文章首发在 极市计算机视觉技术社区

【参考】: Pytorch有什么节省内存(显存)的小技巧? - Lyken的回答 - 知乎 https://www.zhihu.com/question/274635237/answer/755102181

### 相关工具

* These codes can help you to detect your GPU memory during training with Pytorch. [https://github.com/Oldpan/Pytorch-Memory-Utils](https://github.com/Oldpan/Pytorch-Memory-Utils)
* Just less than nvidia-smi? [https://github.com/wookayin/gpustat](https://github.com/wookayin/gpustat)

### 参考资料

* Pytorch有什么节省内存(显存)的小技巧? - 郑哲东的回答 - 知乎 https://www.zhihu.com/question/274635237/answer/573633662
* 浅谈深度学习: 如何计算模型以及中间变量的显存占用大小 [https://oldpan.me/archives/how-to-calculate-gpu-memory](https://oldpan.me/archives/how-to-calculate-gpu-memory)
* 如何在Pytorch中精细化利用显存 [https://oldpan.me/archives/how-to-use-memory-pytorch](https://oldpan.me/archives/how-to-use-memory-pytorch)
* Pytorch有什么节省显存的小技巧? - 陈瀚可的回答 - 知乎:https://www.zhihu.com/question/274635237/answer/756144739

## 其他技巧

### 重现

#### 强制确定性操作

**PyTorch 1.7新更新功能**

``` python
>>> import torch
>>> torch.set_deterministic(True)
```

【参考】:<https://pytorch.org/docs/stable/notes/randomness.html#avoiding-nondeterministic-algorithms>

#### 设置随机数种子

``` python
def seed_torch(seed=1029):
    random.seed(seed)
    os.environ['PYTHONHASHSEED'] = str(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed(seed)
    torch.cuda.manual_seed_all(seed) # if you are using multi-GPU.
    torch.backends.cudnn.benchmark = False
    torch.backends.cudnn.deterministic = True

seed_torch()
```

【参考】:<https://www.zdaiot.com/MLFrameworks/Pytorch/Pytorch%E9%9A%8F%E6%9C%BA%E7%A7%8D%E5%AD%90/>