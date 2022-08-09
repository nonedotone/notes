# ssh-keygen

## 创建一个ssh-key
```bash
ssh-keygen -t rsa -C "your_email@example.com"
```

* -t 指定密钥类型，默认是rsa，可以省略。
* -C 设置注释文字，比如邮箱。
* -f 指定密钥文件存储文件名。

## 生成ssh公钥
```bash
ssh-keygen -y -f ~/.ssh/id_rsa > ~/.ssh/id_rsa.pub
```

检查生成的公钥的详细信息
```bash
ssh-keygen -l -f ~/.ssh/id_rsa

4096 d6:7b:c7:7a:4f:3c:4d:29:54:62:5f:2c:58:b2:cb:86 ~/.ssh/id_rsa (RSA)
```