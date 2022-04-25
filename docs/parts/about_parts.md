# 什么是 Part
> A part is a Python class that wraps a functional component of a vehicle.
一个 part 就是一个 python 的类, 可以添加一个功能到我们的驴车上.
这些包括:

* Sensors - 摄像头, 雷达, 速度计, GPS 
* Actuators - 电机控制类 
* Pilots - 道路检测, 克隆模型等等功能.
* Controllers - 基于网页或者蓝牙.
* Stores - Tub, 或者存储数据的方法

下面是一个示例显示如何使用 PiCamera part 去在每个驾驶的循环中发布一个图片到"cam/img" 通道.

```python
V = dk.Vehicle()

# 创建并初始化camera part
cam = PiCamera()

#添加摄像头到vehicle.
V.add(cam, outputs=['cam/img'])

V.start()
```

## 剖析一个 part 
所有部件(part) 共享同一个共同的结构,因此他们都可以由车辆的实例驱动在整个程序运行状态中不断运行和更新数据.
所有的部件都需要有一个"run" 或者" run_threaded" 功能,它有时候也需要``inputs=['单引号'],'逗号分割']``
这种输入的格式,也会在调用的时候生成类似``outputs=['关键字', '逗号分割']``的输出数据.

如果车的部件会抓取部分硬件资源信息, 类似摄像头和串口等设备,	在驴车停止的时候,那么它应该包含一个``shutdown`` 的方法来释放资源.

 下面举个例子:  一个 part 接受一个数字,然后与一个随机数字完成乘法运算并返回结果.

```python
import random

class RandPercent:
    def run(self, x):
        return x * random.random()
```

把这个类添加到 donkeycar 的 vehicle:

```python
V = dk.Vehicle()

# initialize the channel value
V.mem['const'] = 4

# add the part to read and write to the same channel.
V.add(RandPercent, inputs=['const'], outputs=['const'])

V.start(max_loops=5)
```

## 部件线程化
如果想要保证驴车运行的性能则需要保证 drive loop 必须保证在 10-30
次/秒,(检测数据来自: ```DRIVE_LOOP_HZ```, 这个配置来自你的配置文件(myconfig.py),默认值是 ```20hz```)
因此, 一些慢速的部件应该线程化(threaded)从而避免延长每个 drive loop 的时间.

一个线程化的部件需要去定义方法去运行在独立的线程上, 并且方法在被调用的时候能够非常快的返回当前最新的数据.

当你添加一个部件到车辆, 就是`V.add`之前, 创建的部件如果添加了```theaded = True ``` , 那么驴车主程序会调用部件的
```run_threaded``` 方法去替代```run```方法,目前你可以参考下面的例子来实现.


```V.add(RandPercent, inputs=['const'], outputs=['const'], threaded=True)```

一旦你有一个"run_threaded" 方法, donkey 就会自动去寻找一个"update"方法并且在它自己的线程中运行它, 

下面的例子是如何将`RandPercent` 部件线程化,如果这个部件需要运行一段时间才能完成的话,意思就是如果你的部件耗时长,就线程化,这样不会影响整个大循环.

```python
import random
import time

class RandPercent:
    self.in = 0.0
    self.out = 0.0
    def run(self, x):
        return x * random.random()
        time.sleep(1)

    def update(self):
        # the function run in its own thread
        while True:
            self.out = self.run(self.in)

    def run_threaded(self, x):
        self.in = x
        return self.out

```

* `part.run` : 方法是运行这个部件.
* `part.run_threaded` : 如果部件线程化,那么驴车主循环会以线程方式运行这个部件.
* `part.update` : 线程更新的方法
* `part.shutdown` 关闭进程
