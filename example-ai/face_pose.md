---
title: '人脸姿态检测实验'
sidebar_position: 3
---

# 人脸姿态检测实验

## 前言

在上一章节中，我们已经学习了如何在CanMV下使用CanMV AI视觉开发框架和MicroPython编程方法实现人脸关键点检测的功能，本章将通过人脸姿态检测实验，介绍如何使用CanMV AI视觉开发框架和MicroPython编程实现人脸姿态检测并将人脸朝向绘制出来。在本实验中，我们首先采集摄像头捕获的图像，然后经过图像预处理、模型推理和输出处理结果等一系列步骤，完成人脸检测的功能，然后在检测到人脸的区域，进一步使用人脸姿态检测的模型进行推理，从而将人脸区域的人脸朝向解析出来。最后，将检测结果绘制并显示在显示器上。通过本章的学习，读者将掌握如何在CanMV下使用CanMV AI视觉开发框架和MicroPython编程方法实现人脸姿态检测功能。

## AI开发框架介绍

有关AI开发框架的介绍，请见[CanMV AI开发框架](development_framework.md)

## 硬件设计

### 例程功能

1. 获取摄像头输出的图像，然后将图像输入到CanMV K230D的AI模型进行推理，本实验使用了两个AI模型，一个是前面章节使用到的人脸检测模型，另一个是用于识别人脸姿态解析模型。人脸检测模型负责找出图像中的人脸区域，然后将该区域传递给人脸姿态解析模型进行推理，人脸姿态解析模型能够对检测到的每一个人脸做人脸姿态估计，然后用欧拉角（roll/yaw/pitch）表示人脸的姿态，roll表示人脸左右摇头的程度，yaw表示人脸左右旋转的程度，pitch表示人脸低头抬头的程度。我们根据人脸的姿态角构建可视化的矩形投影，最后将图像显示在LCD上。

### 硬件资源

1. 本章实验内容主要讲解K230D的神经网络加速器KPU的使用，无需关注硬件资源。


### 原理图

1. 本章实验内容主要讲解K230D的神经网络加速器KPU的使用，无需关注原理图。

## 实验代码

``` python
from libs.PipeLine import PipeLine, ScopedTiming
from libs.AIBase import AIBase
from libs.AI2D import Ai2d
import os
import ujson
from media.media import *
from media.sensor import *
from time import *
import nncase_runtime as nn
import ulab.numpy as np
import time
import image
import aidemo
import random
import gc
import sys

# 自定义人脸检测任务类
class FaceDetApp(AIBase):
    def __init__(self,kmodel_path,model_input_size,anchors,confidence_threshold=0.25,nms_threshold=0.3,rgb888p_size=[1280,720],display_size=[1920,1080],debug_mode=0):
        super().__init__(kmodel_path,model_input_size,rgb888p_size,debug_mode)
        # kmodel路径
        self.kmodel_path=kmodel_path
        # 检测模型输入分辨率
        self.model_input_size=model_input_size
        # 置信度阈值
        self.confidence_threshold=confidence_threshold
        # nms阈值
        self.nms_threshold=nms_threshold
        self.anchors=anchors
        # sensor给到AI的图像分辨率，宽16字节对齐
        self.rgb888p_size=[ALIGN_UP(rgb888p_size[0],16),rgb888p_size[1]]
        # 视频输出VO分辨率，宽16字节对齐
        self.display_size=[ALIGN_UP(display_size[0],16),display_size[1]]
        # debug模式
        self.debug_mode=debug_mode
        # 实例化Ai2d，用于实现模型预处理
        self.ai2d=Ai2d(debug_mode)
        # 设置Ai2d的输入输出格式和类型
        self.ai2d.set_ai2d_dtype(nn.ai2d_format.NCHW_FMT,nn.ai2d_format.NCHW_FMT,np.uint8, np.uint8)

    # 配置预处理操作，这里使用了pad和resize，Ai2d支持crop/shift/pad/resize/affine，具体代码请打开/sdcard/app/libs/AI2D.py查看
    def config_preprocess(self,input_image_size=None):
        with ScopedTiming("set preprocess config",self.debug_mode > 0):
            # 初始化ai2d预处理配置，默认为sensor给到AI的尺寸，可以通过设置input_image_size自行修改输入尺寸
            ai2d_input_size=input_image_size if input_image_size else self.rgb888p_size
            # 计算padding参数，并设置padding预处理
            self.ai2d.pad(self.get_pad_param(), 0, [104,117,123])
            # 设置resize预处理
            self.ai2d.resize(nn.interp_method.tf_bilinear, nn.interp_mode.half_pixel)
            # 构建预处理流程,参数为预处理输入tensor的shape和预处理输出的tensor的shape
            self.ai2d.build([1,3,ai2d_input_size[1],ai2d_input_size[0]],[1,3,self.model_input_size[1],self.model_input_size[0]])

    # 自定义后处理，results是模型输出的array列表，这里使用了aidemo库的face_det_post_process接口
    def postprocess(self,results):
        with ScopedTiming("postprocess",self.debug_mode > 0):
            res = aidemo.face_det_post_process(self.confidence_threshold,self.nms_threshold,self.model_input_size[0],self.anchors,self.rgb888p_size,results)
            if len(res)==0:
                return res
            else:
                return res[0]

    # 计算padding参数
    def get_pad_param(self):
        dst_w = self.model_input_size[0]
        dst_h = self.model_input_size[1]
        # 计算最小的缩放比例，等比例缩放
        ratio_w = dst_w / self.rgb888p_size[0]
        ratio_h = dst_h / self.rgb888p_size[1]
        if ratio_w < ratio_h:
            ratio = ratio_w
        else:
            ratio = ratio_h
        new_w = (int)(ratio * self.rgb888p_size[0])
        new_h = (int)(ratio * self.rgb888p_size[1])
        dw = (dst_w - new_w) / 2
        dh = (dst_h - new_h) / 2
        top = (int)(round(0))
        bottom = (int)(round(dh * 2 + 0.1))
        left = (int)(round(0))
        right = (int)(round(dw * 2 - 0.1))
        return [0,0,0,0,top, bottom, left, right]

# 自定义人脸姿态任务类
class FacePoseApp(AIBase):
    def __init__(self,kmodel_path,model_input_size,rgb888p_size=[1920,1080],display_size=[1920,1080],debug_mode=0):
        super().__init__(kmodel_path,model_input_size,rgb888p_size,debug_mode)
        # kmodel路径
        self.kmodel_path=kmodel_path
        # 人脸姿态模型输入分辨率
        self.model_input_size=model_input_size
        # sensor给到AI的图像分辨率，宽16字节对齐
        self.rgb888p_size=[ALIGN_UP(rgb888p_size[0],16),rgb888p_size[1]]
        # 视频输出VO分辨率，宽16字节对齐
        self.display_size=[ALIGN_UP(display_size[0],16),display_size[1]]
        # debug模式
        self.debug_mode=debug_mode
        # 实例化Ai2d，用于实现模型预处理
        self.ai2d=Ai2d(debug_mode)
        # 设置Ai2d的输入输出格式和类型
        self.ai2d.set_ai2d_dtype(nn.ai2d_format.NCHW_FMT,nn.ai2d_format.NCHW_FMT,np.uint8, np.uint8)

    # 配置预处理操作，这里使用了affine，Ai2d支持crop/shift/pad/resize/affine，具体代码请打开/sdcard/app/libs/AI2D.py查看
    def config_preprocess(self,det,input_image_size=None):
        with ScopedTiming("set preprocess config",self.debug_mode > 0):
            # 初始化ai2d预处理配置，默认为sensor给到AI的尺寸，可以通过设置input_image_size自行修改输入尺寸
            ai2d_input_size=input_image_size if input_image_size else self.rgb888p_size
            # 计算affine矩阵并设置affine预处理
            matrix_dst = self.get_affine_matrix(det)
            self.ai2d.affine(nn.interp_method.cv2_bilinear,0, 0, 127, 1,matrix_dst)
            # 构建预处理流程,参数为预处理输入tensor的shape和预处理输出的tensor的shape
            self.ai2d.build([1,3,ai2d_input_size[1],ai2d_input_size[0]],[1,3,self.model_input_size[1],self.model_input_size[0]])

    # 自定义后处理，results是模型输出的array列表，计算欧拉角
    def postprocess(self,results):
        with ScopedTiming("postprocess",self.debug_mode > 0):
            R,eular = self.get_euler(results[0][0])
            return R,eular

    def get_affine_matrix(self,bbox):
        # 获取仿射矩阵，用于将边界框映射到模型输入空间
        with ScopedTiming("get_affine_matrix", self.debug_mode > 1):
            # 设置缩放因子
            factor = 2.7
            # 从边界框提取坐标和尺寸
            x1, y1, w, h = map(lambda x: int(round(x, 0)), bbox[:4])
            # 模型输入大小
            edge_size = self.model_input_size[1]
            # 平移距离，使得模型输入空间的中心对准原点
            trans_distance = edge_size / 2.0
            # 计算边界框中心点的坐标
            center_x = x1 + w / 2.0
            center_y = y1 + h / 2.0
            # 计算最大边长
            maximum_edge = factor * (h if h > w else w)
            # 计算缩放比例
            scale = edge_size * 2.0 / maximum_edge
            # 计算平移参数
            cx = trans_distance - scale * center_x
            cy = trans_distance - scale * center_y
            # 创建仿射矩阵
            affine_matrix = [scale, 0, cx, 0, scale, cy]
            return affine_matrix

    def rotation_matrix_to_euler_angles(self,R):
        # 将旋转矩阵（3x3 矩阵）转换为欧拉角（pitch、yaw、roll）
        # 计算 sin(yaw)
        sy = np.sqrt(R[0, 0] ** 2 + R[1, 0] ** 2)
        if sy < 1e-6:
            # 若 sin(yaw) 过小，说明 pitch 接近 ±90 度
            pitch = np.arctan2(-R[1, 2], R[1, 1]) * 180 / np.pi
            yaw = np.arctan2(-R[2, 0], sy) * 180 / np.pi
            roll = 0
        else:
            # 计算 pitch、yaw、roll 的角度
            pitch = np.arctan2(R[2, 1], R[2, 2]) * 180 / np.pi
            yaw = np.arctan2(-R[2, 0], sy) * 180 / np.pi
            roll = np.arctan2(R[1, 0], R[0, 0]) * 180 / np.pi
        return [pitch,yaw,roll]

    def get_euler(self,data):
        # 获取旋转矩阵和欧拉角
        R = data[:3, :3].copy()
        eular = self.rotation_matrix_to_euler_angles(R)
        return R,eular

# 人脸姿态任务类
class FacePose:
    def __init__(self,face_det_kmodel,face_pose_kmodel,det_input_size,pose_input_size,anchors,confidence_threshold=0.25,nms_threshold=0.3,rgb888p_size=[1280,720],display_size=[1920,1080],debug_mode=0):
        # 人脸检测模型路径
        self.face_det_kmodel=face_det_kmodel
        # 人脸姿态模型路径
        self.face_pose_kmodel=face_pose_kmodel
        # 人脸检测模型输入分辨率
        self.det_input_size=det_input_size
        # 人脸姿态模型输入分辨率
        self.pose_input_size=pose_input_size
        # anchors
        self.anchors=anchors
        # 置信度阈值
        self.confidence_threshold=confidence_threshold
        # nms阈值
        self.nms_threshold=nms_threshold
        # sensor给到AI的图像分辨率，宽16字节对齐
        self.rgb888p_size=[ALIGN_UP(rgb888p_size[0],16),rgb888p_size[1]]
        # 视频输出VO分辨率，宽16字节对齐
        self.display_size=[ALIGN_UP(display_size[0],16),display_size[1]]
        # debug_mode模式
        self.debug_mode=debug_mode
        self.face_det=FaceDetApp(self.face_det_kmodel,model_input_size=self.det_input_size,anchors=self.anchors,confidence_threshold=self.confidence_threshold,nms_threshold=self.nms_threshold,rgb888p_size=self.rgb888p_size,display_size=self.display_size,debug_mode=0)
        self.face_pose=FacePoseApp(self.face_pose_kmodel,model_input_size=self.pose_input_size,rgb888p_size=self.rgb888p_size,display_size=self.display_size)
        self.face_det.config_preprocess()

    # run函数
    def run(self,input_np):
        # 人脸检测
        det_boxes=self.face_det.run(input_np)
        pose_res=[]
        for det_box in det_boxes:
            # 对检测到的每一个人脸做人脸姿态估计
            self.face_pose.config_preprocess(det_box)
            R,eular=self.face_pose.run(input_np)
            pose_res.append((R,eular))
        return det_boxes,pose_res


    # 绘制人脸姿态角效果
    def draw_result(self,pl,dets,pose_res):
        pl.osd_img.clear()
        if dets:
            draw_img_np = np.zeros((self.display_size[1],self.display_size[0],4),dtype=np.uint8)
            draw_img=image.Image(self.display_size[0], self.display_size[1], image.ARGB8888,alloc=image.ALLOC_REF,data=draw_img_np)
            line_color = np.array([255, 0, 0 ,255],dtype=np.uint8)    #bgra
            for i,det in enumerate(dets):
                # （1）获取人脸姿态矩阵和欧拉角
                projections,center_point = self.build_projection_matrix(det)
                R,euler = pose_res[i]
                # （2）遍历人脸投影矩阵的关键点，进行投影，并将结果画在图像上
                first_points = []
                second_points = []
                for pp in range(8):
                    sum_x, sum_y = 0.0, 0.0
                    for cc in range(3):
                        sum_x += projections[pp][cc] * R[cc][0]
                        sum_y += projections[pp][cc] * (-R[cc][1])
                    center_x,center_y = center_point[0],center_point[1]
                    x = (sum_x + center_x) / self.rgb888p_size[0] * self.display_size[0]
                    y = (sum_y + center_y) / self.rgb888p_size[1] * self.display_size[1]
                    x = max(0, min(x, self.display_size[0]))
                    y = max(0, min(y, self.display_size[1]))
                    if pp < 4:
                        first_points.append((x, y))
                    else:
                        second_points.append((x, y))
                first_points = np.array(first_points,dtype=np.float)
                aidemo.polylines(draw_img_np,first_points,True,line_color,2,8,0)
                second_points = np.array(second_points,dtype=np.float)
                aidemo.polylines(draw_img_np,second_points,True,line_color,2,8,0)
                for ll in range(4):
                    x0, y0 = int(first_points[ll][0]),int(first_points[ll][1])
                    x1, y1 = int(second_points[ll][0]),int(second_points[ll][1])
                    draw_img.draw_line(x0, y0, x1, y1, color = (255, 0, 0 ,255), thickness = 2)
            pl.osd_img.copy_from(draw_img)

    def build_projection_matrix(self,det):
        x1, y1, w, h = map(lambda x: int(round(x, 0)), det[:4])
        # 计算边界框中心坐标
        center_x = x1 + w / 2.0
        center_y = y1 + h / 2.0
        # 定义后部（rear）和前部（front）的尺寸和深度
        rear_width = 0.5 * w
        rear_height = 0.5 * h
        rear_depth = 0
        factor = np.sqrt(2.0)
        front_width = factor * rear_width
        front_height = factor * rear_height
        front_depth = factor * rear_width  # 使用宽度来计算深度，也可以使用高度，取决于需求
        # 定义立方体的顶点坐标
        temp = [
            [-rear_width, -rear_height, rear_depth],
            [-rear_width, rear_height, rear_depth],
            [rear_width, rear_height, rear_depth],
            [rear_width, -rear_height, rear_depth],
            [-front_width, -front_height, front_depth],
            [-front_width, front_height, front_depth],
            [front_width, front_height, front_depth],
            [front_width, -front_height, front_depth]
        ]
        projections = np.array(temp)
        # 返回投影矩阵和中心坐标
        return projections, (center_x, center_y)


if __name__=="__main__":
    # 显示模式，默认"lcd"
    display_mode="lcd"
    display_size=[640,480]
    # 人脸检测模型路径
    face_det_kmodel_path="/sdcard/examples/kmodel/face_detection_320.kmodel"
    # 人脸姿态模型路径
    face_pose_kmodel_path="/sdcard/examples/kmodel/face_pose.kmodel"
    # 其它参数
    anchors_path="/sdcard/examples/utils/prior_data_320.bin"
    rgb888p_size=[1280,960]
    face_det_input_size=[320,320]
    face_pose_input_size=[120,120]
    confidence_threshold=0.5
    nms_threshold=0.2
    anchor_len=4200
    det_dim=4
    anchors = np.fromfile(anchors_path, dtype=np.float)
    anchors = anchors.reshape((anchor_len,det_dim))

    # 初始化PipeLine，只关注传给AI的图像分辨率，显示的分辨率
    sensor = Sensor(width=1280, height=960) # 构建摄像头对象
    pl = PipeLine(rgb888p_size=rgb888p_size, display_size=display_size, display_mode=display_mode)
    pl.create(sensor=sensor)  # 创建PipeLine实例
    fp=FacePose(face_det_kmodel_path,face_pose_kmodel_path,det_input_size=face_det_input_size,pose_input_size=face_pose_input_size,anchors=anchors,confidence_threshold=confidence_threshold,nms_threshold=nms_threshold,rgb888p_size=rgb888p_size,display_size=display_size)
    try:
        while True:
            os.exitpoint()
            with ScopedTiming("total",1):
                img=pl.get_frame()                      # 获取当前帧
                det_boxes,pose_res=fp.run(img)          # 推理当前帧
#                print(det_boxes,pose_res)               # 打印结果
                fp.draw_result(pl,det_boxes,pose_res)   # 绘制推理效果
                pl.show_image()                         # 展示推理效果
                gc.collect()
    except Exception as e:
        sys.print_exception(e)
    finally:
        fp.face_det.deinit()
        fp.face_pose.deinit()
        pl.destroy()
```

可以看到一开始是先定义显示模式、图像大小、模型相关的一些变量。

接着是通过初始化PipeLine，这里主要初始化sensor和display模块，配置摄像头输出两路不同的格式和大小的图像，以及设置显示模式，完成创建PipeLine实例。

然后调用自定义FacePose类构建人脸姿态任务类，FacePose类会通过调用FaceDetApp类和FacePoseApp类完成对AIBase接口的初始化以及使用Ai2D接口的方法定义人脸检测模型和人脸姿态解析模型输入图像的预处理方法。

最后在一个循环中不断地获取摄像头输出的RGBP888格式的图像帧，然后依次将图像输入到人脸检测模型、人脸姿态检测模型进行推理，然后将推理结果通过print打印，同时通过结果信息将人脸的关键点绘制到图像上，并在LCD上显示图像。

## 运行验证

实验原图如下所示：

![01](./img/06.png)

将K230D BOX开发板连接CanMV IDE，点击CanMV IDE上的“开始(运行脚本)”按钮后，将摄像头对准人脸，让其采集到人脸图像，随后便能在LCD上看到摄像头输出的图像，同时图像中的人脸姿态通过构建投影矩阵的方式实现人脸朝向的可视化展示，如下图所示：  

![01](./img/07.png)