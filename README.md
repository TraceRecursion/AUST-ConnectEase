# AUST-ConnectEase

安理易连是一个开源项目，旨在为安徽理工大学的学生和教职工提供一个简单、快捷的校园网自动登录解决方案。通过使用本脚本，用户可以轻松地连接到校园网络，无需手动输入登录信息，从而节省时间，提高学习和工作效率。

此项目将教你在Windows电脑和OpenWrt软路由上实现自动校园网登录。

安理校园网的登录方式随WiFi和宽带的区别而有所不同。

此为宽带

```sh
curl -d "callback=dr1003&DDDDD=学号@登录入口&upass=密码&0MKKey=123456" http://10.255.0.19/drcom/login
```

此为WiFi

```cmd
curl "http://10.255.0.19/drcom/login?callback=dr1003&DDDDD=学号@登录入口&upass=密码&0MKKey=123456"
```

入口和对应参数

| 登录入口 | 对应参数 |
| -------- | :------: |
| 教职工   |   @jzg   |
| 电信     |  @aust   |
| 联通     | @unicom  |
| 移动     |  @cmcc   |

## 在Windows上自动登录

依据连接的网络来选择脚本，填充完整后保存为.bat文件。单击.bat文件即可执行一次登录操作，此时你实现了一键手动操作。

定时自动登录可以通过schtasks命令创建定时任务来实现，在cmd控制台中输入以下命令：

```cmd
schtasks /create /tn "MyTask" /tr C:\\abc\\script.bat /sc ONSTART
```

其中，MyTask是定时任务的名字，C:\\abc\\script.bat是bat脚本文件的路径，ONSTART表示在系统启动时执行该任务。可以根据实际情况修改这些参数。

## 在OpenWrt上自动登录

此处以宽带为例，登录OpenWrt，进入系统->启动项->本地启动脚本页，你会看到：

```shell
此处为 /etc/rc.local 的内容。启动脚本插入到“exit 0”之前即可随系统启动运行。

# Put your custom commands here that should be executed once
# the system init finished. By default this file does nothing.

exit 0
```

将脚本写入此处即可。

我经过多次尝试搓出来一个绝妙的脚本，而且恰巧这里有足够的空间可以写的下：

```shell
# Put your custom commands here that should be executed once
# the system init finished. By default this file does nothing.

{
  count=0
  while true; do
    ping -c 1 www.jd.com
    if [ $? -eq 0 ]
    then
      count=0
    else
      count=$((count+1))
      if [ $count -eq 10 ]
      then
        curl -d "callback=dr1003&DDDDD=学号@登录入口&upass=密码&0MKKey=123456" http://10.255.0.19/drcom/login
        count=0
      fi
    fi
    sleep 3
  done
} &

exit 0
```

- **循环检测**：脚本使用一个无限循环来定期检查网络连接。
- **网络检测**：通过`ping`命令检查是否能够访问`www.jd.com`。
- **登录尝试**：如果`ping`失败超过10次，脚本会执行`curl`命令来发送登录请求。
- **睡眠间隔**：每次检测之间有3秒的间隔。

通过条件判断和循环控制来实现自动化的网络登录功能。脚本会通过ping京东的结果来决定要不要登录校园网，避免了断网后没有及时自动登录和反复自动登录被运营商查水表。（对不住了东哥，ping你家服务器比ping百度快一倍时间）
