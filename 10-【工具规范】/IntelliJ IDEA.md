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

# 自定义文档注释

每次写文档注释很麻烦，之前用eclipse的自定义文档注释很方面，后来就想把idea的文档注释也改一下。注释主要分为文件注释，类注释，方法注释。

## 文件注释

```
/**
* 一些声明信息
* Description: <br/>
* date: ${DATE} ${TIME}<br/>
* @author ${USER}<br/>
* @version
* @since JDK 1.8
*/
```

![image-20200522162634776](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200522162637-683066.png)

## 类注释

```
/**
* ClassName: ${NAME} <br/>
* Description: <br/>
* date: ${DATE} ${TIME}<br/>
* @author ${USER}<br/>
*/
```

![image-20200522162742097](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200522162743-969522.png) 

## 定义快捷键注释

![image-20200522162809036](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200522162809-451232.png) 

这个相对实用性很强，先选择2增加模板群组 我起名叫 MyTemplateGroup

然后选择创建好的模板群组创建两个新的模板

![image-20200522162855475](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200522162856-803404.png) 

1. 快捷输入
2. 模板描述
3. 模板内变量映射
4. 快捷输入触发方式
5. 模板内容

### /*c 为Class通用

```
/**
 * @description:
 * @author: $author$
 * @date: $date$
 */
```

![image-20200522162950418](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200522162951-605465.png) 

![image-20200522163003590](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200522163006-568627.png) 

### /*m为Method通用

```
/**
 * @description: 
 * @author: $author$
 * @date: $date$
 * $params$
 * @return: $return$
 */
```

![image-20200522163032823](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200522163033-637650.png) 

params中为特殊处理，请将以下代码复制到空格内按Enter确认

```
groovyScript("def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList(); for(i = 0; i < params.size(); i++) {result+=(i==0 ? '@param ' : ' * @param ') + params[i] + ': ' + ((i < params.size() - 1) ? '\\n' : '')}; return result", methodParameters())
```

