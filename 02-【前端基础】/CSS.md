# 最佳清除浮动clearfix

**浮动元素会脱离文档流并向左/向右浮动，直到碰到父元素或者另一个浮动元素！！！**
 **浮动会导致父元素高度坍塌：**浮动使元素脱离了文档流，不占据实际位置，父元素无法被撑开。

**父元素样式**

```css
.container{
        border: 5px solid red;
    }
```

**子元素样式**

```css
.box{
    /* 浮动会脱离文档流，这个问题对整个页面布局有很大的影响。 */
    float: left;
    width: 100px;
    height: 100px;
    margin: 20px;
    background: green;
}
**清除浮动代码**
```

![image-20200616101959050](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200616102002-961895.png) 

- 全浏览器通用的clearfix方案【推荐】
- 引入了zoom以支持IE6/7
- 同时加入:before以解决现代浏览器上边距折叠的问题

```bash
.clearfix:before,
.clearfix:after {
    display: table;
    content: '';
}

.clearfix:after {
    clear: both;
}

.clearfix {
    *zoom: 1;
}
```

![image-20200616102022798](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200616102025-189675.png)  

![image-20200616102042525](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200616102045-34449.png)  

**完整代码**

```xml
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>关于浮动</title>
    <style>
    .container {
        border: 5px solid red;
    }

    .box {
        /* 浮动会脱离文档流，这个问题对整个页面布局有很大的影响。 */
        float: left;
        width: 100px;
        height: 100px;
        margin: 20px;
        background: green;
    }
    /* 
        全浏览器通用的clearfix方案【推荐】 
        引入了zoom以支持IE6/7 同时加入:before以解决现代浏览器上边距折叠的问题 
        */

    .clearfix:before,
    .clearfix:after {
        display: table;
        content: '';
    }

    .clearfix:after {
        clear: both;
    }

    .clearfix {
        *zoom: 1;
    }
    </style>
</head>

<body>
    <div class="container clearfix">
        <div class="box"></div>
        <div class="box"></div>
        <div class="box"></div>
        <!-- 不要在浮动元素上清除浮动 -->
    </div>
</body>

</html>
```