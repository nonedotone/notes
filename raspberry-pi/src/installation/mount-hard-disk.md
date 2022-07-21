# Mount Hard Disk

## 硬盘盘符

```bash
sudo fdisk -l
```

## 取消挂载
```bash
sudo umount /dev/sda
```

## 格式化硬盘
```bash
sudo mkfs -t ext4  /dev/sda

# ext3为文件系统格式，还有其他格式可选：
# mkfs -t ext4  /dev/sda
# mkfs -t ext3  /dev/sda
# mkfs -t ext2  /dev/sda
# mkfs -t reiserfs  /dev/sda
# mkfs -t fat32   /dev/sda
# mkfs -t msdos   /dev/sda
```

## 创建挂载地址、设置权限、挂载
```bash
sudo mkdir /data
sudo chmod 777 /data
sudo mount /dev/sda /data
```

## 查询设备UUID
```bash
sudo blkid
```

## 设置开机自动挂载
```bash
sudo vim /etc/fstab

# 添加下面内容
UUID=7b5cd23a-e25c-4553-9965-2016048e339e /data ext4 defaults 0 0
```