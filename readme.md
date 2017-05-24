## 使用resumable.js断点续传,vue改写组件

resumable.js 这个文件在index.html中加载即可

使用方式:
``` html
<template>
    <resumable :uploadList='uploadFile' :fileList='alreadyUpload' :clear='clear' :target: 'target'>
</template>
<script>
    export default {
        data: {
            uploadFile: [],  //父组件,上传的所有文件
            alreadyUpload: [], //已经上传的文件,可以传过去显示
            clear: true,
            target: '/upload/path'
        }
    }
</script>
```
新增一个demo,在demo目录下:
```
npm install  //安装模块
npm run dev  //运行测试
```

demo用的是[element-starter](https://github.com/ElementUI/element-starter)模板


```
├── src
│   ├── assets
│   │   └── logo.png
│   ├── App.vue     //使用上传的父组件
│   ├── index.html  //模板文件
│   ├── main.js
│   ├── resumable.js //断点续传的js文件
│   ├── resumable.vue //断点续传子组件
│   └── vendor.js
├── .babelrc
├── Makefile
├── npm-debug.log
├── package.json
├── postcss.config.js
├── webpack.config.js
└── yarn.lock
```
