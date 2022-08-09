# supervisord

## 安装
```bash
sudo apt-get install supervisor vim -y
sudo systemctl enable supervisor # 开机自启动
sudo systemctl restart supervisor
sudo systemctl status supervisor # 查看supervisord服务状态
```

/etc/supervisor/ 下的目录结构如下：
```bash
.
├── conf.d
│   ├── frpc.conf
│   ├── minio.conf
│   └── syncthing.conf
└── supervisord.conf
```

frpc.conf 为方向代理客户端服务 minio.conf 为文件服务 syncthing.conf 为去中心的文件同步

## 配置
frpc.conf 举例说明如何写需要看护的进程
```bash
[program:frpc]
command=/home/pi/frp/frpc -c /home/pi/frp/frpc.ini
stderr_logfile=/var/log/supervisor/frpc.log
stdout_logfile=/var/log/supervisor/frpc.log
#directory=/home/pi/frp
autostart=true  
user=root
autorestart=true
startsecs=20
```

sudo supervisorctl reload 即可启动frpc服务

## 其他命令
```bash
supervisorctl status #查看进程状态
supervisorctl stop app_name #停止进程
supervisorctl start app_name #启动进程
supervisorctl restart app_name #重启进程
supervisorctl reload #重启所有进程
supervisorctl update #更新那些改过配置的进程
```