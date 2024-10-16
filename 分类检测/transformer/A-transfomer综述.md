

# Transformer综述

网址：https://aistudio.baidu.com/aistudio/education/lessonvideo/1537947

zhongshu: https://mp.weixin.qq.com/s/tloHFx-ngO8h7Oi3m4rjrg

## Attention发展阶段

![image-20211122185500236](..\..\images\image-20211122185500236.png)

## Transformer注意力机制

![image-20211123113753714](..\..\images\image-20211123113753714.png)

![image-20211123114736683](..\..\images\image-20211123114736683.png)

![image-20211123114831792](..\..\images\image-20211123114831792.png)

![image-20211123115008000](..\..\images\image-20211123115008000.png)

![image-20211123115100159](..\..\images\image-20211123115100159.png)

![image-20211123115201833](..\..\images\image-20211123115201833.png)

![image-20211123115336664](..\..\images\image-20211123115336664.png)

![image-20211123133801422](..\..\images\image-20211123133801422.png)

效果展示

![image-20211123134020970](..\..\images\image-20211123134020970.png)

![image-20211123134050769](..\..\images\image-20211123134050769.png)

Transfromer 优缺点：

| 优点 | <img src="..\..\images\image-20211123134357591.png" />       |
| :--- | ------------------------------------------------------------ |
| 缺点 | ![image-20211123134502493](..\..\images\image-20211123134502493.png) |

## Transformer在CV中的应用

### ViT

#### 论文：

![image-20211123141220008](..\..\images\image-20211123141220008.png)

虚拟开始块：可以和其他所有的块进行交互，可以提取其他所有的语义信息，因此可以利用虚拟块去做全图的分类特征。

ViT为啥没有Decoder：只是分类任务，只需提取特征，Encoder足以。

#### 代码：

```python
'''
参数配置
'''
train_parameters = {
    "input_size": [3, 120, 120],                             #输入图片的shape
    "class_dim": 3,                                          #分类数
    "src_path":"/home/aistudio/data/data72920/Data.zip",     #原始数据集路径
    "target_path":"/home/aistudio/work/",                    #要解压的路径
    "train_list_path": "/home/aistudio/data/train.txt",      #train.txt路径
    "eval_list_path": "/home/aistudio/data/eval.txt",        #eval.txt路径
    "label_dict":{'0':'汽车','1':'摩托车','2':'货车'},        #标签字典
    "num_epochs": 40,                                        #训练轮数
    "train_batch_size": 32,                                   #训练时每个批次的大小
    "learning_strategy": {                                   #优化函数相关的配置
        "lr": 1.0e-5                                        #超参数学习率
    }, 
    'skip_steps': 50,                                        #每N个批次打印一次结果
    'save_steps': 500,                                       #每N个批次保存一次模型参数
    "checkpoints": "/home/aistudio/work/checkpoints"         #保存的路径

}

def seed_paddle(seed=1024):
    seed = int(seed)
    random.seed(seed)
    os.environ['PYTHONHASHSEED'] = str(seed)
    np.random.seed(seed)
    paddle.seed(seed)

seed_paddle(seed=1024)

'''
构建模型
'''

# 定义ViT
trunc_normal_ = TruncatedNormal(std=.02)
zeros_ = Constant(value=0.)
ones_ = Constant(value=1.)
dim =256
heads=16
patch_size = 8
num_layers = 1
num_patch = int((120/patch_size) * (120/patch_size))

# x[int] -> tuple(x, x)
def to_2tuple(x):
    return tuple([x] * 2)

# 独立层，即什么操作都没有的网络层
class Identity(nn.Layer):
    def __init__(self):
        super(Identity, self).__init__()

    def forward(self, input):
        return input

class PatchEmbed(nn.Layer):
    def __init__(self, img_size=120, patch_size=patch_size, in_chans=3, embed_dim=dim):
        super().__init__()
        img_size = to_2tuple(img_size)
        patch_size = to_2tuple(patch_size)
        num_patches = (img_size[1] // patch_size[1]) * \
            (img_size[0] // patch_size[0])
        self.img_size = img_size
        self.patch_size = patch_size
        self.num_patches = num_patches

        self.proj = nn.Conv2D(in_chans, embed_dim,
                              kernel_size=patch_size, stride=patch_size)

    def forward(self, x):
        B, C, H, W = x.shape
        # 分块线性变换 + 向量展平 + 维度转置
        x = self.proj(x).flatten(2).transpose((0, 2, 1)) # 每一块输出：1x1xembed_dim
        return x

class Attention(nn.Layer):  # 多头注意力层
    def __init__(self, dim, num_heads=8, qkv_bias=False, qk_scale=None, attn_drop=0., proj_drop=0.):
        super().__init__()
        self.num_heads = num_heads
        head_dim = dim // num_heads
        self.scale = qk_scale or head_dim ** -0.5

        self.qkv = nn.Linear(dim, dim * 3, bias_attr=qkv_bias)
        self.attn_drop = nn.Dropout(attn_drop)
        self.proj = nn.Linear(dim, dim)
        self.proj_drop = nn.Dropout(proj_drop)

    def forward(self, x):
        B, N, C = x.shape

        # 线性变换
        qkv = self.qkv(x).reshape((B, N, 3, self.num_heads, C //
                                   self.num_heads)).transpose((2, 0, 3, 1, 4))
        
        # 分割 query key value
        q, k, v = qkv[0], qkv[1], qkv[2]
        
        # Matmul + Scale
        attn = (q.matmul(k.transpose((0, 1, 3, 2)))) * self.scale

        # SoftMax
        attn = nn.functional.softmax(attn, axis=-1)
        
        # Attention Dropout
        attn = self.attn_drop(attn)
        
        # Matmul
        x = (attn.matmul(v)).transpose((0, 2, 1, 3)).reshape((B, N, C))

        # 线性变换
        x = self.proj(x)

        # Linear Dropout
        x = self.proj_drop(x)
        return x

class Mlp(nn.Layer):
    def __init__(self, in_features, hidden_features=None, out_features=None, act_layer=nn.GELU, drop=0.):
        super().__init__()
        out_features = out_features or in_features
        hidden_features = hidden_features or in_features
        self.fc1 = nn.Linear(in_features, hidden_features)
        self.act = act_layer()
        self.fc2 = nn.Linear(hidden_features, out_features)
        self.drop = nn.Dropout(drop)

    def forward(self, x): 
        x = self.fc1(x) 
        x = self.act(x) 
        x = self.drop(x) 
        x = self.fc2(x) 
        x = self.drop(x)
        return x

def drop_path(x, drop_prob=0., training=False):
    """Drop paths (Stochastic Depth) per sample (when applied in main path of residual blocks).
    the original name is misleading as 'Drop Connect' is a different form of dropout in a separate paper...
    See discussion: https://github.com/tensorflow/tpu/issues/494#issuecomment-532968956 ...
    """
    if drop_prob == 0. or not training:
        return x
    keep_prob = paddle.to_tensor(1 - drop_prob)
    shape = (x.shape[0],) + (1,) * (x.ndim - 1)
    random_tensor = keep_prob + paddle.rand(shape, dtype=x.dtype)
    random_tensor = paddle.floor(random_tensor) # binarize
    output = x.divide(keep_prob) * random_tensor
    return output


class DropPath(nn.Layer):
    """Drop paths (Stochastic Depth) per sample  (when applied in main path of residual blocks).
    """

    def __init__(self, drop_prob=None):
        super(DropPath, self).__init__()
        self.drop_prob = drop_prob

    def forward(self, x):
        return drop_path(x, self.drop_prob, self.training)

class Block(nn.Layer):
    def __init__(self, dim, num_heads, mlp_ratio=4., qkv_bias=False, qk_scale=None, drop=0., attn_drop=0.,
                 drop_path=0., act_layer=nn.GELU, norm_layer='nn.LayerNorm', epsilon=1e-5):
        super().__init__()
        self.norm1 = eval(norm_layer)(dim, epsilon=epsilon)
        self.attn = Attention(
            dim, num_heads=num_heads, qkv_bias=qkv_bias, qk_scale=qk_scale, attn_drop=attn_drop, proj_drop=drop)
        # NOTE: drop path for stochastic depth, we shall see if this is better than dropout here
        self.drop_path = DropPath(drop_path) if drop_path > 0. else Identity()
        self.norm2 = eval(norm_layer)(dim, epsilon=epsilon)
        mlp_hidden_dim = int(dim * mlp_ratio)
        self.mlp = Mlp(in_features=dim, hidden_features=mlp_hidden_dim,
                       act_layer=act_layer, drop=drop)

    def forward(self, x):
        # Norm + Attention + DropPath + Residual Connect
        x = x + self.drop_path(self.attn(self.norm1(x)))
        
        # Norm + MLP + DropPath + Residual Connect
        x = x + self.drop_path(self.mlp(self.norm2(x)))
        return x

class VisionTransformer(nn.Layer):
    def __init__(self, img_size=120, patch_size=patch_size, in_chans=3, class_dim=train_parameters['class_dim'], embed_dim=dim, depth=num_layers,
                 num_heads=heads, mlp_ratio=4, qkv_bias=False, qk_scale=None, drop_rate=0., attn_drop_rate=0.,
                 drop_path_rate=0., norm_layer='nn.LayerNorm', epsilon=1e-5, **args):
        super().__init__()
        self.class_dim = class_dim

        self.num_features = self.embed_dim = embed_dim

        self.patch_embed = PatchEmbed(
            img_size=img_size, patch_size=patch_size, in_chans=in_chans, embed_dim=embed_dim)
        num_patches = self.patch_embed.num_patches

        self.pos_embed = self.create_parameter(  # 位置编码 + 1 虚拟块首
            shape=(1, num_patches + 1, embed_dim), default_initializer=zeros_)
        self.add_parameter("pos_embed", self.pos_embed)
        self.cls_token = self.create_parameter(  # 虚拟块首
            shape=(1, 1, embed_dim), default_initializer=zeros_)
        self.add_parameter("cls_token", self.cls_token)
        self.pos_drop = nn.Dropout(p=drop_rate)

        dpr = [x for x in paddle.linspace(0, drop_path_rate, depth)]

        self.blocks = nn.LayerList([
            Block(
                dim=embed_dim, num_heads=num_heads, mlp_ratio=mlp_ratio, qkv_bias=qkv_bias, qk_scale=qk_scale,
                drop=drop_rate, attn_drop=attn_drop_rate, drop_path=dpr[i], norm_layer=norm_layer, epsilon=epsilon)
            for i in range(depth)])

        self.norm = eval(norm_layer)(embed_dim, epsilon=epsilon)

        # Classifier head
        self.head = nn.Linear(
            embed_dim, class_dim) if class_dim > 0 else Identity()

        trunc_normal_(self.pos_embed)
        trunc_normal_(self.cls_token)
        self.apply(self._init_weights)
    
    # 参数初始化
    def _init_weights(self, m):
        if isinstance(m, nn.Linear):
            trunc_normal_(m.weight)
            if isinstance(m, nn.Linear) and m.bias is not None:
                zeros_(m.bias)
        elif isinstance(m, nn.LayerNorm):
            zeros_(m.bias)
            ones_(m.weight)
    
    # 获取图像特征
    def forward_features(self, x):
        B = x.shape[0] 
        # Image Patch Embedding
        x = self.patch_embed(x) 
        # 分类 tokens
        cls_tokens = self.cls_token.expand((B, -1, -1)) 
        # 拼接 Embedding 和 分类 tokens
        x = paddle.concat((cls_tokens, x), axis=1) 
        # 加入位置嵌入 Position Embedding
        x = x + self.pos_embed 
        # Embedding Dropout
        x = self.pos_drop(x)
        # Transformer Encoder
        # 由多个基础模块组成
        for blk in self.blocks:
            x = blk(x) 
        # Norm
        x = self.norm(x) 
        # 提取分类 tokens 的输出
        return x[:, 0]
    
    def forward(self, x):
        x = paddle.reshape(x, shape=[-1, 3,120,120])
        # 获取图像特征
        x = self.forward_features(x) 
        # 图像分类 
        x = self.head(x) 
        return x
```

#### ViT中的超参数：

① Encoder-Decoder的层数 ② 头数 ③ 维度 ④ drop-out/drop path 比例 ⑤ patch-size个数

### DERT

![image-20211123141358664](..\..\images\image-20211123141358664.png)
