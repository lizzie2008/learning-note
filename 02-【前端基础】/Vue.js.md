# 异常解决

1、安装vue-cli后，执行`vue --version`报错：

```
vue : 无法加载文件 C:\Users\Administrator\AppData\Roaming\npm\vue.ps1，因为在此系统上禁止运行脚本。
```

解决方案：

1. 以管理员身份（我没以管理员身份也解决了，都试下好）运行PowerShell
2. 执行：get-ExecutionPolicy，回复Restricted，表示状态是禁止的
   3.执行：set-ExecutionPolicy RemoteSigned
   4.选择Y
   5.在此执行

# 配置环境

1、在package.json文件中添加

```json
"scripts": {
    "serve": "vue-cli-service serve", //调用开发api
    "build": "vue-cli-service build", //上线
    "test": "vue-cli-service build --mode test",//需要添加的内容，测试
},
```

2、在根目录下创建.env文件，并配置

```json
NODE_ENV = 'production'
VUE_APP_FLAG = 'pro' //vue代码可以直接使用VUE_APP_名字
```

3、在根目录下创建.env.test文件

```json
NODE_ENV = 'production'
VUE_APP_FLAG = 'test'
outputDir = test  //可以更改打包时输出的目录名字，默认为dist
```

4、在根目录下创建vue.config.js

```json
module.express = {
baseUrl: process.env.NODE_ENV === 'production' ? '/static/' : './',
devServe: {
port: 8080,
// disableHostCheck:true,//处理host不识别问题
},
baseUrl: '/', //基本路径，不要随意更改
outputDir: process.env.outputDir, // 打包生成目录
assetDir: 'static',
lintOnSave: false,
runtimeCompiler: true, // 是否使用包含运行时编译器的 Vue 构建版本
productionSourceMap: false, // 生产环境的 source map

}
```

005、在main.js里配置api变量

```json
/*第一层if判断生产环境和开发环境*/
if (process.env.NODE_ENV === 'production') {
    /*第二层if，根据.env文件中的VUE_APP_FLAG判断是生产环境还是测试环境*/
    if (process.env.VUE_APP_FLAG === 'pro') {
        //production 生产环境
        axios.defaults.baseURL = 'http://api.xinggeyun.com';//路径

    } else {
        //test 测试环境
        axios.defaults.baseURL = 'http://192.168.0.152:8102';//路径
} } else { //dev 开发环境 axios.defaults.baseURL = 'http://192.168.0.152:8102';//路径
 }
```

最后npm run test 或者 yarn run test 

# vue-cli3使用svg的最佳实践

## 封装svg组件

代码都是基于 vue 的实例(ps: react 也很简单，原理都是类似的)

```jsx
//components/Icon-svg
<template>
  <svg class="svg-icon" aria-hidden="true">
    <use :xlink:href="iconName"></use>
  </svg>
</template>

<script>
export default {
  name: 'icon-svg',
  props: {
    iconClass: {
      type: String,
      required: true
    }
  },
  computed: {
    iconName() {
      return `#icon-${this.iconClass}`
    }
  }
}
</script>

<style>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```

使用svg组件

```jsx
//引入svg组件
import IconSvg from '@/components/IconSvg'

//全局注册icon-svg
Vue.component('icon-svg', IconSvg)

//在代码中使用
<icon-svg icon-class="password" />
```

## svg雪碧图

通过这篇文章 [ iconfont的三种使用方式及其优缺点](https://www.jianshu.com/p/5f6e22b5efea)，我们已经能在项目中使用svg图标了，但是有一个麻烦是，eg：我们会在iconfont上面创建一个项目，然后把需要的图标添加到项目里，接下来再生成文件，然后把文件保存下来。然而当我再去添加几个图标的时候，上面的动作我们需要重来一遍，这个太麻烦了，所以我们会用到svg-sprite(svg雪碧图,跟以前的css-sprite类似)。具体操作如下：

1. 在vue项目中配置svg-sprite-loader
    请参考文章[vue-cli3中新添加svg-sprite-loader和file-loader](https://www.jianshu.com/p/82496e12e868)
2. 使用svg
    配置好以后，我们把svg文件都放到根目录的icons下面，使用的时候，需要那个就加载那个，举例：(这里的svg-icon是我们上面封装的svg组件)

```xml
<template>
  <div id="app">
    <svg-icon iconClass="me"></svg-icon>
  </div>
</template>

<script>
import '@/icons/svg/me.svg'
export default {
  
}
</script>
```

我们在项目中使用了很多svg图标，svg-sprite-loader能帮我们把这些图标合成一张大的雪碧图，就像这样

![image-20200611152516510](https://typora-lancelot.oss-cn-beijing.aliyuncs.com/typora/20200611152517-449980.png) 

## 自动导入所有图标

经过上面的一波操作，我们实现了按需加载，如果我们要添加图标，直接往icons里放就好了，这个svg的图标可以是你们ui制作的，也可以是从iconfont图标库download下来的。不过，这里还有两个地方可以再优化一下。

 首先我们创建一个专门放置图标 icon 的文件夹如：`@/src/icons`，将所有 icon 放在这个文件夹下。 之后我们就要使用到 webpack 的 [require.context](https://link.juejin.im/?target=https%3A%2F%2Fwebpack.js.org%2Fguides%2Fdependency-management%2F%23require-context)。很多人对于 `require.context`可能比较陌生，直白的解释就是

```ruby
require.context("./test", false, /.test.js$/);
这行代码就会去 test 文件夹（不包含子目录）下面的找所有文件名以 .test.js 结尾的文件能被 require 的文件。
更直白的说就是 我们可以通过正则匹配引入相应的文件模块。
```

require.context有三个参数：

- directory：说明需要检索的目录
- useSubdirectories：是否检索子目录
- regExp: 匹配文件的正则表达式
   了解这些之后，我们就可以这样写来自动引入 @/src/icons 下面所有的图标了

```tsx
const requireAll = requireContext => requireContext.keys().map(requireContext)
const req = require.context('./svg', false, /\.svg$/)
requireAll(req)
```

复制代码之后我们增删改图标直接直接文件夹下对应的图标就好了，什么都不用管，就会自动生成 svg symbol了。

## 批量下载单个svg文件

 我们上面的操作，在icons里面，一个svg文件就是一个图标，而阿里图标库下载下来的文件是把所有的图标都合在一个文件里面，这显然不符合我们的要求，但是一个一个去下载也觉得麻烦，这时可以这样操作

点击批量操作，选择要添加的图标，点击批量加入到购物车，去购物车页面，点击下载素材，出现以下页面，点击svg即可

