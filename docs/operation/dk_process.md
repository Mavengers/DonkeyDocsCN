# 安装远程登录软件
	一般情况下,驴车安装配置完成后,可通过Wi-Fi连入网络,建议通过远程终端软件登录驴车进行调试.

常见远程登录软件

* [putty](https://www.putty.org/)
* [mobaxterm](https://mobaxterm.mobatek.net/)
* [SecureCRT](https://www.vandyke.com/products/securecrt/)
* [Xshell](https://www.netsarang.com/en/xshell/)

登录信息

* 用户名: `donkeycar`
* 密码: `donkeycar`
* IP地址: 请在比赛现场联网后通过下面命令获取.

```
hostname -I
```
或

```
ifconfig wlan0
```

<b>驴车项目路径</b>

驴车项目位于`/home/donkeycar/projects`目录, 请在远程登录系统后,通过`cd` 命令切换至该目录,并确认当前为`(donkey)`虚拟环境.
如果当前位于:`(base)[donkeycar@donkeycar0X ~]$`

> 此时是conda 基础环境需要切换至驴车环境.

```
conda activate donkey 
```

如果需要退出虚拟环境:
```
conda deactivate donkey 
```

## 启动驴车

在终端输入:
```
python manage.py  drive
```

驴车启动后,终端会被占用,如果需要终止驴车运行请在键盘按下: `ctrl + c`

## 网页端控制 

默认情况下,驴车在启动后会通过tornado 库实现一个简单的web页面,该页面可用于监控驴车行驶状态及网页端控制,可通过浏览器访问驴车IP
地址及端口来获取.

* 网页端打开

```
http://驴车当前IP地址:8887 端口
```

> 默认端口: 8887 
> 驴车在驾驶过程中,会不断通过摄像头采集图片信息并整合当前的角度和油门值存储在`data`目录.
> 在执行终端中可以通过键盘输入: CTRL + C 结束采集. 

## 压缩打包数据 

为上传到Azure 进行云端进行训练,加快训练进程.

```
cd /home/pi/projects/mycar/ 
tar -czvf data.tar.gz  data/ 
ls  
```

> 如果有 data.tar.gz 的红色压缩包就好. 

## 上传云主机 

* 通过`scp`命令拷贝

```
scp -P50001 data.tar.gz -i DONKEYCAR_KEY.pem azureuser@AZURE_SERVER_IP:/home/azureuser/mycar/
```

## 训练方法

* 解压  
```
tar -xf data.tar.gz  
```

> 请确认数据包加压到`/home/azureuser/mycar/data` 目录.

* 训练 

在终端中

```bash
donkey train --tub data/  --model models/donkeycar01.h5 
```
> donkeycar01.h5 就是模型的名字, donkeycar01 为当前驴车的主机名. 

## 转换模型 

训练完成后会在驴车实例的 models 目录中生成模型文件. 由于默认训练出来的模型类型是:`keras`
需要转换为`tensorflow`类型,再转换成`OpenVINO`能识别的类型.

> 该操作TBD

```bash
cd /home/azureuser/projects/traincar/models/ 
scp -P 50001 azureuser@AZURE_SERVER_IP://home/azureuser/projects/traincar/models/donkeycar01.h5
```

> 这里的`-P` 后面填写的服务器端口请参考对应的服务器端口填写.

* 自动驾驶 

首先要确保你的模型文件放置在驴车的/home/pi/projects/mycar/models/下. 
然后在终端执行: 

```bash
cd /home/pi/projects/mycar/ 
python manage.py drive --model models/donkeycar01.h5 
```
* 网页控制

通过浏览器访问`http://驴车当前IP地址:8887/`, 替换驴车地址信息为驴车设别地址信息.

先点击页面下方的: `start vehicle`按钮然后在 `mode & pilot` 选择`local pilot`小车就开始自行驾驶了. 

需要终止请在终端上按下 `Ctrl + C`

> 以上所有操作需要在拥有硬件驴车和 azure 云服务器的情况下进行.

---
