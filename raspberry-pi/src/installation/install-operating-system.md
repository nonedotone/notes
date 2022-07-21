# Install Operating System

## Downloads

[all system](https://downloads.raspberrypi.org/) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; [images](https://www.raspberrypi.com/software/operating-systems/)

## Install

### 磁盘信息

```bash
df -h
```

### 卸载分区(卸载磁盘)
```bash
diskutil unmount /dev/disk2s1
```

### 设别列表

```bash
diskutil list
```

### 安装

* 设备字母disk3前得加个字母r 
* 用sudo命令，否则会提示无权限

> if=path/to/image of=原始字符设备

```bash
sudo dd bs=4m if=2018-11-13-raspbian-xxxx.img of=/dev/rdisk2
```

### 卸载设备
```bash
diskutil unmountDisk /dev/disk2
```


## Other

### 允许SSH

根目录下创建一个 SSH 的文件(没有后缀名)
```bash
touch ssh
```

### 连接wifi网络
根目录下新建 wpa_supplicant.conf 文件，文件内容如下

```bash
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    # Wi-Fi名称
    ssid="Wi-Fi名称"
    # Wi-Fi密码
    psk="Wi-Fi密码"
    # 优先级，数字越大优先级越高，不可以是负数
    priority=1
    # 是否隐藏wifi，1表示隐藏，不是就去掉下面一行
    scan_ssid=1
}
```

创建写入文件
```bash
cat > wpa_supplicant.conf << EOF
country=CN
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="wifi-name"
    psk="password"
    priority=99
    scan_ssid=1
}
EOF
```

### 初始用户
boot分区中创建一个名为userconf.txt的文件

其中包含字符串username:encrypted-password

秘密密码
```bash
echo '12345678' | openssl passwd -6 -stdin

# nonedotone:12345678
nonedotone:$6$Ic9WD3Cp6lvRCGcK$hlbQqm5OEeB6wSA4ZSNSfI40lzKVtHHTm6g0RpbXaO9L7aITPG127nGOxicMYdDYdALl4zE91PweULh5koGoa.
```

创建写入文件
```bash
cat > userconf.txt << EOF
nonedotone:$6$Ic9WD3Cp6lvRCGcK$hlbQqm5OEeB6wSA4ZSNSfI40lzKVtHHTm6g0RpbXaO9L7aITPG127nGOxicMYdDYdALl4zE91PweULh5koGoa.
EOF
```

### 设置语言，更新
```bash
sudo localedef -i en_US -f UTF-8 en_US.UTF-8

sudo apt-get update
```

### 安装工具
```bash
sudo apt-get install mosh -y
sudo apt-get install git -y
sudo apt-get install vim -y
sudo apt-get install zsh -y

sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```