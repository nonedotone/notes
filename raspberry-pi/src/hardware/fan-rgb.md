# 风扇RGB

[zip文件](./temp_control.zip)

## 前置条件

打开I2C设置
PS：树莓派智能贴身管家与树莓派的控制方式是通过I2C来操作的，所以我们先使能树莓派的I2C服务。
```bash
sudo raspi-config
# 选择
Interfacing Options
# 选择
P5 I2C
```

## 安装wiringPi 
> 一般树莓派官方raspbian系统默认会自带wiringPi，可以运行gpio –v查看版本
```bash
git clone git://git.drogon.net/wiringPi 

#如果此步骤下载不了，请使用以下命令下载非官方的wiringPi镜像
git clone https://github.com/WiringPi/WiringPi.git
cd WiringPi
sudo ./build
```

## 安装gcc
> 树莓派官方raspbian带桌面软件版本系统有自带，可以运行gcc –v查看版本
```bash
sudo apt-get install gcc
```


### 风扇RGB

```bash
gcc -o temp_control temp_control.c ssd1306_i2c.c -lwiringPi
```