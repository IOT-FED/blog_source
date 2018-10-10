---
title: webpack使用初探
author: simin
date: 2018-10-10 12:30:00
tags: 实验
---

### 想法
 webpack是目前最流行的项目打包工具，看起来好用就是不会用，相信很多朋友都有和我一样看到vue-cli脚手架里花里胡哨的配置懵逼的经历，从来都不敢去改，生怕改了一点就原地爆炸哦，可能有很多配置是用不上的，这样的话，不说理解打包过程，就有单纯的使用也有点困难。

 现在想要把多个vue组件打包并通过import引入使用（一次打包多个组件一次引入使用，就像element一样），我这里想尝试<font color="#00ffff" size=3 >从头到尾演示一遍打包的过程，过程中尽量简化操作和配置，</font>并记录细节问题。

### 思路
 还是撘起vue的脚手架工程vue-cli，只不过不使用默认的打包命令，自定义一个脚本命令，写一个webpack打包的配置项，然后就可以打包完成了，当然最后要验证一下好不好用

### 开始吧

第一步： 单独建了一个文件夹，在里面完成打包操作，外面跑起工程引用打包好的文件，方便验证：
   
   ![avatar](https://chensimin0103.github.io/ChenSimin.github.io/img/res/20181009-项目目录.jpg)

  因为包管理工具是npm，内容要符合npm规范，在此路径下先执行`npm init`，填入一些必要配置，之后会生成一份package.json文件

   ![avatar](https://chensimin0103.github.io/ChenSimin.github.io/img/res/20181009-npm包初始化.jpg)

   package.json 需要添加的几项依赖：
   ``` json
    "vue": "^2.4.4",
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-preset-env": "^1.6.0",
    "babel-preset-stage-3": "^6.24.1",
    "cross-env": "^5.0.5",
    "css-loader": "^0.28.7",
    "file-loader": "^1.1.4",
    "node-sass": "^4.5.3",
    "sass-loader": "^6.0.6",
    "vue-loader": "^13.0.5",
    "vue-template-compiler": "^2.4.4",
    "webpack": "^3.6.0",
    "webpack-dev-server": "^2.9.1"
   ```
   脚本命令这样写吧
   ``` json
    "build": "webpack --config webpack.config.js"
   ```
   还需要修改下打包之后工程的入口配置，设置为打包之后的js文件，注意这里文件名及路径和之后的webpack打包配置有关
   ``` json
    "main": "dist/demo.js",
   ```

   运行 `npm install` ，安装项目依赖

第二步： 在src/components目录下写两个简单的vue组件，注意要添加name属性，内容随便写一写（这里markdown引入vue代码段有问题 -_-|| ）：

   第一个vue文件：
  ```javascript
        <template>
          <div>
            <p class="me">这是一个想要被打包出来的vue组件</p>
          </div>
        </template>

        <script>
        export default {
          name: 'demo',
        };
        </script>

        <style>
        .me {
          color: blue;
        }
        </style>
  ```
   第二个vue文件：
  ```javascript
        <template>
          <div>
            <p class="hello">这是第二个想要被打包出来的vue组件</p>
          </div>
        </template>

        <script>
        export default {
          name: 'demo2',
        };
        </script>

        <style>
        .hello {
          color: red;
        }
        </style>
  ```
  因为我们期望的使用方法是将所有组件打包成一个插件，并通过一次Vue.use()来使用，必需提供一个install方法（参见[vue.js全局api之Vue.use](https://cn.vuejs.org/v2/api/#Vue-use)），在其中使用Vue.component()来将我们的组件注册为Vue的全局组件，

  在 index.js 中做这件事：

  ``` javascript
    import vueDemo from './components/demo';
    import vueDemo2 from './components/demo2';

    const demo = {
      install(Vue, options) {
        Vue.component(vueDemo2.name, vueDemo2);
        Vue.component(vueDemo.name, vueDemo);
      },
    };

    // 以 <script> 标签形式使用组件时做的特殊处理
    if (typeof window !== 'undefined' && window.Vue) {
      window.Vue.use(demo);
    }

    export default demo;
  ```

  在 webpack.config.js 中定义一份简化的webpack配置：
  ``` javascript
    const path = require('path')

    module.exports = {
      entry: './src/index.js', //打包 插件入口
      output: {
        // 输出路径及文件名
        path: path.resolve(__dirname, './dist'),
        publicPath: '/dist/',
        filename: 'demo.js',
        // 支持多种模块系统
        library:'demo',
        libraryTarget:'umd',
        umdNamedDefine:true
      },
      module: {
        rules: [
          {
            test: /\.css$/,
            use: [
              'vue-style-loader',
              'css-loader'
            ],
          },
          {
            test: /\.scss$/,
            use: [
              'vue-style-loader',
              'css-loader',
              'sass-loader'
            ],
          },
          {
            test: /\.vue$/,
            loader: 'vue-loader',
          },
          {
            test: /\.js$/,
            loader: 'babel-loader',
            exclude: /node_modules/
          },
          {
            test: /\.(png|jpg|gif|svg)$/,
            loader: 'file-loader',
            options: {
              name: '[name].[ext]?[hash]'
            }
          }
        ],
        loaders: [
          {
            test: /\.js$/,
            exclude: /(node_modules|bower_components)/,
            loader: 'babel',
            query: {
              presets: ['es2015'],
            },
          },
        ],
      },
      resolve: {
        alias: {
          'vue$': 'vue/dist/vue.esm.js'
        },
        extensions: ['*', '.js', '.vue', '.json']
      },
    }
  ```

第三步： 开始打包

运行打包命令 `npm run build`，之后可以看到目录下多了这个文件：

   ![avatar](https://chensimin0103.github.io/ChenSimin.github.io/img/res/20181009-插件打包之后.jpg)

再通过正常引入使用组件的方式来用，在vue工程中的main.js文件里，实例Vue的代码之前添加：
``` javascript
  import demo from './componentsPackage'
  Vue.use(demo)
```
这样就把我们打包好的插件挂到了Vue实例上，之后就可以在任意vue文件里通过引入标签的方式使用组件，比如这样：
``` javascript
  <template>
    <div>
      <demo></demo>
      <demo2></demo2>
    </div>
  </template>
```
效果：

   ![avatar](https://chensimin0103.github.io/ChenSimin.github.io/img/res/20181009-实现效果.jpg)

下一步，再写个README吧，就可以尝试把打包完的组件发布在npm上，给其他开发者下载使用，嗯。

> 这里有完整的代码示例 ： https://github.com/ChenSimin0103/Demo/tree/master/webpack%E6%89%93%E5%8C%85%E5%88%9D%E6%8E%A2%20demo