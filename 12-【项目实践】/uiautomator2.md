# uiautomator2+逍遥模拟器

1. 将逍遥模拟器设置网络桥接

   ![image-20210318105927819](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20210318105928-131051.png) 

2. 安装uiautomator2

   ```
   pip install --pre uiautomator2
   
   pip install pillow
   ```

3. 初始化

   ```
   python -m uiautomator2 init
   ```

   安装完成，设备上会多一个ATX的应用。

   如果ATX-Agent无法启动，则执行下面步骤：

   命令行执行

   ```shell
   adb shell
   chmod 755 /data/local/tmp/atx-agent
   data/local/tmp/atx-agent version # 查看版本
   /data/local/tmp/atx-agent server -d # 启动atx-agent并切换到后台运行
   ```

   打开之后可以看到手机IP

   ![image-20210318110238335](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20210318110239-157192.png) 

4. weditor 网页查看控件

   ```shell
   # 安装
   pip install --pre --upgrade weditor
   # 使用
   Python3 -m weditor
   ```

   在页面左上角选择Android，输入设备IP（10.0.32.254），点击Connect按钮。

   ![image-20210318110546218](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20210318110547-677329.png) 

5. Python连接手机

   ```python
   import uiautomator2 as u2
   
   # connect to device
   d = u2.connect_wifi('10.0.32.254')
   print(d.info)
   ```

   ![image-20210318110418212](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20210318110419-985514.png) 

   如果一切正常的话，会打印出手机信息