# 定时任务 crontab 

#### 编译与安装（挂载NFS时不好用）

- 在`busybox`中添加`crontab`的支持，并编译复制即可
- `mkdir -p /var/spool/cron/crontabs`
- `crontab -e`进入编辑
- `crond`启动进程，可以通过ps查看
- `crontab -l`查看当前任务

#### 使用方法

```
* * * * * command

minute 00-59
hour 00-23
day-of-month 01-31
month-of-year 01-12
day-of-week 0-6(0 = Sunday)

1：整点整时
*/2：每2个
7-23/3：7点到23点，每3小时
```

