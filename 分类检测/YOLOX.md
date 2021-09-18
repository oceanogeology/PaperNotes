- Github： https://github.com/Megvii-BaseDetection/YOLOX.

## Abstract

- 引入anchor-free
- decoupled head ,simOTA
- Performance:  yolo-nano:  1.8% ap； yolov3 : 3%; YOLOv5-L:1.8% ; 
- provide tensorrt onnx version

##  Introduction

- 近两年目标检测的研究热点： anchor-free； advanced label assignment strategies[37,36,12,41,22,4]; NMs-free detectors[2,32,39]
- YOLOv3 as default
- we boost the YOLOv3 to 47.3%AP (YOLOX-DarkNet53) on COCO with 640 × 640 resolution, surpassing the current best practice of YOLOv3(44.3% AP, ultralytics version2) by a large margin.'
- YOLOv5 640x640； 50.0% AP, supass 1.8%AP
- 

## YOLOX

### Implementation details

- 300 epoch , 5 epoch warmup
- lr = init_lr * (batchsize/64),    init_lr = 0.01,   cos lr schedule,  (8-gpu, batchsize:128)
- input size : 448 to 832 , 32 strides

### YOLOv3 baseline

- DarkNet53 +  SPPlayer
- ==adding EMA weights updating, cosine lr schedule, IoU loss and IoU-aware branch.  We use BCE Loss for training cls and obj branch, and IoU Loss for training reg branch.==

### Decoupled head



![image-20210918121226730](..\images\image-20210918121226730.png)

![image-20210918121247956](..\images\image-20210918121247956.png)

