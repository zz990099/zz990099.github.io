---
layout:     post
title:      "(一)`FoundationPose`推理原理及c++实现--intro"
subtitle:   "基于3D模型的6D位姿检测通用模型——`FoundationPose`整体框架介绍"
date:       2025-03-27 20:00:00
author:     "zz990099"
header-img: "img/zz990099/FoundationPose/demo.png"
catalog: true
tags:
---

# Introduction

写这篇博客记录分享我学习FoundationPose算法的过程，主要是算法的部署推理优化。

FoundationPose Python工程：[NVlabs/FoundationPose](https://github.com/NVlabs/FoundationPose)

Nvidia开源的isaac_pose_esitimation工程，部署了FoundationPose的整体Pipeline， 链接：[isaac_pose_esitimation](https://github.com/NVIDIA-ISAAC-ROS/isaac_ros_pose_estimation)

在学习过程中，整理了一版不依赖于ros、nvidia_nitros等复杂库的实现，仅依赖于cuda、tensorrt等，可以快速实现算法迁移，链接：[foundationpose_cpp](https://github.com/zz990099/foundationpose_cpp)

# FoundationPose算法整体框架

算法的整体推理过程主要由两个部分组成：**图形渲染**和**网络推理**。

## 1. FoundationPose网络推理部分

首先讨论网络推理部分，共有两个网络：`Refine`网络和`Score`网络。

- `Refine网络`目的是根据实际观察到的目标物图像，以及渲染得到的目标物图像，推测两者之间的位姿偏差，示意图如图1。

  | <img src="/img/zz990099/FoundationPose/stage1.jpg" alt="1" width="800"> |
  |:----------------------------------------:|
  | **图1. Refine网络示意图**  |

  - 这里`crop_rgb`、`crop_xyz`、`render_rgb`、`render_xyz`是前处理后得到的模型输入。

  - `Refine`网络推理得到的是 `观测 -> 渲染` 的位姿偏差，包括位移偏差`transition`和姿态偏差`rotation`。

- `Score网络`目的是预测 `观测 -> 渲染` 的一致性得分，其整体框架示意图如图2。 

  | <img src="/img/zz990099/FoundationPose/stage2.jpg" alt="1" width="800"> |
  |:----------------------------------------:|
  | **图1. Score网络示意图**  |

  - 结合两个网络的推理框架可以看到，两者的渲染过程是一致的，输入一批假设位姿hyp-poses，渲染过程根据实际目标物的mesh模型，渲染得到render_rgb和render_xyz，而crop_rgb和crop_xyz是根据假设位姿从输入图像中裁剪得到的。

## 2. FoundationPose图形渲染部分

将渲染部分看作一个整体，其输入是一批hyp-poses和原始输入图像，输出是：`crop_rgb`、`crop_xyz`、`render_rgb`、`render_xyz`，这里的xyz即与像素点对应的伪点云数据。分为三个内容：
  - 图形渲染的工作流程。
  - Refine部分生成hyp-poses的逻辑 
  - Score部分生成hyp-poses的逻辑 

# 结语
这篇文档记录了FoundationPose算法的大致流程和框架，部署FoundationPose的大部分工作集中的**图形渲染**部分，下篇文章讲解。 