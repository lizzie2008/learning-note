# 路径别名（@）

场景：我们在利用VS   code开发的时候，经常需要引入一个文件、图片等，这时候我们会下载（path-intellisense）等插件来获得路径的提示，没错，这确实很方便，但是我们在利用webpack构建脚手架开发的时候，或者说用vue开发的时候，我们会利用到路径别名这么一个理念，就是说  我们可以设置一个变量  比如 @ 来表示一个相对路径的文件目录，已达到好看，或简写的效果（vue 官方十分推荐。）但是这里我们就出现了一些问题。

## 问题一

如果我们用了路径别名（@）等，那么我们的插件（path-intellisense）将不支持自动提示。

**解决办法：**

插件的mappings设置如下。

```bash
"path-intellisense.mappings": {
      "@": "${workspaceRoot}/src"
   }
```

## 问题二

我们希望 Ctrl + 鼠标左键点击一个外部方法时，能够快速跳转到对应的外部文件。

> 很多人说 webstorm 都可以。那我告诉你 VS code 也可以，vs code不比任何一个前端开发编辑器差。只是你的使用方法不对。同样的原因，之所以不能跳转，也是因为路径别名（@）不能被 VScode 识别，所以你同样需要在vs code 里面做映射

**解决办法：**

> 在项目package.json所在同级目录下创建文件jsconfig.json,写上如下配置。

```json
{
    "compilerOptions": {
        "target": "ES6",
        "module": "commonjs",
        "allowSyntheticDefaultImports": true,
        "baseUrl": "./",
        "paths": {
          "@/*": ["src/*"]
        }
    },
    "exclude": [
        "node_modules"
    ]
}
```