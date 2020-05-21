# POST请求的两种编码格式

POST请求的两种编码格式：`application/x-www-urlencoded`是浏览器默认的编码格式，用于键值对参数，参数之间用`&`间隔；`multipart/form-data`常用于文件等二进制，也可用于键值对参数，最后连接成一串字符传输(参考Java OK HTTP)。除了这两个编码格式，还有`application/json`也经常使用。

