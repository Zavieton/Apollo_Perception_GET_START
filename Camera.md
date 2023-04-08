
## 1. 基于Camera的自定义目标检测算法部署
### 1.1 摄像头模块的直接调用
摄像头模块调用采用opencv-python读取端口的方式进行，其中Apollo搭载两个单目相机，端口号分别为0/2，可以采用如下的方式调用端口并读取数据

```python
import cv2
import numpy as np

cap = cv2.VideoCapture(0)
while(1):
    # get a frame
    ret, frame = cap.read()
    # show a frame
    cv2.imshow("capture", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
cv2.destroyAllWindows() 
```

**要注意的是**，apollo存在端口占用的问题。
apollo cyber_launch (命令行或dreamviewer开启摄像头后), 本地自定义代码将无法读取到端口信息，需要关闭对应的端口
可以进入 cyber_monitor 查看调用情况


### 1.2 基于camera的自动驾驶环境下的目标检测
我们的工作主要参考了[YOLOv5](https://github.com/ultralytics/yolov5/tree/v5.0)实时目标检测算法，并在此基础上进行优化 
相关的代码部署在 *apollo/perception/yolov5* 目录

检测模型的训练基于[KITTI](https://www.cvlibs.net/datasets/kitti/)数据集进行，考虑到该开源数据集历史相对悠久（~10year），且采集自国外场景。当前环境下的一些视觉特征（车辆款式、行人穿搭等）与数据集略有出入，因此检测可能产生误检 ~-.-~

读取摄像头并进行实时检测 & 展示

> cd \\project_dir\\
> 
> conda activate base
> 
> sh detect.sh
> 
> *or* python detect.py --source 0 --weights xxx

检测效果将大致如下
![Image text](https://github.com/Zavieton/Apollo_Perception_GET_START/blob/main/1.png)


### 1.3 基于Apollo recoder的数据记录和离线调用
打开Dreamviewer界面
> cd apollo
> 
> ./apollo.sh # 进入docker环境
> 
> bash ./scripts/bootstrap.sh
> 
> 如果需要关闭Dreamviewer 
> 
> bash./scripts/bootstrap.sh stop
>
> 在浏览器打开 http://localhost:8888/
> 
> 选择 --vehicle-- 和 --map--

启动camera记录
> 在Module Controller 启动所需要的Camera模块和Recode模块
>
> 操控平台运动，此时已经开始记录数据
> 
> 关闭Recode模块，终止记录，记录的数据会保存至apollo/data下，格式类似于example.record.00000
> 

基于上述生成的记录文件，可以从中提取出图像信息，并用于进一步的离线分析
> conda activate base

> pip3 install cyber_record record_msg

这里采用了第三方解析模块，[项目地址](https://cyber-record.readthedocs.io/en/latest/#%E2%80%8Bcyber-record.readthedocs.io/en/latest/#), 该模块在下面具体介绍

保存图片
```python
from record_msg.parser import ImageParser

image_parser = ImageParser(output_path='../test')
for topic, message, t in record.read_messages():
  if topic == "/apollo/sensor/camera/front_6mm/image":
    image_parser.parse(image)
    # or use timestamp as image file name
    # image_parser.parse(image, t) 
```
Recode记录的图片将会以逐帧形式保存至output_path，便于进一步分析

