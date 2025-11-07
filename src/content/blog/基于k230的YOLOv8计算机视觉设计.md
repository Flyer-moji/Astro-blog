---
title: 基于k230的YOLOv8计算机视觉设计（对鱼）
description: 我的竞赛项目笔记
pubDate: 11 29 2024
image: ""
categories:
  - tech
tags:
  - yolov8
---

## yolov8 概述

YOLOv8 是由 ultralytics 团队在 2020 年 11 月发布的最新版本的目标检测模型，相比于之前的版本 YOLOv3、YOLOv4、YOLOv5，YOLOv8 主要有以下改进：

1. 更大的感受野：YOLOv8 相比于之前的版本，在感受野上有了更大的提升，可以检测更大的目标。
2. 更多的锚点：在锚点的数量上有了更大的提升，可以检测更多的目标。
3. 更好的精度：在精度上有了更大的提升，可以检测更准确的目标。
4. 更好的速度：在速度上有了更大的提升，可以实时处理视频流。

### yolov8 架构

| 层级 | 名称 |作用 |
| --- | --- | --- |
|框架 | pytorch | 提供通用的深度学习计算图、自动求导、GPU 加速等底层功能。 |
|算法实现| yolov8 | 利用 PyTorch 实现的具体目标检测模型（含网络结构、训练/推理逻辑）。 |
|库| ultralytics/yolov8 | 基于 PyTorch 的 YOLOv8 库，提供了丰富的API 接口、训练/推理配置、数据集、模型、工具等。通过安装 ultralytics 这个 Python 库来使用 YOLOv8 |

## 对鱼识别-数据集标注

### 1.数据采集

在智慧鱼缸项目中，模型的目标是识别不同种类鱼并测量体长。因此，数据采集阶段的核心任务是：

获取包含各种鱼类（红金鱼、狮子金鱼、孔雀鱼、锦鲤等）的高分辨率图片；

尽量覆盖鱼在不同姿态、光照、距离、角度下的情况；

确保数据分布接近实际使用环境（真实鱼缸场景）。

### 2.数据筛选与清洗

去除模糊、曝光过度、光反射严重的图片；

剔除重复帧或同场景多帧相似图片（可通过哈希去重）。

确保每类鱼的样本数量相对平衡（防止模型类别偏差）。

### 3.标注阶段技术细节

标注是数据采集阶段的延伸部分，这里是你能展现“细节能力”的地方。

| 步骤        | 工具与格式                                                                | 说明 |
| --------- | -------------------------------------------------------------------- | -- |
| **工具**    | LabelImg（Python + Qt）                                                |    |
| **格式**    | YOLO TXT 格式（每行：`class x_center y_center width height`，坐标归一化到 [0, 1]） |    |
| **类别定义**  | red_goldfish, lionhead, koi, guppy 等 4 – 5 种鱼类                       |    |
| **样本数量**  | 共标注 4127 条鱼体边界框                                                      |    |
| **检查与修正** | 随机抽查 10% 样本，核对标签准确率；若类别或框偏移，手动修正。                                    |    |

#### 3.1 标注工具

LabelImg 是一个开源图形化图像标注工具，用 Python + Qt 写的，主要用于给图像打上 矩形边界框（Bounding Box） 并指定类别。他是一个可视化图形界面，可以方便地标注多张图片，并可以保存为 YOLO TXT 格式。
输出可选 PASCAL VOC (XML)（默认） 或 YOLO (TXT) 格式，后者正好是 YOLOv8 所使用的格式。

![](https://pica.zhimg.com/v2-579f7dc65caedf1f3e17cae168dec944_r.jpg)

#### 3.2 标注格式

YOLO TXT 格式（每行：`class x_center y_center width height`，坐标归一化到 [0, 1]）

每张图生称一个对应名称的.txt 文件，文件中每行对应一个边界框(图里有多只鱼，每只鱼对应一行)，格式为：
```
class x_center y_center width height
```
| 字段         | 含义        | 数值范围       | 说明                    |
| ---------- | --------- | ---------- | --------------------- |
| `class_id` | 类别编号（整数）  | 0, 1, 2, … | 从 0 开始计数，比如 0=金鱼，1=锦鲤 |
| `x_center` | 边框中心点的横坐标 | 0–1        | 相对于整张图宽度归一化           |
| `y_center` | 边框中心点的纵坐标 | 0–1        | 相对于整张图高度归一化           |
| `width`    | 边框宽度      | 0–1        | 相对于整张图宽度归一化           |
| `height`   | 边框高度      | 0–1        | 相对于整张图高度归一化           |

#### 3.3 标注样本比例

按照7：2：1的比例，训练集：测试集：验证集。
| 数据集                      | 主要作用           | 是否参与模型训练         | 说明                    |
| ------------------------ | -------------- | ---------------- | --------------------- |
| **训练集 (Training Set)**   | 用于让模型学习特征与权重参数 | ✅ 是              | 模型在此数据上计算损失并反向传播更新参数。 |
| **验证集 (Validation Set)** | 用于调整模型结构和超参数   | ⚠️ 仅前向推理，不参与反向传播 | 评估模型是否过拟合、判断何时停止训练。   |
| **测试集 (Test Set)**       | 用于最终性能评估       | ❌ 否              | 仅在训练完成后使用，检验模型的泛化能力。  |
##### 3.3.1 划分方法

随机划分

用脚本或 YOLOv8 自带的数据集配置工具随机打乱划分，避免类别集中某一子集导致偏差。

类别平衡

保证不同鱼类（红金鱼、锦鲤、孔雀鱼等）在各子集中的占比一致，防止验证或测试集只包含某类鱼。

## 对鱼识别-模型训练

### 环境配置

| 项目         | 内容                                                  |
| ---------- | --------------------------------------------------- |
| **开发环境**   | PyCharm + Python 3.x                                |
| **深度学习框架** | PyTorch                                             |
| **训练工具**   | Ultralytics YOLOv8（通过 `pip install ultralytics` 安装） |
| **运行平台**   | PC 端训练后再部署至 CanMV K230 板                            |
| **GPU**    | NVIDIA 显卡（若使用 CUDA 加速）                              |
| **数据集**    | 自建鱼类数据集（2611 张图片，7:2:1 划分）                          |

### 训练原理

[参考视频-bilibili_YOLO算法原理讲解](https://www.bilibili.com/video/BV1sR4y1h7s4)

### 损失函数

| 损失项          | 含义                               | 公式/说明                                              |
| ------------ | -------------------------------- | -------------------------------------------------- |
| **box_loss** | 边界框定位误差                          | 使用 IoU（Intersection over Union）或 CIoU 计算预测框与真实框重叠度 |
| **cls_loss** | 分类误差                             | 使用交叉熵损失（Cross Entropy）衡量预测类别与真实类别差距                |
| **dfl_loss** | 分布焦点损失 (Distribution Focal Loss) | 用于更精确的边界框坐标回归，提高定位精度      |

训练过程中，损失值应持续下降并趋于稳定。

### 评价指标

| 指标名称             | 说明                 | 理想趋势        |
| ---------------- | ------------------ | ----------- |
| **Precision**    | 模型预测为“鱼”的样本中有多少是真鱼 | 越高越好        |
| **Recall**       | 实际存在鱼的样本中有多少被检测出来  | 越高越好        |
| **mAP@0.5**      | IoU（交并比） 阈值 0.5 下的平均精度  | 越高越好        |
| **mAP@0.5:0.95** | 多阈值平均精度，更严格的指标     | 越高越好        |
| **val_loss**     | 验证集上的损失            | 越低越好（防止过拟合） |

### 训练过程

#### 1.数据集准备

将数据集按照 YOLOv8 要求的格式整理好，并放入指定目录。
```bash
dataset/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
└── labels/
    ├── train/
    ├── val/
    └── test/
```
写一个 data.yaml 文件告诉 YOLO 数据集位置和类别：
```yaml
path: ./dataset
train: images/train
val: images/val
test: images/test

names:
  0: red_goldfish
  1: lionhead
  2: koi
  3: guppy
```

#### 2.训练配置

```python
# train_fish.py
from ultralytics import YOLO

# 载入预训练模型
model = YOLO("yolov8n.pt")

# 开始训练
results = model.train(
    data="data.yaml",    # 数据集配置文件
    epochs=102,          # 训练轮次
    imgsz=640,           # 输入图像尺寸
    batch=16,            # 每批次图片数
    workers=4,           # 数据加载线程数
    lr0=0.01,            # 初始学习率
    device=0,            # 使用 GPU 0；若无 GPU 则设为 'cpu'
    project="runs/train",# 输出结果保存路径
    name="fish_yolov8",  # 训练任务名
    pretrained=True,     # 使用预训练权重加速收敛
    verbose=True         # 打印详细日志
)
```
运行：
```bash
python train_fish.py
```

#### 3.训练结果

训练过程会输出日志，包括训练损失、验证损失、mAP 等指标。

训练完成后，会在 `runs/train` 目录下生成 `fish_yolov8` 文件夹，里面包含训练日志、权重文件、配置文件等。
```bash
runs/
└── train/
    └── fish_yolov8/
        ├── weights/
        │   ├── best.pt      # 验证集上表现最好的模型
        │   └── last.pt      # 最后一个 epoch 的模型
        ├── results.png      # 精度、召回率、loss 曲线
        ├── confusion_matrix.png
        └── opt.yaml         # 训练配置记录
```
在 Python 中加载并验证模型:
```python
# evaluate_fish.py
from ultralytics import YOLO

model = YOLO("runs/train/fish_yolov8/weights/best.pt")
metrics = model.val()   # 在验证集上评估
print(metrics)
```
输出的 metrics 是一个字典（或命名元组）
```bash
{
  'metrics/precision': 0.935,
  'metrics/recall': 0.911,
  'metrics/mAP50': 0.920,
  'metrics/mAP50-95': 0.866,
  'val/box_loss': 0.025,
  'val/cls_loss': 0.010,
  'val/dfl_loss': 0.015
}
```
#### 4.模型部署

将训练好的权重文件 `fish_yolov8.pt` 复制到板上，并修改 `data.yaml` 文件中的 `path` 字段为板上数据集路径。

在板上运行：
```bash
python detect.py --weights fish_yolov8.pt --source 0 --conf 0.25 --save-txt
```

其中：
- `--weights`：指定权重文件路径
- `--source`：指定摄像头设备号或视频文件路径