<template>
    <div class="upload">
        <div class="resumable-error" v-show="resumable_error">
            Your browser, unfortunately, is not supported by Resumable.js. The library requires support for <a href="http://www.w3.org/TR/FileAPI/">the HTML5 File API</a>            along with <a href="http://www.w3.org/TR/FileAPI/#normalization-of-params">file slicing</a>.
        </div>

        <div class=resumable-drop v-show="resumable_drop" :class="{'resumable-dragover': resumableDrag}" @dragover="resumableDrag=true"
            @drop="resumableDrag=false" @dragleave="resumableDrag=false">
            Drop video files here to upload or
            <el-button type="text" class="resumable-browse">select from your computer</el-button>
        </div>

        <div class="resumable-list" v-show="resumable_list">
            <ul v-for="file in fileList" :key="file.uniqueIdentifier">
                <li :class="'resumable-file-' + file.uniqueIdentifier" style="position:relative; height: 30px">
                    <span style="position: absolute;left: 0">{{file.load ? 'uploaded' : 'upload'}}</span>
                    <span class="resumable-file-name">{{file.fileName}}</span>
                    <span class="resumable-file-progress" style="position: absolute;left:300px">
                    {{Math.floor(file.progress() * 100) + '%'}}
                    </span>
                    <span style="position:absolute;right:40px">{{file.size | size}}</span>
                    <el-button style="position:absolute;right:5px;" @click="removeFile(file)" type="text">X</el-button>
                </li>
                <li v-show="!file.load" :class="'resumable-progress-' + file.uniqueIdentifier" style="height:25px;margin-top:5px">
                    <table width="100%">
                        <tr>
                            <td width="80%" style="vertical-align:middle">
                                <div class="progress-container">
                                    <div class="progress-bar" :style="progress_bar"></div>
                                </div>
                            </td>
                            <td class="progress-text" nowrap="nowrap" :style="{color: 'activeColor'}">
                                {{progress_text}}
                            </td>
                        </tr>
                    </table>
                </li>
            </ul>
        </div>
    </div>
</template>

<style>
    .resumable-error {
        font-size: 14px;
        font-style: italic;
    }

    .resumable-drop {
        padding: 15px;
        font-size: 13px;
        text-align: center;
        color: #666;
        font-weight: bold;
        background-color: #eee;
        border: 2px dashed #aaa;
        border-radius: 10px;
        margin-top: 40px;
        z-index: 9999;
    }

    .resumable-dragover {
        padding: 30px;
        color: #555;
        background-color: #ddd;
        border: 1px solid #999;
    }
    /* Uploader: Progress bar */

    .progress-container {
        height: 7px;
        background: #9CBD94;
        position: relative;
    }

    .progress-bar {
        position: absolute;
        top: 0;
        left: 0;
        bottom: 0;
        background: #45913A;
        width: 0;
    }

    .progress-text {
        font-size: 11px;
        line-height: 9px;
        padding-left: 10px;
    }

    .resumable-file-name {
        position: absolute;
        left: 80px;
        width: 250px;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis
    }

    .resumable-list {
        background-color: #eee;
        margin: 5px 0 5px;
        width: 50%;
        overflow: auto;
    }
    ul {
        list-style: none;
        /*-webkit-padding-start: 0*/
    }
</style>

<script>
    import Resumable from './resumable.js'
    export default {
        data() {
            return {
                resumable_error: false,
                resumable_drop: false,
                resumable_list: false,
                resumableDrag: false,
                activeColor: 'green',
                progress_text: '',
                progressState: true,
                progress_bar: {
                    width: ''
                }
            }
        },
        props: {
            uploadList: {        //传回的所有上传文件
                type: Array,
                required: true
            },
            target: {
                type: String,
                required: true
            },
            fileList: {           //已经上传过的文件显示
                type: Array,
                default: () => []
            }
        },
        methods: {
            removeFile(file) {
                if (!file.load) {
                    file.cancel();
                }
                for (let i = 0; i < this.fileList.length; i++) {
                    if (this.fileList[i].uniqueIdentifier === file.uniqueIdentifier) {
                        this.fileList.splice(i, 1);
                    }
                }
                for (let i = 0; i < this.uploadList.length; i++) {
                    if (this.uploadList.uid === file.uniqueIdentifier) {
                        this.uploadList.splice(i, 1);
                    }
                }
            }
        },
        filters: {
            size: function (value) {
                if (!value) {
                    return 0;
                }
                let s = value / (1024 * 1024);
                if (parseInt(s, 10) > 0) {
                    return s.toFixed(1) + 'M';
                } else {
                    return (value / 1024).toFixed(1) + 'Kb'
                }
            }
        },
        watch: {
            fileList(newValue, oldValue) {
                if (newValue.length === 0) {
                    for (let i = 0; i < oldValue.length; i++) {
                        oldValue[i].cancel()
                    }
                }
            }
        },
        mounted() {
            this.$nextTick(() => {
                let r = new Resumable({
                    target: this.target,
                    chunkSize: 1024 * 1024,
                    simultaneousUploads: 4,
                    testChunks: true,
                })
                this.r = r;
                if (!r.support) {
                    this.resumable_error = true;
                } else {
                    this.resumable_drop = true;
                    r.assignDrop(document.getElementsByClassName('resumable-drop'));
                    r.assignBrowse(document.getElementsByClassName('resumable-browse'));
                    this.resumable_list = true;
                    r.on('fileAdded', (file) => {
                        this.fileList.push({
                            'uniqueIdentifier': file.uniqueIdentifier,
                            'fileName': file.fileName,
                            'progress': () => file.progress(),
                            'size': file.size,
                            'cancel': () => file.cancel(),
                            'load': false,             //是否已经上传过 
                        })
                        r.upload()
                    })
                    r.on('fileSuccess', (file, message) => {
                        let res = JSON.parse(message);
                        this.uploadList.push({ url: res.message, name: file.fileName, uid: file.uniqueIdentifier, size: file.size })
                        this.progress_text = '上传成功'
                    })
                    r.on('fileError', () => {
                        this.progressState = false;
                        this.activeColor = 'red';
                        this.progress_text = '上传失败';
                    })
                    r.on('fileProgress', (file) => {
                        this.progress_bar = {
                            width: Math.floor(file.progress() * 100) + '%'
                        }
                    })
                }
            })
        }
    }

</script>