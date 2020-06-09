# POST请求的两种编码格式

POST请求的两种编码格式：`application/x-www-urlencoded`是浏览器默认的编码格式，用于键值对参数，参数之间用`&`间隔；`multipart/form-data`常用于文件等二进制，也可用于键值对参数，最后连接成一串字符传输(参考Java OK HTTP)。除了这两个编码格式，还有`application/json`也经常使用。

# HTTPS

HTTPS比HTTP多出的S的事情
1. 请求https连 接获取证书(公钥)
2. 客户端给服务器发送 (对称加密<公钥>)：随机数的密文
3. 客户端同时给发服务发送: (对称加密<公钥>)：随机数+私钥的密文
4. 服务器根据公钥解密出随机数，同时解密出私钥
5. 客户端使用非对称加密进行数据传输，客户端使用公钥加密，服务器使用私钥解密

![image-20200605103534563](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200605103538-680524.png)  