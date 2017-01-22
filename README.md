# webpack-optimazation---3
在实际的项目中，如果要设计一个系统，我们可能会用到vue-router,或者react-router，这样整个系统利用了前端路由，实际上就是一个单页，因为所有的子页面的js最终是在router部分被加载进去，所以整个项目的js会被打包成一个特别大的js，最终导致页面在加载这个js的时候出现加载时间过大的问题。目前有以下几种优化方式。
###1，因为webpack所有的代码是以模块来进行引入的，所以可以将一些公用的模块单独抽离出来。
```javascript
var webpack = require("webpack");

module.exports = {
  entry: {
    app: "./app.js",
    vendor: ["jquery", "underscore", ...],
  },
  output: {
    filename: "bundle.js"
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin(/* chunkName= */"vendor", /* filename= */"vendor.bundle.js")
  ]
};
```
###2,但是上面的这种情况会出现一种问题，就是当其他的页面发生修改的时候，webpack会重新编译，那么这个时候vendor中的文件又会重新编译，其实这些文件是可以不用被重新编译的，所以我们可以建立一个额外的webpack.config.dll文件来单独的打包vendor中的这些文件，这样在每次重新编译的时候，vendor中的文件不再进行重新编译,同时利用DllPlugin插件。执行webpack --config filename,这样生成了lib.js文件,同时生成的mefest.json文件实际上是lib.js与npmmodules包中的文件的一个映射map，直接在页面中引入后，当其他文件变化时，vendor中的代码不再被重新编译。
```javascript
var webpack = require('webpack');

var vendors = [
    'react',
    'react-dom',
    'react-router',
    'react-redux',
    'react-router-redux',
    'antd',
    'immutable',
    'redux',
    'underscore',
    'moment',
    'jquery',
    'lodash',
    'clipboard'
];

module.exports = {
    output: {
        path: 'build',
        filename: '[name].js',
        library: '[name]',
    },
    entry: {
        "lib": vendors,
    },
    plugins: [
        new webpack.optimize.UglifyJsPlugin(
            {
                compress: {
                   //supresses warnings, usually from module minification
                    warnings: false
                }
            }
        ),
        new webpack.DllPlugin({
            path: 'manifest.json',
            name: '[name]',
            context: __dirname,
        }),
    ]
};```
