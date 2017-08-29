---
title: webpack learning 1
date: 2017-08-29 15:12
tags:
  - JavaScript
categories: 前端
---
# Webpack learning 

## 前言

### 目的

使用webpack打包单页应用

### 目录结构

```
│  index.html
│  package-lock.json
│  package.json
│  webpack.config.js
│  
├─dist
│      app.ddc8e77.js
│      index.html
│      ventor.js
└─src
    │  a.js
    │  index.js
    │  
    ├─image
    │      1.png
    │      
    └─style
            index.css
            index.less
```

最终的目录结构: dist是build的目录, src是source目录，webpack.config.js为webpack的配置文件

### package.json

```json
  "scripts": {
    "start": "webpack-dev-server  --hot --inline  --devtool eval --progress --colors",
    "build": "webpack"
  }
```
<!--more-->

npm start 启动webpack-dev-server 来watch文件变化，并且自动刷新页面

npm run build 启动build过程，构建应用

## 开始

创建一个文件夹，并生成package.json

```
mkdir webpack-learning 
cd webpack-learning
npm init -y
```

安装webpack 和 webpack-dev-server

```
npm i webpack webpack-dev-server -D
```

新建文件: webpack.config.js

```javascript
const path = require('path') //path模块
module.exports = {
  entry: {
    app: path.resolve(__dirname, './src/main.js')
  }, //入口
  output: {
    path: path.resolve(__dirname, './dist/js'),
    filename: '[name].[hash:7].js' //[name]会被入口中的key代替，此处为app
  }, //输出
  module: {
    loaders: [
      
    ]
  },
  plugins: [
    
  ]
}
```

在src中新建`main.js`

随便写一些代码 执行`npm run build`

最后在输出文件到`/dist/js/app.e924425.js`

## 添加一些modules和plugins

### 添加modules即一些类型的loader

####将代码转为es2015 需添加babel-loader

```
npm i babel-loader babel-core babel-preset-es2015 -D
```

在webpack.config.js中添加

```javascript
  module: {
    loaders: [
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        loader: 'babel-loader?presets[]=es2015'
      }
    ]
  }
```

这里的test是匹配文件

这里编译以后会将

```javascript
const st = 'world'
console.log(`hello ${st}`)
//转成
var st = 'world';
console.log('hello ' + st);
```

#### 添加less css style loader

```
npm i less less-loader css-loader style-loader -D
```

在webpack.config.js中添加

```
module: {
    loaders: [
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        loader: 'babel-loader?presets[]=es2015'
      },
      {
        test: /\.less$/,
        loader: 'style-loader!css-loader!less-loader'
      },
    ]
  }
```

这里要说明一下这个loader 他执行顺序是从右向左，也就是先loader less 把less转为css 再loader css 把css丢到js里面。



其他loader套路都差不多，图片等资源文件使用url-loader

### 添加 plugins

目前使用的plugins不多。需要安装的有

```javascript
const UglifyJSPlugin = require('uglifyjs-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CleanWebpackPlugin = require('clean-webpack-plugin')
```

```javascript
plugins: [
    new UglifyJSPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      filename: 'ventor.js'
    }),
    new CleanWebpackPlugin(['dist'], {
      root: __dirname,
      verbose: true,
      dry: false
    })
  ]
```

`HtmlWebpackPlugin`这个插件可以将生成的js自动插入html里面

`CommonsChunkPlugin`当然是公共模块的插件

`CleanWebpackPlugin`因为每次build后js会带hash值，文件名不一样，为了防止目录变得

```
-js
---app.a1b467a.js
---app.e924425.js
....
```

如此的臃肿，所以build的时候应当清空dist目录。这个插件就是用来清空dist目录的

## 结束

最后我们的`webpack.config.js`长这样子

```javascript
const path = require('path')
const UglifyJSPlugin = require('uglifyjs-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const CleanWebpackPlugin = require('clean-webpack-plugin')
const webpack = require('webpack')
module.exports = {
  entry: {
    app: path.resolve(__dirname, './src/main.js')
  },
  output: {
    path: path.resolve(__dirname, './dist/js'),
    filename: '[name].[hash:7].js',
    chunkFilename: '[id].bundle.[hash:7].js'
  },
  module: {
    loaders: [
      {
        test: /\.js[x]?$/,
        exclude: /node_modules/,
        loader: 'babel-loader?presets[]=es2015&presets[]=react'
      },
      {
        test: /\.less$/,
        loader: 'style-loader!css-loader!less-loader'
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 100,
          name: '[name].[hash:7].[ext]'
        }
      }
    ]
  },
  plugins: [
    new UglifyJSPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      filename: 'ventor.js'
    }),
    new CleanWebpackPlugin(['dist'], {
      root: __dirname,
      verbose: true,
      dry: false
    })
  ]
}

```

这里推荐一个不错的教程[webpack-howto](https://github.com/petehunt/webpack-howto)

下回，多页的webpack