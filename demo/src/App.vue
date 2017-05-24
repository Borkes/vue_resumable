<template>
  <div id="app">
    <img src="./assets/logo.png">
    <h1>{{ msg }}</h1>
    <div class="upload">
      <resumable :uploadList="uploadList" :target="target" :fileList="fileList"></resumable>
    </div>
  </div>
</template>

<script>
  import resumable from './resumable.vue'
  export default {
    data() {
      return {
        target: '/your/upload/path',
        uploadList: [],
        msg: '上传文件:',
        fileList: [
          {
            'uniqueIdentifier': 'id',
            'fileName': 'filename',
            'progress': () => 1,
            'size': 65536,
            'load': true,             //是否已经上传过 
          }
        ]
      }
    },
    components: {
      resumable
    }
  }

</script>

<style>
  body {
    font-family: Helvetica, sans-serif;
  }

  .upload {
    margin: 0 auto;
    width: 1000px
  }
</style>