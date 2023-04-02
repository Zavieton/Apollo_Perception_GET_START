# For Apollo D-Kit Advance Development
This private repo is belong to [AIUS](http://aius.hit.edu.cn/main.htm), Harbin Institute of Technology, Harbin, 2023

## 0. 写在最前面

主要参考资料
1. [Apollo](https://apollo.baidu.com)
2. [Apollo开放平台8.0版文档](https://apollo.baidu.com/community/Apollo-Homepage-Document/Apollo_Doc_CN_8_0?doc=%2F%25E4%25BD%25BF%25E7%2594%25A8%25E6%258C%2587%25E5%258D%2597%2F%25E5%25BF%25AB%25E9%2580%259F%25E4%25B8%258A%25E6%2589%258B)

[github markdown 语法](http://aius.hit.edu.cn/main.htm)



### 更新公告
1. 本机新增了VSCODE IDE
2. 新增minianaconda环境，启动请**conda activate base**，支持python语言的编写和编译
3. 配置了opencv-python，pytorch-gpu等深度学习与视觉环境
>> conda activate base // 进入环境
>> 
>> conda deactivate // 退出

4. 新增了**wechat for ubuntu**，可以在本机登录微信啦 ～^-^~



## 1. Camera模块
### 1.1 摄像头模块的调用
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

## 2. Lidar模块

