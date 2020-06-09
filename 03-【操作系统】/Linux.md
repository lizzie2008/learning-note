# Shell脚本

## 换行执行错误

window下的换行是回车符+换行符，也就是\r\n,而unix下是换行符\n。ubuntu下不识别\r为回车符，所以导致每行的配置都多了个\r，因此是脚本编码的问题。

- 安装dos2unix

```bash
[root@localhost ~]# yum install -y dos2unix
```

- 将shell文件编码转unix编码

```bash
[root@localhost ~]# dos2unix your_shellname.sh
```

# FTP服务器

## 安装与配置

- 安装 vsftpd

```bash
[root@localhost ~]# yum install -y vsftpd
```

以下提示说明安装成功：
![image-20200114174846721](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200114174847-874758.png) 

- 相关配置文件


> `/etc/vsftpd/vsftpd.conf` //主配置文件，核心配置文件
>
> `/etc/vsftpd/ftpusers` //黑名单，这个里面的用户不允许访问FTP服务器
>
> `/etc/vsftpd/user_list` //白名单，允许访问FTP服务器的用户列表

- 相关命令，可以看到服务已经正常启动，开始监听21端口

> `systemctl enable vsftpd` //设置开机自启动
>
> `systemctl restart vsftpd` //重新启动ftp服务
>
> `netstat -antup | grep ftp` //查看ftp服务端口

![image-20200114175515709](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200114175521-123105.png) 

- 首先，我们看当前系统都有哪些用户，发现已经有个ftp用户，便用这个用户进行登录，不用创建新的用户。

```bash
[root@lancelot-server vsftpd]# cut -d : -f 1 /etc/passwd
```

![image-20200114183959183](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200114184000-636238.png) 

- 当然如果你需要创建新用户访问也可以

```bash
[root@localhost ~]# useradd -d /data/wwwroot -m ftpuser -s /sbin/nologin
[root@localhost ~]# cd /data/wwwroot/
[root@localhost ~]# chmod -R 777 *
[root@localhost ~]# passwd ftpuser
```

- 进入相关目录，可以看到 vsftpd.conf 配置文件，我们主要修改这个文件

```bash
[root@localhost ~]# cd /etc/vsftpd
[root@localhost ~]# vi vsftpd.conf
```

![image-20200114175011728](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200114175018-730422.png)  

- 修改不允许匿名用户登录，并支持本地用户访问

```properties
anonymous_enable=NO
local_enable=YES
```

## 问题处理

无权访问：vsftpd 530 login incorrect 的几种情况

1. 密码错误。

2. 检查/etc/vsftpd/vsftpd.conf配置

   ```properties
   local_enable=YES  
   pam_service_name=vsftpd   //这里重要，有人说ubuntu是pam_service_name=ftp，可以试试
   userlist_enable=YES 
   ```

3. 检查/etc/pam.d/vsftpd，注释掉

   ```prop
   #auth required pam_shells.so
   ```

## 主动模式

FTP的两种传输模式：**主动（FTP Port）模式**和**被动（FTP Passive）模式**。

在主动模式的 FTP 中，客户端从一个随机的非系统端口(N > 1023)连接到 FTP 服务器的命令端口端口 21。然后，客户端开始监听端口 N+1，并将 FTP 命令端口 N+1 告诉 FTP 服务器，“请把数据发送给我的 N+1 端口”。然后，服务器将从本地数据端口 (端口20) 连接回客户端的数据端口，也就是 N+1 端口。

因为服务器防火墙的隔离作用，我们应该确保服务器 FTP 到客户端的一下几个通道的畅通:

FTP 服务器端口 21 （接受全部客户端）

FTP 服务器端口 21 到 > 1023 的端口 ( 服务器响应客户端控制端口 )

FTP 服务器端口 20 到 > 1023 的端口 ( 服务器发起到客户端的数据端口的连接 )

从 > 1023的端口到 FTP 服务器端口 20 ( 客户端发送 ack 到服务器的数据端口 )

用图来表示这些通道：

![image-20200116175926924](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200116175928-654709.png) 

第 1 步，客户端的命令端口与服务器的命令端口连接并发送命令端口 1027。然后，服务器在第 2 步时将一个 ACK 发送回客户端的命令端口。第 3 步，服务器在其本地数据端口上启动连接，连接到前面指定的客户端的数据端口。最后，客户端返回 ACK，如第 4 步所示。

主动模式的 FTP 主要问题实际上落在客户端。FTP 的客户端并不会主动连接到服务器的数据端口，而是是告诉服务器它正在监听哪个端口，然后服务器发起连接到客户端上指定的端口。但是，这样的连接有时候会被客户端的防火墙阻止。‘

## 被动模式

为了解决服务器主动发起到客户端连接会被阻止的问题，另一种更完善的工作模式出现了，它就是 FTP 的被动模式，缩写作 PASV，它工作的前提是客户端明确告知 FTP 服务器它使用被动模式。

在被动模式的 FTP 中，客户端启动到服务器的两个连接，解决了防火墙阻止从服务器到客户端的传入数据端口连接的问题。FTP 连接建立后，客户端在本地打开两个随机的非系统端口 N 和 N + 1(N > 1023)。第一个端口连接服务器上的 21 端口，但是客户端这次将会发出 PASV 命令，也就是不允许服务器连接回其数据端口。这样，服务器随后会打开一个随机的非系统端口 P (P > 1023)，并将 P 发送给客户端作为 PASV 命令的响应。然后客户端启动从端口 N+1 到端口 P 的连接来传输数据。

在被动模式中，要保持一下通道的畅通：

FTP服务器的 21 端口（接受所有客户端）

FTP服务器的 21端口到 > 1023 的远程端口 ( 服务器响应客户端控制端口 )

FTP服务器 > 1023 的端口（接受所有客户端发起的连接到服务器指定的随机端口）

FTP服务器 > 1023 的端口到 > 1023 的远程端口（服务器发送 ack 和数据到客户端数据端口）

被动模式用图表示：

![image-20200116180017573](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200116180018-538469.png)  

第 1 步，客户端在命令端口上与服务器连接，并发出 PASV 命令。然后，服务器在第 2 步时使用端口 2024 进行响应，告诉客户端它正在监听的数据连接端口。第 3 步，客户端启动从其数据端口到指定服务器数据端口的数据连接。最后，服务器在第 4 步将 ACK 发送回客户端的数据端口。

服务器防火墙需要给 FTP 的被动模式开放一个端口范围允许所有客户端连接，比如 5000 - 6000。

# CPU负载过高分析

如何定位是哪个服务进程导致CPU过载，哪个线程导致CPU过载，哪段代码导致CPU过载？

## 1.找到最耗CPU的进程

工具：top

方法：

- 执行top -c ，显示进程运行信息列表
- 键入P (大写p)，进程按照CPU使用率排序

图示：

![image-20200509153600739](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509153605-468072.png) 

如上图，最耗CPU的进程PID为10765

## 2.找到最耗CPU的线程

工具：top

方法：

- top -Hp 10765 ，显示一个进程的线程运行信息列表
- 键入P (大写p)，线程按照CPU使用率排序

图示：

![image-20200509153701011](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509153702-423300.png) 

如上图，进程10765内，最耗CPU的线程PID为10804

## 3.将线程PID转化为16进制

工具：printf

方法：`printf “%x” 10804`

10804对应的16进制是0x2a34，当然，这一步可以用计算器。

之所以要转化为16进制，是因为堆栈里，线程id是用16进制表示的。

## 4.查看堆栈，找到线程在干嘛

工具：pstack/jstack/grep

方法：jstack 10765 | grep ‘0x2a34’ -C5 --color

- 打印进程堆栈
- 通过线程id，过滤得到线程堆栈

图示：

![image-20200509153825624](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200509153826-790218.png) 

如上图，找到了耗CPU高的线程对应的线程名称“AsyncLogger-1”，以及看到了该线程正在执行代码的堆栈。

# nohup命令

nohup 命令运行由 Command参数和任何相关的 Arg参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 & （ 表示“and”的符号）到命令的尾部。

nohup 是 no hang up 的缩写，就是不挂断的意思。

nohup命令：如果你正在运行一个进程，而且你觉得在退出帐户时该进程还不会结束，那么可以使用nohup命令。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。

在缺省情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中。

## 案例

```bash
nohup command > myout.file 2>&1 & 
```

在上面的例子中，0 – stdin (standard input)，1 – stdout (standard output)，2 – stderr (standard error) ；

2>&1是将标准错误（2）重定向到标准输出（&1），标准输出（&1）再被重定向输入到myout.file文件中。

```bash
0 22 * * * /usr/bin/python /home/pu/download_pdf/download_dfcf_pdf_to_oss.py > /home/pu/download_pdf/download_dfcf_pdf_to_oss.log 2>&1
```

这是放在crontab中的定时任务，晚上22点时候怕这个任务，启动这个python的脚本，并把日志写在download_dfcf_pdf_to_oss.log文件中

## nohup和&的区别

& ： 指在后台运行

nohup ： 不挂断的运行，注意并没有后台运行的功能，，就是指，用nohup运行命令可以使命令永久的执行下去，和用户终端没有关系，例如我们断开SSH连接都不会影响他的运行，注意了nohup没有后台运行的意思；&才是后台运行

&是指在后台运行，但当用户推出(挂起)的时候，命令自动也跟着退出

那么，我们可以巧妙的吧他们结合起来用就是
nohup COMMAND &
这样就能使命令永久的在后台执行

例如：

```bash
sh test.sh & 
```

将sh test.sh任务放到后台 ，即使关闭xshell退出当前session依然继续运行，但**标准输出和标准错误信息会丢失（缺少的日志的输出）**

```bash
nohup sh test.sh 
```


将sh test.sh任务放到后台，关闭标准输入，**终端不再能够接收任何输入（标准输入）**，重定向标准输出和标准错误到当前目录下的nohup.out文件，即使关闭xshell退出当前session依然继续运行。

```bash
nohup sh test.sh  & 
```

将sh test.sh任务放到后台，但是依然可以使用标准输入，**终端能够接收任何输入**，重定向标准输出和标准错误到当前目录下的nohup.out文件，即使关闭xshell退出当前session依然继续运行。