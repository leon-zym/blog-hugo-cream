---
title: Webpack 学习 环境配置
date: 2022-05-05
categories: Programming
tags: [Webpack]
---

npm下载`webpack`和`webpack-cli`两个包。

# webpack.config.js配置文件

所有构建工具都是基于node.js平台运行的，所以模块化语法遵循commonjs。而项目文件中模块化语法多遵循ES6的。

webpack.config.js配置文件五大核心：entry、output、module、plugins、mode

```js
// resolve是用来拼接绝对路径的方法
const { resolve } = require('path');

module.exports = {
  // 入口起点
  entry: './src/index.js',
  
  // 输出
  output: {
    // 输出文件名
    filename: 'built.js',
    // 输出路径
    // __dirname是nodejs的变量，代表当前文件目录的绝对路径
    path: resolve(__dirname, 'build')
  },
  
  // loader的配置
  module: {
    // 详细loader配置
  },
  
  // plugins的配置
  plugins: [
    // 详细plugins的配置
  ],
  
  // 模式
  mode: 'development', // 开发模式
  // mode: 'production'  // 生产模式
}
```

配置好后使用`webpack`命令编译打包输出。

# 配置开发环境

```js
const { resolve } = require('path');
// plugins需要先引入在配置使用
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/js/index.js',
  output: {
    filename: 'js/built.js',
    path: resolve(__dirname, 'build')
  },
  
  // loader的配置
  module: {
    rules: [
      {
        // 处理less资源
        test: /\.less$/,
        use: ['style-loader', 'css-loader', 'less-loader']
      },
      {
        // 处理css资源
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        // 处理图片资源
        test: /\.(jpg|png|gif)$/,
        loader: 'url-loader',
        options: {
          // 体积小于8kb的图片将被base64
          limit: 8 * 1024,
          // hash名称长度改为10位
          name: '[hash:10].[ext]',
          // 为和下面html-loader配合使用，需关闭es6模块化语法
          esModule: false,
          outputPath: 'imgs'
        }
      },
      {
        // 处理html中img资源
        test: /\.html$/,
        loader: 'html-loader'
      },
      {
        // 处理其他资源
        exclude: /\.(html|js|css|less|jpg|png|gif)/,
        loader: 'file-loader',
        options: {
          name: '[hash:10].[ext]',
          outputPath: 'media'
        }
      }
    ]
  },
  
  // plugins的配置
  plugins: [
    // 处理HTML资源
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ],
  
  // 模式选择
  mode: 'development',
  
  // 配置开发调试自动化
  devServer: {
    contentBase: resolve(__dirname, 'build'),
    compress: true,
    port: 3000,
    open: true
  }
};
```

配置好后使用`npx webpack-dev-server `命令临时、自动编译打包运行。

# 配置生产环境

webpack.config.js中配置：

```js
const { resolve } = require('path');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCssAssetsWebpackPlugin = require('optimize-css-assets-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

// 定义nodejs环境变量：选择使用browserslist的哪个环境（production默认 / development）
process.env.NODE_ENV = 'production';

// 在外部定义一个包含loader的数组，为的是方便复用loader
const commonCssLoader = [
  // 提取css资源成单独文件
  MiniCssExtractPlugin.loader,
  'css-loader',
  {
    // css兼容性处理。还需在package.json中定义browserslist
    loader: 'postcss-loader',
    options: {
      ident: 'postcss',
      plugins: () => [require('postcss-preset-env')()]
    }
  }
];

module.exports = {
  entry: './src/js/index.js',
  output: {
    filename: 'js/built.js',
    path: resolve(__dirname, 'build')
  },
  
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [...commonCssLoader]
      },
      {
        test: /\.less$/,
        use: [...commonCssLoader, 'less-loader']
      },
      
      // 正常来讲，一个文件只能被一个loader处理。当一个文件要被多个loader处理时，则一定要指定loader执行的先后顺序(从右到左，从下到上依次执行)。这里为先执行eslint，再执行babel
      {
        // js语法检查，还需在package.json中配置eslintConfig
        test: /\.js$/,
        exclude: /node_modules/,
        // 优先执行
        enforce: 'pre',
        loader: 'eslint-loader',
        options: {
          fix: true
        }
      },
      {
        // js兼容性处理
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        options: {
          presets: [
            [
              '@babel/preset-env',
              {
                // 按需加载
                useBuiltIns: 'usage',
                corejs: {version: 3},
                targets: {
                  chrome: '60',
                  firefox: '50'
                }
              }
            ]
          ]
        }
      },
      {
        test: /\.(jpg|png|gif)/,
        loader: 'url-loader',
        options: {
          limit: 8 * 1024,
          name: '[hash:10].[ext]',
          outputPath: 'imgs',
          esModule: false
        }
      },
      {
        test: /\.html$/,
        loader: 'html-loader'
      },
      {
        exclude: /\.(js|css|less|html|jpg|png|gif)/,
        loader: 'file-loader',
        options: {
          outputPath: 'media'
        }
      }
    ]
  },
  
  plugins: [
    // 提取css资源成单独文件
    new MiniCssExtractPlugin({
      filename: 'css/built.css'
    }),
    // 压缩css资源
    new OptimizeCssAssetsWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: './src/index.html',
      // 压缩HTML资源
      minify: {
        collapseWhitespace: true,
        removeComments: true
      }
    })
  ],
  
  // 当模式调为生产模式时，js资源会被自动压缩
  mode: 'production'
};
```

package.json中追加配置：

```json
"browserslist": {
  "development": [
    "last 1 chrome version",
    "last 1 firefox version",
    "last 1 safari version"
  ],
  "production": [
    ">0.2%",
    "not dead",
    "not op_mini all"
  ]
},

"eslintConfig": {
  "extends": "airbnb-base",
  "env": {
    "browser": true
  }
}
```

