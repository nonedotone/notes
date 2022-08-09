# crontab

## 示例
```bash
# 编辑定时任务
crontab -e

# 每天两点半执行command
30 2 * * * /your/command

# 每隔3天，3点半执行command
30 3 */3 * * /your/command
```

## 格式
```bash
> MIN HOUR DOM MON DOW CMD

# MIN  ---> Minute field 0 to 59
# HOUR ---> Hour field 0 to 23
# DOM  ---> Day of Month 1-31
# MON  ---> Month field 1-12
# DOW  ---> Day Of Week 0-6
# CMD  ---> Command Any command to be executed.


 30 2 * * * /your/command
# ^ ^
# | hour
# minute

+---------------- minute (0 - 59)
|  +------------- hour (0 - 23)
|  |  +---------- day of month (1 - 31)
|  |  |  +------- month (1 - 12)
|  |  |  |  +---- day of week (0 - 6) (Sunday=0 or 7)
|  |  |  |  |
*  *  *  *  *  command to be executed
```


## 参考
* https://www.codenong.com/14710257/