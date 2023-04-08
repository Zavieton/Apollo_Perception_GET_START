

## 2. Lidar传感器的实时感知
在自动驾驶技术中，感知模块负责获取自动驾驶车辆周围环境信息，是自动驾驶车辆的“眼睛”，下游模块通过感知得到的环境信息来进行下一步决策。

常用的感知传感器包括激光雷达、摄像头、毫米波雷达等，因为激光雷达传感器具备准确的障碍物定位能力等优点，Apollo 目前采取以其为主的自动驾驶感知方案。

激光雷达感知模块接受来自激光雷达驱动的点云信息，利用这些点云信息进行障碍物的检测以及跟踪，得到的结果会被输出到感知融合模块进行下一步处理。

激光雷达感知模块接收到点云数据之后，通过高精度地图 ROI（The Region of Interest）过滤器过滤 ROI 之外的点云，去除背景对象，例如：路边建筑物、树木等，过滤后的点云数据通过障碍物检测深度学习模型进行 3D 障碍物的检测和分类，然后对得到的障碍物进行跟踪，最终得到障碍物的形状、位置、类别、速度等信息。



### 2.1 进入DreamViewer

进入 Apollo Docker 环境。

>./apollo.sh
>
在 Docker 环境里启动 Dreamview。

>bash scripts/bootstrap.sh
>
如果需要关闭 Dreamview，请您执行以下命令：

>./scripts/bootstrap.sh stop
>
在浏览器中输入网址 http://localhost:8888，打开 Dreamview,选择模式、车型和地图信息


### 2.2 开启Lidar模块并采用PointPillars模型进行三维点云检测
把车开到空旷场地，便于接收GPS信号（比如主楼后花园、电机楼门口等）

开启GPS、Transform、Location、Perception、Lidar模块

执行指令

> cyber_monitor

查看各个模块是否已经正常启动

![2021-07-09 19-06-23屏幕截图_2ca9de4](https://user-images.githubusercontent.com/46212574/230709676-ddcee15a-38ec-44ac-a444-cc4d8edaee9a.png)

操作无人车运动，此时Dreamviewer应该可以观察到感知结果，类似这样（不过应该没有车道线）
![Screenshot from 2021-07-14 12-42-23_6c87dd5](https://user-images.githubusercontent.com/46212574/230709898-9605a8ab-40d9-46ed-aeb0-f8109295b990.png)


### 2.3 算法的改进
#### 2.3.1 
Apollo内置算法基于Pointpillars进行了改进 
![image](https://user-images.githubusercontent.com/46212574/230710166-f6bc17ed-e415-4bad-8083-5662c0fb25b5.png)

使用MMDetection3D框架进行训练，在KITTI验证集上结果如下表所示,PointPillars的模型指标来自于Mmdetction3d官方。将PointPillars和模型在KITTI数据集上的检测结果进行了可视化，如下图所示。从图中可以看出模型具有更好的检出效果。可以看到，模型可以召回被截断和阻挡的车辆：
![image](https://user-images.githubusercontent.com/46212574/230710184-43f1d9f8-c651-4e78-a61f-68443f7862b7.png)
![image](https://user-images.githubusercontent.com/46212574/230710186-2bb28075-c13b-4af3-a7e5-b29dc31a4bb1.png)

#### 2.3.2
如果后续对新模型进行部署
