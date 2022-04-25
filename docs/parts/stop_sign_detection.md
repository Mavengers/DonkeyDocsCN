
# Stop Sign检测 

通过摄像头识别停止标识信息,如果检测到有停止标识, 驴车将```pilot/throttle``` 将为0.
并且会关联一个识别框到```cam/image_array```.

---------------

## 设备需求
要使用这个 part,你需要准备 Intel NCS2, 二代神经棒:

- [Intel NCS2 ](https://)

##  如何使用

在文档```myconfig.py```后添加:

```
STOP_SIGN_DETECTOR = True
```

