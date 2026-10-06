### ***\*动手练习\****

\1. 在管理机上改写 check-disk.sh，把 node1 和 node2 纳入巡检，并把阈值改成环境变量传入（提示：THRESHOLD=$1）。

\# hostlist.txt文件内容

192.168.2.130

192.168.2.131

192.168.2.132

 

\# check-disk.sh内容

\#!/bin/bash

\# check-disk.sh —— 批量检查各主机根分区使用率

THRESHOLD=80

 

for ip in $(cat /home/yh/hostlist.txt); do

 usage=$(ssh -o ConnectTimeout=3 yh@$ip \

  "df -P / | awk 'NR==2 {print int(\$5)}'" 2>/dev/null)

 if [ -z "$usage" ]; then

  echo "$ip  连接失败"

 elif [ "$usage" -ge "$THRESHOLD" ]; then

  echo "$ip  磁盘 ${usage}%  需要清理"

 else

  echo "$ip  磁盘 $usage%  正常"

 fi

done

 

\2. 把 rotate-nginx-logs.sh 部署到任意一台虚拟机（先装个 nginx），配置 crontab 每天凌晨 2 点执行，观察日志切割效果。

用 ab 压测工具（推荐）

\# 安装 ab

apt install -y apache2-utils

 

\# 模拟 10000 次请求，产生大量访问日志

ab -n 10000 -c 10 http://127.0.0.1/

 

\# 查看日志大小

ls -lh /var/log/nginx/access.log

\#crontab -l查看定时脚本

0 0 * * * /usr/local/nginx/sbin/scripts/log_rotate.sh

\#直接执行脚本进行日志分割

root@k8s-master:/var/log/nginx# bash /usr/local/nginx/sbin/scripts/log_rotate.sh 

root@k8s-master:/var/log/nginx# ls -lh /var/log/nginx/

access_20261006.log  access.log      error_20261006.log  error.log       rotate.log

root@k8s-master:/var/log/nginx# ls -lh /var/log/nginx/

total 1.0M

-rw-r--r-- 1 root  root 1016K Oct  6 03:00 access_20261006.log

-rw-r--r-- 1 nobody root   0 Oct  6 03:07 access.log

-rw-r--r-- 1 root  root   67 Oct  6 01:51 error_20261006.log

-rw-r--r-- 1 nobody root   0 Oct  6 03:07 error.log

-rw-r--r-- 1 root  root   41 Oct  6 03:07 rotate.log

root@k8s-master:/var/log/nginx# cat rotate.log 

[2026-10-06 03:07:14] 日志切割完成

 

 

\3. 在一台虚拟机上故意把 /etc/fstab 写错一个挂载项并重启，练习进入救援模式修复（完成后务必改回来）。

1.在故障界面按 M 键（Manual recovery），系统会进入 emergency mode，出现 root 登录提示符。

\# 出现以下提示符，输入 root 密码
Welcome to emergency mode! After logging in, type "journalctl -xb" to view logs.
Entering emergency mode. Exit the shell to continue boot.
Type "journalctl" to view system logs.
root@(none):~#

 

2.如果系统直接卡在挂载错误界面，或者按 M 没反应，可以从 GRUB 引导菜单进入救援模式：

1）重启系统，在 GRUB 启动菜单界面按 e 键编辑启动项；

2）找到以 linux 开头的那一行，在行末添加 systemd.unit=emergency.target；

3）按 Ctrl+X 启动，即可进入 emergency mode。

\# GRUB 编辑示例：
\# linux  /boot/vmlinuz-xxx root=UUID=xxx ro quiet splash systemd.unit=emergency.target
\#                                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
\#                                    新增的这一段

 

2.7 其他进入救援模式的方法

2.7.1 使用安装介质救援（最彻底）

如果紧急模式也进不去，可以用系统安装 U 盘/光盘引导，选择「Rescue installed system」进入救援环境，手动挂载根分区后修复 fstab。

2.7.2 进入单用户模式（Single User Mode）

在 GRUB 编辑界面，把 ro 改成 rw init=/bin/bash，直接以 root 身份进入单用户模式，无需密码。这种方式适合忘记 root 密码的场景。

\# GRUB 编辑：
\# linux /boot/vmlinuz-xxx root=UUID=xxx rw init=/bin/bash
\#                   ^^
\#                   原来的 ro 改成 rw，末尾加 init=/bin/bash

 

\4. 用 curl -v 完整观察一次 HTTPS 请求，找出 TCP 握手、TLS 握手、HTTP 响应分别发生在输出的哪几行。

root@k8s-master:~# curl -v baidu.com

Trying 111.63.65.247:80...

Connected to baidu.com (111.63.65.247) port 80 (#0)

GET / HTTP/1.1 Host: baidu.com User-Agent: curl/7.81.0 Accept: /

 

Mark bundle as not supporting multiuse < HTTP/1.1 301 Moved Permanently < Location: https://www.baidu.com/ < Date: Tue, 06 Oct 2026 07:07:33 GMT < Content-Length: 57 < Content-Type: text/html; charset=utf-8 <

[Moved Permanently](chrome://doubao-chat/chat/[https://www.baidu.com/](https://www.baidu.com/)).

Connection #0 to host baidu.com left intact root@k8s-master:~# curl -v https://www.baidu.com/

Trying 39.156.70.239:443...

Connected to [www.baidu.com](http://www.baidu.com/) (39.156.70.239) port 443 (#0)

ALPN, offering h2

ALPN, offering http/1.1

CAfile: /etc/ssl/certs/ca-certificates.crt

CApath: /etc/ssl/certs

TLSv1.0 (OUT), TLS header, Certificate Status (22):

TLSv1.3 (OUT), TLS handshake, Client hello (1):

TLSv1.2 (IN), TLS header, Certificate Status (22):

TLSv1.3 (IN), TLS handshake, Server hello (2):

TLSv1.2 (IN), TLS header, Certificate Status (22):

TLSv1.2 (IN), TLS handshake, Certificate (11):

TLSv1.2 (IN), TLS header, Certificate Status (22):

TLSv1.2 (IN), TLS handshake, Server key exchange (12):

TLSv1.2 (IN), TLS header, Certificate Status (22):

TLSv1.2 (IN), TLS handshake, Server finished (14):

TLSv1.2 (OUT), TLS header, Certificate Status (22):

TLSv1.2 (OUT), TLS handshake, Client key exchange (16):

TLSv1.2 (OUT), TLS header, Finished (20):

TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):

TLSv1.2 (OUT), TLS header, Certificate Status (22):

TLSv1.2 (OUT), TLS handshake, Finished (20):

TLSv1.2 (IN), TLS header, Finished (20):

TLSv1.2 (IN), TLS header, Certificate Status (22):

TLSv1.2 (IN), TLS handshake, Finished (20):

SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256

ALPN, server accepted to use http/1.1

Server certificate:

subject: C=CN; ST=Beijing; L=Beijing; O=Beijing Baidu Netcom Science Technology Co., Ltd.; CN=baidu.com

start date: Jul 9 02:32:55 2026 GMT

expire date: Jan 24 02:32:55 2027 GMT

subjectAltName: host "[www.baidu.com](http://www.baidu.com/)" matched cert's "*.baidu.com"

issuer: C=BE; O=GlobalSign nv-sa; CN=GlobalSign RSA OV SSL CA 2018

SSL certificate verify ok.

TLSv1.2 (OUT), TLS header, Supplemental data (23):

GET / HTTP/1.1 Host: [www.baidu.com](http://www.baidu.com/) User-Agent: curl/7.81.0 Accept: /

 

TLSv1.2 (IN), TLS header, Supplemental data (23):

Mark bundle as not supporting multiuse < HTTP/1.1 200 OK < Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform < Content-Length: 2443 < Content-Type: text/html < Pragma: no-cache < Server: bfe < Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/ < Date: Tue, 06 Oct 2026 07:08:55 GMT < 

TLSv1.2 (IN), TLS header, Supplemental data (23):

TLSv1.2 (IN), TLS header, Supplemental data (23):

![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml8372\wps3.png)t id=su value=百度一下 class="bg s_btn" autofocus> [新闻](chrome://doubao-chat/chat/[http://news.baidu.com](http://news.baidu.com)) [hao123](chrome://doubao-chat/chat/[https://www.hao123.com](https://www.hao123.com)) [地图](chrome://doubao-chat/chat/[http://map.baidu.com](http://map.baidu.com)) [视频](chrome://doubao-chat/chat/[http://v.baidu.com](http://v.baidu.com)) [贴吧](chrome://doubao-chat/chat/[http://tieba.baidu.com](http://tieba.baidu.com)) [更多产品](chrome://www.baidu.com/more/) 

[关于百度](chrome://doubao-chat/chat/[http://home.baidu.com](http://home.baidu.com)) [About Baidu](chrome://doubao-chat/chat/[http://ir.baidu.com](http://ir.baidu.com)) 

©2017 Baidu [使用百度前必读](chrome://doubao-chat/chat/[http://www.baidu.com/duty/](http://www.baidu.com/duty/)) [意见反馈](chrome://doubao-chat/chat/[http://jianyi.baidu.com/](http://jianyi.baidu.com/)) 京ICP证030173号 ![img](file:///C:\Users\ADMINI~1\AppData\Local\Temp\ksohtml8372\wps4.png)

· Connection #0 to host [www.baidu.com](http://www.baidu.com/) left intact 

 

逐段拆分 curl -v https://www.baidu.com/ 日志

说明：curl -v 中以 * 开头是curl内部调试信息，> 是客户端发送请求，< 是服务器返回响应

\*  Trying 39.156.70.239:443...

\* Connected to www.baidu.com (39.156.70.239) port 443 (#0)

✅ TCP三次握手完成

Trying：域名解析后开始发起TCP连接

Connected to ... port 443：TCP三次握手成功，建立TCP通道。

TCP握手：这两行。TCP是传输层，先打通443端口。

 



------



TLS握手阶段（紧接着TCP建立之后，全部 * 开头）

\* ALPN, offering h2

\* ALPN, offering http/1.1

\*  CAfile: /etc/ssl/certs/ca-certificates.crt

\*  CApath: /etc/ssl/certs

\* TLSv1.0 (OUT), TLS header, Certificate Status (22):

\* TLSv1.3 (OUT), TLS handshake, Client hello (1):

\* TLSv1.2 (IN), TLS header, Certificate Status (22):

\* TLSv1.3 (IN), TLS handshake, Server hello (2):

\* TLSv1.2 (IN), TLS header, Certificate Status (22):

\* TLSv1.2 (IN), TLS handshake, Certificate (11):

\* TLSv1.2 (IN), TLS header, Certificate Status (22):

\* TLSv1.2 (IN), TLS handshake, Server key exchange (12):

\* TLSv1.2 (IN), TLS header, Certificate Status (22):

\* TLSv1.2 (IN), TLS handshake, Server finished (14):

\* TLSv1.2 (OUT), TLS header, Certificate Status (22):

\* TLSv1.2 (OUT), TLS handshake, Client key exchange (16):

\* TLSv1.2 (OUT), TLS header, Finished (20):

\* TLSv1.2 (OUT), TLS change cipher, Change cipher spec (1):

\* TLSv1.2 (OUT), TLS header, Certificate Status (22):

\* TLSv1.2 (OUT), TLS handshake, Finished (20):

\* TLSv1.2 (IN), TLS header, Finished (20):

\* TLSv1.2 (IN), TLS header, Certificate Status (22):

\* TLSv1.2 (IN), TLS handshake, Finished (20):

\* SSL connection using TLSv1.2 / ECDHE-RSA-AES128-GCM-SHA256

\* ALPN, server accepted to use http/1.1

\* Server certificate:

\*  subject: C=CN; ST=Beijing; L=Beijing; O=Beijing Baidu Netcom Science Technology Co., Ltd.; CN=baidu.com

\*  start date: Jul  9 02:32:55 2026 GMT

\*  expire date: Jan 24 02:32:55 2027 GMT

\*  subjectAltName: host "www.baidu.com" matched cert's "*.baidu.com"

\*  issuer: C=BE; O=GlobalSign nv-sa; CN=GlobalSign RSA OV SSL CA 2018

\*  SSL certificate verify ok.

✅ TLS握手全过程

Client Hello：客户端发给服务端，支持的TLS版本、加密套件、ALPN（h2/http1.1）

Server Hello：服务端选定TLS1.2、加密套件、ALPN协议

服务端下发证书、公钥信息

客户端校验证书合法性 SSL certificate verify ok.

双方交换密钥材料，完成加密协商

SSL certificate verify ok. 代表TLS握手全部完成，TCP通道已经升级成加密TLS隧道，接下来才发HTTP报文。

 



------



HTTP 请求（客户端发送，> 开头）

\> GET / HTTP/1.1

\> Host: www.baidu.com

\> User-Agent: curl/7.81.0

\> Accept: */*

\> 

客户端在加密TLS隧道里发送HTTP GET请求。

HTTP 响应（服务器返回，< 开头 + HTML内容）

\* TLSv1.2 (IN), TLS header, Supplemental data (23):

\* Mark bundle as not supporting multiuse

< HTTP/1.1 200 OK

< Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform

< Content-Length: 2443

< Content-Type: text/html

< Pragma: no-cache

< Server: bfe

< Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/

< Date: Tue, 06 Oct 2026 07:08:55 GMT

< 

<!DOCTYPE html>

<!--STATUS OK--><html> ...百度首页HTML...

\* TLSv1.2 (IN), TLS header, Supplemental data (23):

\* Connection #0 to host www.baidu.com left intact

✅ HTTP响应：

< HTTP/1.1 200 OK：HTTP状态行

后续一堆 <：HTTP响应头

空行之后：HTTP响应体（网页HTML）

Connection #0 ... left intact：本次请求结束，TCP连接保持（keep-alive）

一句话总结分层顺序

TCP握手：* Trying ... → * Connected，建立443端口TCP连接

TLS握手：从ALPN、Client Hello开始，一直到 SSL certificate verify ok.，TCP通道加密

HTTP：加密隧道内发送 > GET 请求，接收 < HTTP/1.1 200 OK + 响应头+网页body

补充：你前面 curl baidu.com 是HTTP 80端口

\> GET / HTTP/1.1

< HTTP/1.1 301 Moved Permanently

< Location: [https://www.baidu.com/](https://www.baidu.com/)

 

 

 

***\*【划重点】\****本章小结

***\*【小贴士】\****一切皆文件：/etc、/var/log、/proc 是运维活动的四个主战场。

systemd unit 文件让服务具备开机自启与崩溃自动拉起能力，是现代服务的标准形态。

性能排查四板斧：top 看 CPU、free 看 available、iostat 看磁盘、ss 看连接。

网络排查分层定位：dig 查 DNS、ping 查网络层、curl telnet 查端口、curl -I 查应用层。

set -euo pipefail 是生产脚本的第一行，退出码与日志是可靠脚本的两大支柱。

### ***\*自测题\****

\1. 755 和 644 分别是什么权限？/etc/shadow 为什么普通用户读不了，passwd 却能改它？

755是属主有读写执行权限，组内和其他人员有读和执行权限

644是属主有读写权限，组内和其他人有读权限
因为/usr/bin/passwd设置了SUID，执行瞬间会借用文件属主（root）的身份。

 

\2. free -h 里 available 很小但 buff/cache 很大，机器真的内存告急吗？为什么？

Linux 会把空闲内存拿去做文件缓存（buff/cache），这部分随时可以归还，真正该关注的是 available 一列。同理，load 高不一定慢——先看是 CPU 密集还是 IO 等待（top 里按 1 看 us/sy/wa），wa 高说明瓶颈在磁盘而不是 CPU。

\3. 描述三次握手的过程，并解释为什么两次握手不够。

三次握手：客户端发 SYN（喂，听得到吗），服务端回 SYN+ACK（听得到，你听得到我吗），客户端再回 ACK（听得到），连接建立。之所以要三次而不是两次，是为了双方都确认「自己说话对方能听到」——两次握手时服务端无法确认这一点。四次挥手多一次，是因为断开时服务端可能还有话没说完（数据没发完），需要分别确认「我说完了」和「你说的我收到了」

\4. 你的定时脚本手工运行正常、cron 里却不工作，最可能的原因有哪些？（至少两个）

cron手工执行正常，放定时任务里失败：高频原因（按排查优先级排序，Linux通用，k8s宿主机也适用）

核心本质：cron 的执行环境 和 你交互式shell(root)环境完全不一样！ 手工执行：你登录的shell，有PATH、环境变量、当前工作目录、终端、umask； cron执行：极简环境，几乎没有环境变量，没有交互式终端，默认工作目录是 /

\1. PATH 环境变量问题【最高发】

手工shell里PATH很长，包含 /usr/local/bin、/sbin 等；

cron 默认PATH只有：PATH=/usr/bin:/bin

现象：脚本里直接写命令名（如 docker、kubectl、curl），手工能跑，cron报 command not found。

✅ 解决：

脚本内部写命令绝对路径（/usr/bin/curl）

或在crontab头部重新定义PATH

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin

\2. 当前工作目录不一样

你手工在 /root 或某个项目目录执行脚本；

cron 执行脚本时，默认工作目录是根目录 /

现象：脚本使用相对路径读取日志/配置文件，cron运行找不到文件。

✅ 解决：脚本开头第一行切换目录

cd /root || exit

\# 或者用绝对路径引用所有文件

\3. 缺少环境变量（数据库、代理、KUBECONFIG等）

交互式shell加载：~/.bashrc / ~/.profile

cron 不会加载 bashrc、profile！

典型：kubectl 脚本，手工能访问集群，cron执行找不到 KUBECONFIG；

有代理变量 HTTP_PROXY、自定义业务环境变量，cron里全部不存在。

✅ 解决：脚本开头手动加载环境变量

export KUBECONFIG=/root/.kube/config

\# 或者 source 环境文件

source /root/.bashrc

\4. 输出/重定向、终端问题

cron没有TTY终端：

有些命令依赖终端（tput、彩色输出、交互式read）会失败；

不写日志的话，报错信息会被邮件发送（很多机器默认开启cron邮件），看不到报错。

✅ 排查手段：把stdout/stderr写入日志

\* * * * * /root/test.sh >> /var/log/cron_test.log 2>&1

\5. 权限问题

crontab 分两种：

用户crontab：crontab -e（当前用户身份执行）

系统crontab：/etc/crontab，必须指定用户字段

坑：编辑 /etc/crontab 忘记写用户名，任务直接不执行。

 

脚本本身是否有可执行权限 x：chmod +x script.sh

文件/目录SELinux权限（CentOS/RHEL常见，Ubuntu基本没有）

\6. 脚本解释器、换行符问题（Windows编辑脚本）

Windows换行 \r\n，Linux是 \n 如果脚本在Windows写好传到Linux，cron运行报：/bin/bash^M: bad interpreter 手工执行有时候也报错，但偶尔有人在vim里执行正常。

查看换行：cat -v script.sh 修复：dos2unix script.sh

\7. 时间、时区问题

cron 使用系统时区，不是你shell临时设置时区； 容器/虚拟机有时候时区不一致，任务触发时间不符合预期。 查看系统时区：timedatectl

\8. 锁文件、并发冲突（容易忽略）

脚本手工执行很快跑完；cron周期太短，上一次脚本没退出，锁文件阻止第二次运行。 比如脚本内置 lockfile / flock，手工执行时没有残留锁，定时反复触发残留锁文件。

\9. cron服务本身没运行

\# Debian/Ubuntu

systemctl status cron

\# CentOS/RHEL

systemctl status crond

服务没启动，任务完全不跑。

✅ 最简排查步骤（推荐顺序）

给crontab任务加上日志输出，拿到cron真实报错（最重要！）

在脚本头部：定义PATH、cd到正确目录、导入需要的环境变量

全部命令替换成绝对路径测试

检查脚本换行符 cat -v

验证cron服务状态

快速示例修复模板 crontab

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin

\# 每5分钟执行脚本，输出日志

*/5 * * * * /root/myscript.sh >> /var/log/myscript.log 2>&1

如果你需要，我可以给你一个诊断脚本，放到crontab里自动输出cron环境（PATH、PWD、所有env），直接对比交互式shell环境差异。