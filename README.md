# 静态网站，多个html页面（非SPA），WebPack配置 (基础篇)


## Step1. 初始化项目

```javascript
mkdir multiple-htmlpages && cd multiple-htmlpages

yarn init

yarn add -D webpack webpack-cli webpack-dev-server html-webpack-plugin

mkdir src && cd src

echo >index.html

echo >index.js

```


```javascript
//index.js中的内容保持空
//index.html中添加内容

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>首页</title>
</head>
<body>
    <h1 class="page-title">首页</h1>
</body>
</html>

```

## Step2. 配置开发服务器

> 1.`webpack.dev.js`

```javascript
echo >webpack.dev.js

```

```javascript
//webpack.dev.js中添加内容

const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {

  // https://webpack.js.org/concepts/entry-points/#multi-page-application
  entry: {
    index: './src/index.js'
  },

  // https://webpack.js.org/configuration/dev-server/
  devServer: {
    port: 8080
  },

  // https://webpack.js.org/concepts/plugins/
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
      inject: true,
      chunks: ['index'],
      filename: 'index.html'
    })
  ]
};

```
> 2.`package.json`

```javascript
{
  "name": "multiple-htmlpages",
  "version": "1.0.0",
  "description": "静态网站，多个html页面，WebPack配置",
  "author": "fungmo",
  "license": "MIT",
  "scripts": {
    "start": "webpack serve --config webpack.dev.js --mode development"
  },
  "devDependencies": {
    "html-webpack-plugin": "^5.3.1",
    "webpack": "^5.30.0",
    "webpack-cli": "^4.6.0",
    "webpack-dev-server": "^3.11.2"
  }
}

```
> 3.`yarn start` 启动服务器

## Step3. 添加更多 `HTML` 页面

```javascript
echo >about.html
echo >about.js

echo >contacts.html
echo >contacts.js

```

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {

  // https://webpack.js.org/concepts/entry-points/#multi-page-application
  entry: {
    index: './src/index.js',
    about: './src/about.js',
    contacts: './src/contacts.js'
  },

  // https://webpack.js.org/configuration/dev-server/
  devServer: {
    port: 8080
  },

  // https://webpack.js.org/concepts/plugins/
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
      inject: true,
      chunks: ['index'],
      filename: 'index.html'
    }),
    new HtmlWebpackPlugin({
        template: './src/about.html',
        inject: true,
        chunks: ['about'],
        filename: 'about.html'
    }),
    new HtmlWebpackPlugin({
        template: './src/contacts.html',
        inject: true,
        chunks: ['contacts'],
        filename: 'contacts.html'
    })
  ]
};

```

## Step4. 添加CSS

```javascript
yarn add normalize.css

```

```javascript
//index.js、about.js、contacts.js 中，顶部添加
import 'normalize.css/normalize.css';

```

```javascript
yarn add -D css-loader style-loader

```

```javascript
//webpack.dev.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {

  ...

  module: {
      rules: [
          {
              test: /\.css$/,
              use: [
                  'style-loader',
                  'css-loader'
              ]
          }
      ]
  }

};

```

## Step5. 使用最新的JavaScript语言

```javascript
yarn add -D @babel/core babel-loader @babel/preset-env

```

```javascript
//webpack.dev.js
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {

...

  module: {
      rules: [
          {
            test: /\.js$/,
            exclude: /node_modules/,
            loader: 'babel-loader',
            options: {
                presets: ['@babel/preset-env']
            }
        },
      ]
  }

};

```

## Step6. 生产发布

- 压缩css，js文件，因为字节很重要；
- 在压缩文件的名称上添加一个哈希，这样我们就可以配置服务器以为具有适当缓存头的服务器提供服务，并确保浏览器仍会获取新版本；
- 确保在html中使用了正确的文件名；
- 将生成的文件存储在配置的位置；

> 1.可以使用现有的 `webpack.dev.js` 并在内部进行配置，以区分生产和开发环境来执行上述任务。
但随后变得有点复杂且难以阅读。
因此，新建一个 `webpack.prod.js` 配置文件。
首先安装更多的依赖项来帮助我们进行压缩和优化：


```javascript
yarn add -D clean-webpack-plugin mini-css-extract-plugin optimize-css-assets-webpack-plugin http-server

```
> 2.配置 `webpack.prod.js`

```javascript
echo >webpack.prod.js

```

```javascript
// webpack.prod.js

const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin'); 
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const OptimizeCssAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const buildPath = path.resolve(__dirname, 'dist');

module.exports = {

    // This option controls if and how source maps are generated.
    // https://webpack.js.org/configuration/devtool/
    devtool: 'source-map',

    // https://webpack.js.org/concepts/entry-points/#multi-page-application
    entry: {
        index: './src/index.js',
        about: './src/about.js',
        contacts: './src/contacts.js'
    },

    // how to write the compiled files to disk
    // https://webpack.js.org/concepts/output/
    output: {
        filename: '[name].[hash:20].js',
        path: buildPath
    },

    // https://webpack.js.org/concepts/loaders/
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader',
                options: {
                    presets: ['@babel/preset-env']
                }
            },
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    "css-loader"
                ]
            }
        ]
    },

    // https://webpack.js.org/concepts/plugins/
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            template: './src/index.html',
            inject: 'body',
            chunks: ['index'],
            filename: 'index.html'
        }),
        new HtmlWebpackPlugin({
            template: './src/about.html',
            inject: 'body',
            chunks: ['about'],
            filename: 'about.html'
        }),
        new HtmlWebpackPlugin({
            template: './src/contacts.html',
            inject: 'body',
            chunks: ['contacts'],
            filename: 'contacts.html'
        }),
        new MiniCssExtractPlugin({
            filename: "[name].[contenthash].css",
            chunkFilename: "[id].[contenthash].css"
        })
    ],

    // https://webpack.js.org/configuration/optimization/
    optimization: {
        minimize: true,
        minimizer: [
            new OptimizeCssAssetsPlugin({})
        ]
    }
};

```
> 3.配置 `package.json`

```javascript
"scripts": {
	"start": "webpack serve --config webpack.dev.js --mode development",
	"build": "webpack --config webpack.prod.js --mode production",
	"preview": "yarn build && http-server dist"
  },

```
> 4.要在本地构建和预览网站，请运行：

```javascript
yarn preview

```


