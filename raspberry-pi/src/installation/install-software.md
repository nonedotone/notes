# Install Software

## Update Languate
```bash
sudo localedef -i en_US -f UTF-8 en_US.UTF-8
```

## Install Oh My Zsh
```bash
sudo apt-get install git zsh

sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
#或
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

## Install Docker
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc

curl -fsSL https://get.docker.com -o get-docker.sh

sudo sh get-docker.sh

sudo usermod -aG docker $USER
```

## Install Docker-Compose(废弃⚠️)
⚠️请使用```docker compsoe```，安装docker时已安装
```bash
sudo apt-get install -y python python-pip

sudo apt-get install python3.5 -y

sudo rm /usr/bin/python

sudo ln -s /usr/bin/python3.5 /usr/bin/python

sudo apt-get install python3-pip -y

sudo apt-get install libffi-dev -y

sudo pip3 install docker-compose -i https://pypi.mirrors.ustc.edu.cn/simple/  --trusted-host  pypi.mirrors.ustc.edu.cn
```