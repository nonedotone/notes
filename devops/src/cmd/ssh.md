# ssh

## 命令
```bash
ssh user@host
```

指定密钥
```bash
ssh -i path/to/key user@host
```


通过代理
```bash
ssh -o ProxyCommand='nc -X 5 -x 127.0.0.1:1080 %h %p' user@host
```

代理端口
```bash
ssh -L 0.0.0.0:27757:destination:port -o ProxyCommand="nc -X 5 -x 127.0.0.1:1084 %h %p" -i ~/.ssh/ssh.key user@host
```

远程执行命令
```bash
ssh -o StrictHostKeyChecking=no user@ip "ls -a"
```

## 文件
创建`~/.ssh/config`文件

配置别名
```bash
Host example
    HostName example.com
    User john
    Port 2322
```

配置密钥
```bash
Host test
    HostName 192.168.1.10
    User test
    Port 7654
    IdentityFile ~/.ssh/test.key
```

更多配置
```bash
Host test
    HostName 192.168.1.10
    User test
    Port 7654
    IdentityFile ~/.ssh/test.key
    LogLevel INFO
    Compression yes
```

配置代理
```bash
Host *.dev.com
    User test
    ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p
```

端口转发
```bash
Host testnetL
    HostName 34.226.12.232
    User ubuntu
    IdentityFile ~/.ssh/fx-chain/testnet/ssh.key
    User ubuntu
    LocalForward 26657 172.20.64.25:26657
```