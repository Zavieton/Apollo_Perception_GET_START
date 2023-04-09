# 数据记录与可视化


## Cyber_record

### 0. 首先通过以下命令安装“cyber_record”

>pip3 install cyber_record

>// or update version

>pip3 install cyber_record -U

### 1. 可以通过以下命令获取记录文件中的信息

1. cyber_record info

> cyber_record info -f example.record.00000

```
record_file: example.record.00000
version:     1.0
begin_time:  2021-07-23 17:12:15.114944
end_time:    2021-07-23 17:12:15.253911
duration:    0.14 s
size:        477.55 KByte
message_number: 34
channel_number: 8

/apollo/planning                      , apollo.planning.ADCTrajectory         , 1
/apollo/routing_request               , apollo.routing.RoutingRequest         , 0
/apollo/monitor                       , apollo.common.monitor.MonitorMessage  , 0
/apollo/routing_response              , apollo.routing.RoutingResponse        , 0
/apollo/routing_response_history      , apollo.routing.RoutingResponse        , 1
/apollo/localization/pose             , apollo.localization.LocalizationEstimate, 15
/apollo/canbus/chassis                , apollo.canbus.Chassis                 , 15
/apollo/prediction                    , apollo.prediction.PredictionObstacles , 2
```

2. cyber_record echo

>  cyber_record echo -f example.record.00000 -t /apollo/canbus/chassis

```
engine_started: true
speed_mps: 0.0
throttle_percentage: 0.0
brake_percentage: 0.0
driving_mode: COMPLETE_AUTO_DRIVE
gear_location: GEAR_DRIVE
header {
   timestamp_sec: 1627031535.112813
   module_name: "SimControl"
   sequence_num: 76636
}

```

### 2. 从记录文件中读取和写入消息Demo

#### 2.1 读取数据
```python
from cyber_record.record import Record

file_name = "20210521122747.record.00000"
record = Record(file_name)
for topic, message, t in record.read_messages():
   print("{}, {}, {}".format(topic, type(message), t))
```

#### 2.2 过滤数据
```python
def read_filter_by_both():
   record = Record(file_name)
   for topic, message, t in record.read_messages('/apollo/canbus/chassis', \
         start_time=1627031535164278940, end_time=1627031535215164773):
      print("{}, {}, {}".format(topic, type(message), t))
```

#### 2.3 解析消息

> pip3 install record_msg -U

**csv format**
```python
f = open("message.csv", 'w')
writer = csv.writer(f)

def parse_pose(pose):
   '''
   save pose to csv file
   '''
   line = to_csv([pose.header.timestamp_sec, pose.pose])
   writer.writerow(line)

f.close()
```

**image**

```python
image_parser = ImageParser(output_path='../test')
for topic, message, t in record.read_messages():
   if topic == "/apollo/sensor/camera/front_6mm/image":
      image_parser.parse(message)
      # or use timestamp as image file name
      # image_parser.parse(image, t)
```

**lidar**

```python
pointcloud_parser = PointCloudParser('../test')
for topic, message, t in record.read_messages():
   if topic == "/apollo/sensor/lidar32/compensator/PointCloud2":
      pointcloud_parser.parse(message)
      # other modes, default is 'ascii'
      # pointcloud_parser.parse(message, mode='binary')
      # pointcloud_parser.parse(message, mode='binary_compressed')
```
#### 2.4 保存数据
**image**
```python
def write_image():
   image_builder = ImageBuilder()
   write_file_name = "example_w.record.00002"
   with Record(write_file_name, mode='w') as record:
      img_path = 'test.jpg'
      pb_image = image_builder.build(img_path, encoding='rgb8')
      record.write('/apollo/sensor/camera/front_6mm/image',
                  pb_image,
                  int(time.time() * 1e9))
```

**lidar**
```python
def write_point_cloud():
   point_cloud_builder = PointCloudBuilder()
   write_file_name = "example_w.record.00003"
   with Record(write_file_name, mode='w') as record:
      pcd_path = 'test.pcd'
      pb_point_cloud = point_cloud_builder.build(pcd_path)
      record.write('/apollo/sensor/lidar32/compensator/PointCloud2',
                  pb_point_cloud,
                  int(time.time() * 1e9))
```


## Camera 数据
参考 [Camera](https://github.com/Zavieton/Apollo_Perception_GET_START/edit/main/Camera.md)

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


## Lidar 数据
### 录制record包

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

启动记录

> 在Module Controller 启动所需要的Lidar模块和Recode模块
>
> 操控平台运动，此时已经开始记录数据
> 
> 关闭Recode模块，终止记录，记录的数据会保存至apollo/data下，格式类似于example.record.00000

### 数据的解析和生成点云文件

比如带解析文件为  */home/apollo/apollo/data/bag/2023-04-07-15-45-03/20230407154503.record.00000*

运行脚本

```python
from cyber_record.record import Record
from record_msg.parser import PointCloudParser

file_name = "/home/apollo/apollo/data/bag/2023-04-07-15-45-03/20230407154503.record.00000"
record = Record(file_name)
# for topic, message, t in record.read_messages():
    # print("{}, {}, {}".format(topic, message, t))
    # break
    # print(topic)
pointcloud_parser = PointCloudParser('./store/lidar')
for topic, message, t in record.read_messages():
    if topic == "/apollo/sensor/lidar16/PointCloud2":
        pointcloud_parser.parse(message)
        
    # other modes, default is 'ascii'
    # pointcloud_parser.parse(message, mode='binary')
    # pointcloud_parser.parse(message, mode='binary_compressed')


```

![a44cc24cf57d9725555a415be7d7baa](https://user-images.githubusercontent.com/46212574/230770897-be8209be-b280-4d57-bef6-5aa2bf1f2907.png)

### 点云可视化

安装open3d库

>pip install open3d

运行脚本

```python

from pathlib import Path

import numpy as np
import open3d as o3d


class PtVis:
    def __init__(self, name='空格进入下一帧 Z退回上一帧', width=1024, height=768, pcd_files_path=""):
        self.pcd_files_path = pcd_files_path
        self.pcd_files_list = []
        self.index = 0
        self.pcd = None
        self.vis = None
        self.name = name
        self.width = width
        self.height = height
        self.axis_pcd = o3d.geometry.TriangleMesh().create_coordinate_frame()
        self.init_pcd_files_list()

    def init_setting(self):
        opt = self.vis.get_render_option()
        # 设置背景颜色和点大小
        opt.background_color = np.asarray([0, 0, 0])
        opt.point_size = 2

    def show_pcd(self):
        # 绘制open3d坐标系
        self.vis = o3d.visualization.VisualizerWithKeyCallback()
        self.vis.create_window(window_name=f"{Path(self.pcd_files_list[0]).name} {self.name}", width=self.width,
                               height=self.height)
        # 修改显示
        self.init_setting()
        # 初始化显示pcd第一帧
        self.pcd = o3d.io.read_point_cloud(self.pcd_files_list[0])
        self.vis.add_geometry(self.pcd)
        self.vis.add_geometry(self.axis_pcd)
        # 设置键盘响应事件
        self.vis.register_key_callback(90, lambda temp: self.last())
        self.vis.register_key_callback(32, lambda temp: self.next())
        self.vis.run()

    def next(self):
        if 0 <= self.index < len(self.pcd_files_list) - 1:
            self.index += 1
        self.update(self.index)

    def last(self):
        if 0 < self.index <= len(self.pcd_files_list) - 1:
            self.index -= 1
        self.update(self.index)

    def init_pcd_files_list(self):
        for file in Path(self.pcd_files_path).rglob("*.pcd"):
            self.pcd_files_list.append(str(file))
        self.pcd_files_list = sorted(self.pcd_files_list, key=lambda p: Path(p).stem)

    def update(self, index):
        new_pcd_file = self.pcd_files_list[index]
        self.vis.create_window(window_name=f"{Path(new_pcd_file).name} {self.name}", width=self.width,
                               height=self.height)
        self.pcd = o3d.io.read_point_cloud(new_pcd_file)
        self.vis.clear_geometries()  # 清空vis点云
        self.vis.add_geometry(self.pcd)  # 增加vis中的点云
        self.vis.add_geometry(self.axis_pcd)
        self.vis.poll_events()
        self.vis.update_renderer()

    def close(self):
        self.vis.destroy_window()


if __name__ == '__main__':
    pt = PtVis(pcd_files_path='store/lidar')
    print(f"共有PCD个数:{len(pt.pcd_files_list)}")
    pt.show_pcd()
    pt.close()

```

![df577091e37d24ac35033d9b9a2050e](https://user-images.githubusercontent.com/46212574/230770893-ba341e24-daf2-45eb-bb04-8ebd4605fb43.jpg)

### 解析出原始点云数据
```python
import numpy as np
import math
import open3d as o3d
# bin
def read_bin(self, pcd_file):
    ## struct read binary
    import struct
    list = []
    with open(pcd_file, 'rb') as f:
        data = f.read()
        pc_iter = struct.iter_unpack('ffff', data)
        for i, point in enumerate(pc_iter):
            list.append(point)
    
    return np.array(list)

# ASCII
def read_pcd(self, pcd_file):
    data = []
    with open(pcd_file, 'r') as f:
        lines = f.readlines()[11:]
        for line in lines:
            line = list(line.strip('\n').split(' '))
            # line = np.array(line.strip('\n').split(' '), dtype=np.float32)
            x = float(line[0])
            y = float(line[1])
            z = float(line[2])
            i = float(line[3])
            r = math.sqrt(x**2 + y**2 + z**2) * i
            data.append(np.array([x,y,z,i,r]))
         # points = list(map(lambda line: list(map(lambda x: float(x), line.split(' '))), lines))
        points = np.array(data)

    return points


def o3d_read_pcd( pcd_file):
    ## open3d read pcd
    pcd = o3d.io.read_point_cloud(pcd_file)
    points = np.asarray(pcd.points) # x, y, z
    return points

if __name__ == '__main__':
    path = 'store/lidar/00000.pcd'
    p = o3d_read_pcd(path)
    print(p)

```


## 其他传感器数据

```python
from cyber_record.record import Record
from record_msg.parser import PointCloudParser

file_name = "/home/apollo/apollo/data/bag/2023-04-07-15-45-03/20230407154503.record.00000"
record = Record(file_name)
# for topic, message, t in record.read_messages():
    # print("{}, {}, {}".format(topic, message, t))
    # break
    # print(topic)
pointcloud_parser = PointCloudParser('./store/lidar')
for topic, message, t in record.read_messages():
    if topic == // 传感器Module //:
        print(message)

```

