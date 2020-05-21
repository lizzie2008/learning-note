# 查看字节码工具

- 打开Settings->Tools->External Tools -> +

- 选择JDK的javap.exe命令
  输入 -verbose ,选择`$FileClass$` `$OutputPath$`变量，这两个变量可从右边选取。
  ![image-20200422091656685](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200422091656-337834.png) 

- 右键选取执行命令，输出字节码文件

  ![image-20200422091736021](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200422091739-543154.png) 

# 修改Terminal为cmder

# cmder下载

一款Windows环境下非常简洁美观易用的cmd替代者，它支持了大部分的Linux命令。支持ssh连接linux，使用起来非常方便。比起cmd、powershell、conEmu，其界面美观简洁，功能强大。

[官网下载地址](http://cmder.net/)

cmder原生没有 ll 命令，但可以通过设置别名来实现：
 打开cmder安装目录下的\config\user-aliases.cmd文件，添加以下别名设置：

```bash
l=ls --show-control-chars -F --color $*
la=ls -aF --show-control-chars -F --color $*
ll=ls -alF --show-control-chars -F --color $*
```

# 修改终端 Terminal 为 Cmder

- 增加系统变量
   变量名为：CMDER_ROOT
   变量值为：C:\Program Files (x86)\cmder (主机上 Cmder安装主目录)

- 打开IDEA，进入设置（快捷键：Ctrl + Alt + S），进入 Tools字段，再进入 Terminal字段，在Shell path输入
   `"cmd.exe" /k ""%CMDER_ROOT%\vendor\init.bat""`

- 重启IDEA，就可以看到配置成功了。 

  ![image-20200521085631927](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200521085632-120213.png)