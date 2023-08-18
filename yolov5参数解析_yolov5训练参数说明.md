# yolov5参数解析_yolov5训练参数说明
*   yolov5s：
    *   img 640,adam,epoch300,obj.yaml时，40epoch内都在0.45-0.6震荡。
    *   改为voc.yaml和sgd，epoch=100时，后期0.7-0.73震荡
*   yolov5x：
*   img=256.obj.yaml，0.75-0.8震荡。cache貌似没什么用

```python
 	Class     		 Images  Instances        P          R      mAP50   mAP50-95: 100% 7/7 [00:03<00:00,  2.24it/s]
           all         200         420     0.765      0.753      0.794      0.445
        crazing        200         83      0.509      0.449      0.455      0.163
      inclusion        200         90      0.709        0.8       0.85      0.456
 pitted_surface        200         48      0.923      0.833      0.883      0.541
      scratches        200         59      0.945      0.873      0.975      0.536
        patches        200         64      0.845      0.969      0.941      0.672
rolled-in_scale        200         76      0.659      0.592      0.662      0.301

```

一、模型配置文件
--------

  [YOLOv5](https://so.csdn.net/so/search?q=YOLOv5&spm=1001.2101.3001.7020)引入了`depth_multiple`和`width_multiple`系数来得到不同大小模型。

*   查看`models`文件夹下的各个模型配置文件，可以发现各个模型`Anchors`、`backbone`和`head`结构是一样的，只是`depth_multiple`和`width_multiple`两个参数不一样。

| 模型配置 | depth_multiple | width_multiple |
| --- | --- | --- |
| yolov5n | 0.33 | 0.25 |
| yolov5s | 0.33 | 0.5 |
| yolov5m | 0.67 | 0.75 |
| yolov5l | 1.0 | 1.0 |
| yolov5x | 1.33 | 1.25 |

*   `model/hub`文件夹下，有yolov5s6等v6.0版本的各个配置的模型。  
    ![](https://img-blog.csdnimg.cn/42fedfebc97d4416b2f050824644e716.png)
      
    yolov5s模型配置文件如下：

```
 `nc: 80  
depth_multiple: 0.33  
width_multiple: 0.50  
anchors:
  - [10,13, 16,30, 33,23]  
  - [30,61, 62,45, 59,119]  
  - [116,90, 156,198, 373,326]  


backbone:
  
  [[-1, 1, Conv, [64, 6, 2, 2]],  
   [-1, 1, Conv, [128, 3, 2]],  
   [-1, 3, C3, [128]],
   [-1, 1, Conv, [256, 3, 2]],  
   [-1, 6, C3, [256]],
   [-1, 1, Conv, [512, 3, 2]],  
   [-1, 9, C3, [512]],
   [-1, 1, Conv, [1024, 3, 2]],  
   [-1, 3, C3, [1024]],
   [-1, 1, SPPF, [1024, 5]],  
  ]

head:
  [[-1, 1, Conv, [512, 1, 1]],
   [-1, 1, nn.Upsample, [None, 2, 'nearest']],
   [[-1, 6], 1, Concat, [1]],  
   [-1, 3, C3, [512, False]],  

   [-1, 1, Conv, [256, 1, 1]],
   [-1, 1, nn.Upsample, [None, 2, 'nearest']],
   [[-1, 4], 1, Concat, [1]],  
   [-1, 3, C3, [256, False]],  

   [-1, 1, Conv, [256, 3, 2]],
   [[-1, 14], 1, Concat, [1]],  
   [-1, 3, C3, [512, False]],  

   [-1, 1, Conv, [512, 3, 2]],
   [[-1, 10], 1, Concat, [1]],  
   [-1, 3, C3, [1024, False]],  

   [[17, 20, 23], 1, Detect, [nc, anchors]],  
  ]` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48


```

*   from：输入来自那一层，-1代表上一次，1代表第1层，3代表第3层
*   number：模块的数量，最终数量需要乘width，然后四舍五入取整，如果小于1，取1。
*   module：子模块
*   args：模块参数，channel，kernel_size，stride，padding，bias等
*   Focus：对特征图进行切片操作，\[64,3\]得到\[3,32,3\]，即输入channel=3（RGB），输出为640.5(width_multiple)=32，3为卷积核尺寸。
*   Conv：nn.conv(kenel\_size=1，stride=1，groups=1，bias=False) + Bn + Leaky\_ReLu。\[-1, 1, Conv, \[128, 3, 2\]\]：输入来自上一层，模块数量为1个，子模块为Conv，网络中最终有1280.5=32个卷积核，卷积核尺寸为3，stride=2,。
*   BottleNeckCSP：借鉴CSPNet网络结构，由3个卷积层和X个残差模块Concat组成，若有False，则没有残差模块，那么组成结构为nn.conv+Bn+Leaky_ReLu
*   SPP：\[-1, 1, SPP, \[1024, \[5, 9, 13\]\]\]表示5×5，9×9，13×13的最大池化方式，进行多尺度融合。源代码如下：

二、超参文件
------

*   `yolov5/data/hyps/hyp.scratch-low.yaml` ：YOLOv5 COCO训练从头优化，数据增强低
*   `yolov5/data/hyps/hyp.scratch-mdeia.yaml`（数据增强中）
*   `yolov5/data/hyps/hyp.scratch-high.yaml`（数据增强高）

```
`lr0: 0.01  
lrf: 0.2  
momentum: 0.937  
weight_decay: 0.0005  
warmup_epochs: 3.0  
warmup_momentum: 0.8  
warmup_bias_lr: 0.1  
box: 0.05  
cls: 0.5  
cls_pw: 1.0  
obj: 1.0  
obj_pw: 1.0  
iou_t: 0.20  
anchor_t: 4.0  

fl_gamma: 0.0  
hsv_h: 0.015  
hsv_s: 0.7  
hsv_v: 0.4  
degrees: 0.0  
translate: 0.1  
scale: 0.5  
shear: 0.0  
perspective: 0.0  
flipud: 0.0  
fliplr: 0.5  
mosaic: 1.0  
mixup: 0.0` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31


```

三、运行过程参数解析
----------

### 3.1 `train.py`参数解析

> *   训练参数解析参考：[《手把手带你调参Yolo v5 (v6.2)（训练）》](https://blog.csdn.net/weixin_43694096/article/details/124411509)
> *   `train.py`代码解析参考[《【YOLOV5-5.x 源码解读】train.py》](https://blog.csdn.net/qq_38253797/article/details/119733964?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522166504157316782425139040%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=166504157316782425139040&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~pc_rank_v39-3-119733964-null-null.142%5Ev51%5Epc_rank_34_2,201%5Ev3%5Econtrol_2&utm_term=--evolve&spm=1018.2226.3001.4187)

```
`parser = argparse.ArgumentParser()
    parser.add_argument('--weights', type=str, default=ROOT / 'yolov5s.pt', help='initial weights path')
    parser.add_argument('--cfg', type=str, default='', help='model.yaml path')
    parser.add_argument('--data', type=str, default=ROOT / 'data/coco128.yaml', help='dataset.yaml path')
    parser.add_argument('--hyp', type=str, default=ROOT / 'data/hyps/hyp.scratch-low.yaml', help='hyperparameters path')
    parser.add_argument('--epochs', type=int, default=300, help='total training epochs')
    parser.add_argument('--batch-size', type=int, default=16, help='total batch size for all GPUs, -1 for autobatch')
    parser.add_argument('--imgsz', '--img', '--img-size', type=int, default=640, help='train, val image size (pixels)')
    parser.add_argument('--rect', action='store_true', help='rectangular training')
    parser.add_argument('--resume', nargs='?', const=True, default=False, help='resume most recent training')
    parser.add_argument('--nosave', action='store_true', help='only save final checkpoint')
    parser.add_argument('--noval', action='store_true', help='only validate final epoch')
    parser.add_argument('--noautoanchor', action='store_true', help='disable AutoAnchor')
    parser.add_argument('--noplots', action='store_true', help='save no plot files')
    parser.add_argument('--evolve', type=int, nargs='?', const=300, help='evolve hyperparameters for x generations')
    parser.add_argument('--bucket', type=str, default='', help='gsutil bucket')
    parser.add_argument('--cache', type=str, nargs='?', const='ram', help='--cache images in "ram" (default) or "disk"')
    parser.add_argument('--image-weights', action='store_true', help='use weighted image selection for training')
    parser.add_argument('--device', default='', help='cuda device, i.e. 0 or 0,1,2,3 or cpu')
    parser.add_argument('--multi-scale', action='store_true', help='vary img-size +/- 50%%')
    parser.add_argument('--single-cls', action='store_true', help='train multi-class data as single-class')
    parser.add_argument('--optimizer', type=str, choices=['SGD', 'Adam', 'AdamW'], default='SGD', help='optimizer')
    parser.add_argument('--sync-bn', action='store_true', help='use SyncBatchNorm, only available in DDP mode')
    parser.add_argument('--workers', type=int, default=8, help='max dataloader workers (per RANK in DDP mode)')
    parser.add_argument('--project', default=ROOT / 'runs/train', help='save to project/name')
    parser.add_argument('--name', default='exp', help='save to project/name')
    parser.add_argument('--exist-ok', action='store_true', help='existing project/name ok, do not increment')
    parser.add_argument('--quad', action='store_true', help='quad dataloader')
    parser.add_argument('--cos-lr', action='store_true', help='cosine LR scheduler')
    parser.add_argument('--label-smoothing', type=float, default=0.0, help='Label smoothing epsilon')
    parser.add_argument('--patience', type=int, default=100, help='EarlyStopping patience (epochs without improvement)')
    parser.add_argument('--freeze', nargs='+', type=int, default=[0], help='Freeze layers: backbone=10, first3=0 1 2')
    parser.add_argument('--save-period', type=int, default=-1, help='Save checkpoint every x epochs (disabled if < 1)')
    parser.add_argument('--seed', type=int, default=0, help='Global training seed')
    parser.add_argument('--local_rank', type=int, default=-1, help='Automatic DDP Multi-GPU argument, do not modify')

    
    parser.add_argument('--entity', default=None, help='Entity')
    parser.add_argument('--upload_dataset', nargs='?', const=True, default=False, help='Upload data, "val" option')
    parser.add_argument('--bbox_interval', type=int, default=-1, help='Set bounding-box image logging interval')
    parser.add_argument('--artifact_alias', type=str, default='latest', help='Version of dataset artifact to use')` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41


```

对照上表解析参数：

*   `weights`：指定预训练权重路径；如果这里设置为空的话，就是自己从头开始进行训练
*   `cfg`：模型配置文件，比如`models/yolov5s.yaml`。
*   `data`：数据集对应的yaml参数文件；里面主要存放数据集的类别和路径信息，例如：

```python
yaml:
names:
- crazing
- inclusion
- pitted_surface
- scratches
- patches
- rolled-in_scale
nc: 6
path: /kaggle/working/
train: /kaggle/working/train.txt
val: /kaggle/working/val.txt

```

*   `rect`:是否使用矩阵推理的方式训练模型

> 所谓矩阵推理就是不再要求你训练的图片是正方形了；矩阵推理会加速模型的推理过程，减少一些冗余信息。下图分别是矩阵推理方式和方形推理方式  
> ![](https://img-blog.csdnimg.cn/83c16d3c67c14719b83eb1d880fe4b69.png)

*   `resume`：断点续训：即是否在之前训练的一个模型基础上继续训练，default 值默认是 false。一种方式是先将train.py中这一行default=False 改为 default=True：

```python
parser.add_argument('--resume', nargs='?', const=True, default=True, help='resume most recent training')

```

然后执行代码：

```python
python train.py --resume \runs\train\exp\weights\last.pt

```

或者参考其他写法

*   `nosave`：是否只保存最后一轮的pt文件；我们默认是保存best.pt和last.pt两个的。
    
*   `noval`：是否只在最后一轮测试  
    正常情况下每个epoch都会计算mAP，但如果开启了这个参数，那么就只在最后一轮上进行测试，不建议开启。
    
*   `noautoanchor`：是否禁用自动锚框；默认是开启的。  
    yolov5中预先设定了一下锚框，这些锚框是针对coco数据集的，其他目标检测也适用，如下图所示：  
    ![](https://img-blog.csdnimg.cn/f696c5069e7841d482372a5023fb416a.png)
      
    如果开启了noautoanchor，在训练开始前，会自动计算数据集标注信息针对默认锚框的最佳召回率，当最佳召回率大于等于0.98时，则不需要更新锚框；如果最佳召回率小于0.98，则需要重新计算符合此数据集的锚框。建议不要改动此选项。
    
*   `noplots`：开启这个参数后将不保存绘图文件
    
*   `evolve`：yolov5使用[遗传](https://so.csdn.net/so/search?q=%E9%81%97%E4%BC%A0&spm=1001.2101.3001.7020)超参数进化，提供的默认参数是通过在COCO数据集上使用超参数进化得来的（也就是hpy文件夹下默认的超参数）。由于超参数进化会耗费大量的资源和时间，所以建议大家不要动这个参数。（开了貌似不评估mertic了）
    

> *   遗传算法是利用种群搜索技术将种群作为一组问题解，通过对当前种群施加类似生物遗传环境因素的选择、交叉、变异等一系列的遗传操作来产生新一代的种群，并逐步使种群优化到包含近似最优解的状态，遗传算法调优能够求出优化问题的全局最优解，优化结果与初始条件无关，算法独立于求解域，具有较强的鲁棒性，适合于求解复杂的优化问题，应用较为广泛。

*   `bucket`：谷歌云盘；通过这个参数可以下载谷歌云盘上的一些东西，但是现在没必要使用了
*   `cache`：是否提前缓存图片到内存，以加快训练速度，默认False；开启这个参数就会对图片进行缓存，从而更好的训练模型
*   `image-weights`：是否启用加权图像策略，默认是不开启的  
    **主要是为了解决样本不平衡问题。开启后会对于上一轮训练效果不好的图片，在下一轮中增加一些权重**
*   `multi-scale`：是否启用多尺度训练，默认是不开启的  
    多尺度训练是指设置几种不同的图片输入尺度，训练时每隔一定iterations随机选取一种尺度训练，这样训练出来的模型鲁棒性更强。

>   输入图片的尺寸对检测模型的性能影响很大，在基础网络部分常常会生成比原图小数十倍的特征图，导致小物体的特征描述不容易被检测网络捕捉。通过输入更大、更多尺寸的图片进行训练，能够在一定程度上提高检测模型对物体大小的鲁棒性。

*   `single-cls`：训练数据集是否是单类别，默认False
*   `optimizer`：优化器；默认为SGD，可选SGD，Adam，AdamW。
*   `sync-bn`：是否开启跨卡同步BN  
    开启后，可使用 SyncBatchNorm 进行多 GPU分布式训练
*   `workers`：进程数
*   `project`：指定模型的保存路径；默认在runs/train。
*   `name`：模型保存的文件夹名，默认在exp文件夹。
*   `exist-ok`：每次模型预测结果是否保存在原来的文件夹
    *   如果指定了这个参数的话，那么本次预测的结果还是保存在上一次保存的文件夹里
    *   如果不指定，就是每预测一次结果，就保存在一个新的文件夹里。
*   `quad`：暂不明
*   `cos-lr`：是否开启余弦学习率。（下面是开启前后学习率曲线对比图）  
    ![](https://img-blog.csdnimg.cn/cb5fa314b852456484821a714686ebb2.png)
    
*   `label-smoothing`：是否启用标签平滑处理，默认不启用
*   `patience`：早停轮数，默认100。  
    如果模型在100轮里没有提升，则停止训练模型
*   `freeze`：指定冻结层数量；可以在yolov5s.yaml中查看主干网络层数

>   冻结训练是迁移学习常用的方法。当数据集较小时，我们会选择预训练好的模型进行微调。大型数据集预训练好的权重主干特征提取能力是比较强的，这个时候我们只需要冻结主干网络，fine-tune后面层就可以了，不需要从头开始训练，大大减少了时间而且还提高了性能。

```python
python train.py --freeze 8 

```

```python

backbone:
  
  [[-1, 1, Conv, [64, 6, 2, 2]],  
   [-1, 1, Conv, [128, 3, 2]],  
   [-1, 3, C3, [128]],
   [-1, 1, Conv, [256, 3, 2]],  
   [-1, 6, C3, [256]],
   [-1, 1, Conv, [512, 3, 2]],  
   [-1, 9, C3, [512]],
   [-1, 1, Conv, [1024, 3, 2]],  
   [-1, 3, C3, [1024]],
   [-1, 1, SPPF, [1024, 5]],  
  ]

```

下面是一个项目的冻结效果对比图：  
![](https://img-blog.csdnimg.cn/a75837b19cc44a9d9e80a100959333aa.png)

*   `save-period`：多少个epoch保存一下checkpoint，default=-1。
*   `seed`：随机种子。v6.2版本更新的一个非常重要的参数，使用torch>=1.12.0的单GPU YOLOv5训练现在完全可再现
*   `local_rank`：DistributedDataParallel 单机多卡训练，单GPU设备不需要设置
*   `entity`：在线可视化工具wandb
*   `upload_dataset`：是否上传dataset到wandb tabel，默认False  
    启用后，将数据集作为交互式 dsviz表 在浏览器中查看、查询、筛选和分析数据集
*   `bbox_interval`：设置界框图像记录间隔 Set bounding-box image logging interval for W&B 默认-1
*   `artifact_alias`：使用数据的版本

### 3.2 `detact.py`参数解析

下面是推理脚本`detect.py`的参数：

```
`def parse_opt():
    parser = argparse.ArgumentParser()
    parser.add_argument('--weights', nargs='+', type=str, default=ROOT / 'yolov5s.pt', help='model path or triton URL')
    parser.add_argument('--source', type=str, default=ROOT / 'data/images', help='file/dir/URL/glob/screen/0(webcam)')
    parser.add_argument('--data', type=str, default=ROOT / 'data/coco128.yaml', help='(optional) dataset.yaml path')
    parser.add_argument('--imgsz', '--img', '--img-size', nargs='+', type=int, default=[640], help='inference size h,w')
    parser.add_argument('--conf-thres', type=float, default=0.25, help='confidence threshold')
    parser.add_argument('--iou-thres', type=float, default=0.45, help='NMS IoU threshold')
    parser.add_argument('--max-det', type=int, default=1000, help='maximum detections per image')
    parser.add_argument('--device', default='', help='cuda device, i.e. 0 or 0,1,2,3 or cpu')
    parser.add_argument('--view-img', action='store_true', help='show results')
    parser.add_argument('--save-txt', action='store_true', help='save results to *.txt')
    parser.add_argument('--save-conf', action='store_true', help='save confidences in --save-txt labels')
    parser.add_argument('--save-crop', action='store_true', help='save cropped prediction boxes')
    parser.add_argument('--nosave', action='store_true', help='do not save images/videos')
    parser.add_argument('--classes', nargs='+', type=int, help='filter by class: --classes 0, or --classes 0 2 3')
    parser.add_argument('--agnostic-nms', action='store_true', help='class-agnostic NMS')
    parser.add_argument('--augment', action='store_true', help='augmented inference')
    parser.add_argument('--visualize', action='store_true', help='visualize features')
    parser.add_argument('--update', action='store_true', help='update all models')
    parser.add_argument('--project', default=ROOT / 'runs/detect', help='save results to project/name')
    parser.add_argument('--name', default='exp', help='save results to project/name')
    parser.add_argument('--exist-ok', action='store_true', help='existing project/name ok, do not increment')
    parser.add_argument('--line-thickness', default=3, type=int, help='bounding box thickness (pixels)')
    parser.add_argument('--hide-labels', default=False, action='store_true', help='hide labels')
    parser.add_argument('--hide-conf', default=False, action='store_true', help='hide confidences')
    parser.add_argument('--half', action='store_true', help='use FP16 half-precision inference')
    parser.add_argument('--dnn', action='store_true', help='use OpenCV DNN for ONNX inference')
    parser.add_argument('--vid-stride', type=int, default=1, help='video frame-rate stride')
    opt = parser.parse_args()
    opt.imgsz *= 2 if len(opt.imgsz) == 1 else 1  
    print_args(vars(opt))
    return opt` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33


```

> NMS步骤：
> 
> 1.  对 BBox 按置信度排序，选取置信度最高的 BBox（所以一开始置信度最高的 BBox 一定会被留下来）；
> 2.  对剩下的 BBox 和已经选取的 BBox 计算 IOU，淘汰（抑制） IOU 大于设定阈值的 BBox（在图例中这些淘汰的 BBox 的置信度被设定为0）。
> 3.  重复上述两个步骤，直到所有的 BBox 都被处理完，这时候每一轮选取的 BBox 就是最后结果。

### 3.3 `val.py`参数解析

`val.py`的作用：

1.  我们在训练结束后会打印出每个类别的一些评价指标，但是如果当时忘记记录，那么我们就可以通过这个文件再次打印这些评价指标
2.  可以打印出测试集评价指标，测试集的图片也是需要标注的。

```
`def parse_opt():
    parser = argparse.ArgumentParser()
    parser.add_argument('--data', type=str, default=ROOT / 'data/coco128.yaml', help='dataset.yaml path')
    parser.add_argument('--weights', nargs='+', type=str, default=ROOT / 'yolov5s.pt', help='model path(s)')
    parser.add_argument('--batch-size', type=int, default=32, help='batch size')
    parser.add_argument('--imgsz', '--img', '--img-size', type=int, default=640, help='inference size (pixels)')
    parser.add_argument('--conf-thres', type=float, default=0.001, help='confidence threshold')
    parser.add_argument('--iou-thres', type=float, default=0.6, help='NMS IoU threshold')
    parser.add_argument('--max-det', type=int, default=300, help='maximum detections per image')
    parser.add_argument('--task', default='val', help='train, val, test, speed or study')
    parser.add_argument('--device', default='', help='cuda device, i.e. 0 or 0,1,2,3 or cpu')
    parser.add_argument('--workers', type=int, default=8, help='max dataloader workers (per RANK in DDP mode)')
    parser.add_argument('--single-cls', action='store_true', help='treat as single-class dataset')
    parser.add_argument('--augment', action='store_true', help='augmented inference')
    parser.add_argument('--verbose', action='store_true', help='report mAP by class')
    parser.add_argument('--save-txt', action='store_true', help='save results to *.txt')
    parser.add_argument('--save-hybrid', action='store_true', help='save label+prediction hybrid results to *.txt')
    parser.add_argument('--save-conf', action='store_true', help='save confidences in --save-txt labels')
    parser.add_argument('--save-json', action='store_true', help='save a COCO-JSON results file')
    parser.add_argument('--project', default=ROOT / 'runs/val', help='save to project/name')
    parser.add_argument('--name', default='exp', help='save to project/name')
    parser.add_argument('--exist-ok', action='store_true', help='existing project/name ok, do not increment')
    parser.add_argument('--half', action='store_true', help='use FP16 half-precision inference')
    parser.add_argument('--dnn', action='store_true', help='use OpenCV DNN for ONNX inference')
    opt = parser.parse_args()
    opt.data = check_yaml(opt.data)  
    opt.save_json |= opt.data.endswith('coco.yaml')
    opt.save_txt |= opt.save_hybrid
    print_args(vars(opt))
    return opt` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30


```

前六个参数和detect.py意义一样。

*   task：可以是train, val, test。比如：`python val.py --task test`表示打印测试集指标
*   augment：测试是否使用TTA Test Time Augment，指定这个参数后各项指标会明显提升几个点。
*   verbose：是否打印出每个类别的mAP，默认False。
*   save-hybrid：将标签+预测混合结果保存到 .txt
*   save-json：是否按照coco的json格式保存预测框，并且使用cocoapi做评估（需要同样coco的json格式的标签） 默认False
*   half：是否使用半精度推理 默认False

其它参数内容同`detect.py`。

五、添加[注意力机制](https://so.csdn.net/so/search?q=%E6%B3%A8%E6%84%8F%E5%8A%9B%E6%9C%BA%E5%88%B6&spm=1001.2101.3001.7020)
------------------------------------------------------------------------------------------------------------------

> *   参考[《手把手带你Yolov5 (v6.1)添加注意力机制(一)（并附上30多种顶会Attention原理图）》](https://blog.csdn.net/weixin_43694096/article/details/124443059?spm=1001.2014.3001.5502)
> *   参考[《手把手带你Yolov5 (v6.1)添加注意力机制(二)（在C3模块中加入注意力机制）》](https://yolov5.blog.csdn.net/article/details/124695537)

```python
Epoch    GPU_mem   box_loss   obj_loss   cls_loss  Instances       Size
98/99      3.81G    0.03403     0.0175   0.002002         72        256: 100% 75/75 [00:29<00:00,  2.52it/s]
            Class     Images  Instances          P          R      mAP50   mAP50-95: 100% 7/7 [00:02<00:00,  2.93it/s]
              all        200        420      0.765      0.753      0.794      0.445

```