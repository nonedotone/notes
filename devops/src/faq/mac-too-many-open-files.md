# macOS问题too many open files

too many open files 大致有以下三种可能:
1. 操作系统打开的文件句柄数过多（内核的限制）
2. launchd 对进程进行了限制
3. shell 对进程进行了限制


## 内核的限制
整个操作系统可以打开的文件数受内核参数影响，可以通过以下命令查看
```bash
$ sysctl kern.maxfiles
$ sysctl kern.maxfilesperproc
```
在我电脑上输出如下
```bash
kern.maxfiles: 49152
kern.maxfilesperproc: 42576
```
好像已经很大了。

如果需要临时修改的话，运行如下命令
```bash
$ sudo sysctl -w kern.maxfiles=20480 # 或其他你选择的数字
$ sudo sysctl -w kern.maxfilesperproc=18000 # 或其他你选择的数字
```
永久修改，需要在 /etc/sysctl.conf 里加上类似的下述内容
```bash
kern.maxfiles=20480
kern.maxfilesperproc=18000
```
这个文件可能需要自行创建


## launchd 对进程的限制
获取当前的限制：
```bash
$ launchctl limit maxfiles
```
输出类似这样：
```bash
maxfiles    256            unlimited
```
其中前一个是软限制，后一个是硬件限制。

临时修改：
```bash
$ sudo launchctl limit maxfiles 65536 200000
```
系统范围内修改则需要在文件夹 /Library/LaunchDaemons 下创建一个 plist 文件 limit.maxfiles.plist：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>limit.maxfiles</string>
    <key>ProgramArguments</key>
    <array>
      <string>launchctl</string>
      <string>limit</string>
      <string>maxfiles</string>
      <string>65536</string>
      <string>200000</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>ServiceIPC</key>
    <false/>
  </dict>
</plist>
```
修改文件权限
```bash
$ sudo chown root:wheel /Library/LaunchDaemons/limit.maxfiles.plist
$ sudo chmod 644 /Library/LaunchDaemons/limit.maxfiles.plist
```
载入新设定
```bash
$ sudo launchctl load -w /Library/LaunchDaemons/limit.maxfiles.plist
```


## shell 的限制
通过下述命令查看现有限制
```bash
$ ulimit -a
```
得到如下输出
```bash
...
-n: file descriptors                256
...
```

通过 
```bash
ulimit -S -n 4096 
```
如果需要保持修改，可以将这一句命令加入你的 .bash_profile 或 .zshrc 等。



## 参考

* https://blog.abreto.net/archives/2020/02/macos-too-many-open-files.html