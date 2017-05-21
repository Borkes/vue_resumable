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
