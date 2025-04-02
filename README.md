本项目是站在巨人的肩膀上修改的，巨人的项目 👉 [the-file-preview](https://github.com/MOONCOM/the-file-preview)

## 安装
```
npm i vue-file-previewer
```

## 描述
支持vue2和vue3，实现了对线上文件和本地上传文件的预览

## 参数选项
| 参数    | 类型  | 描述 | 返回值 |
|-------|-----|----|-----|
| url   | String | 线上文件的url | - |
| file  | File | 本地的上传文件 | - |
| loaded | Event | 文件加载成功回调 | { type: String, url: String } |
| error | Event | 文件加载、渲染报错 | { type: String, url: String, error: Error } |

## 示例
点击查看 [线上演示](https://wanlinqiang.github.io/vue-file-previewer/demo/).

## 项目地址
- [github](https://github.com/wanlinqiang/vue-file-previewer)
- [npm](https://www.npmjs.com/package/vue-file-previewer)

## 使用
main.js 中注册全局组件
```
import VueFilePreviewer from 'vue-file-previewer';
Vue.component('VueFilePreviewer', VueFilePreviewer);
```
1. 当使用线上url预览时
```
<template>
    <VueFilePreviewer :url="url" @loaded="handleLoaded" @error="handleError" />
</template>
```
2. 当预览本地文件时
``` 
<template>
    <div class="upload">
      要上传的文件： <input type="file" @change="uploadHandle"/>
    </div>
    <VueFilePreviewer :file="file"  @loaded="handleLoaded" @error="handleError" />
</template>

export default {
  name: 'Demo',
  data() {
    return {
      file: null,
    };
  },
  methods: {
    uploadHandle(data){
      this.file = data.target.files[0];
    },
    handleLoaded({ type, url }) {
      console.log(type, url)
    },
    handleError({ type, url, error }) {
      console.log(type, url, error)
    }
  },
};
```



