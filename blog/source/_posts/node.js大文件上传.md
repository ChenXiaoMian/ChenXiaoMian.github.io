---
title: node.js大文件上传
date: 2020-04-29
categories: node
tags: [node,file,upload,大文件上传]
---

## form 表单提交，有刷新、体验差

```html
<form method="post" action="http://localhost:8100" enctype="multipart/form-data">
  选择文件: <input type="file" name="f1"/> input 必须设置 name 属性，否则数据无法发送<br/><br/>
  标题：<input type="text" name="title"/><br/><br/><br/>
  <button type="submit" id="btn-0">上 传</button>
</form>
```

服务端文件的保存基于现有的库`koa-body`结合 `koa2 `实现服务端文件的保存和数据的返回。

```js
/**
 * 服务入口
 */
var http = require('http');
var koaStatic = require('koa-static');
var path = require('path');
var koaBody = require('koa-body');//文件保存库
var fs = require('fs');
var Koa = require('koa2');

var app = new Koa();
var port = process.env.PORT || '8100';

var uploadHost= `http://localhost:${port}/uploads/`;

app.use(koaBody({
    formidable: {
        //设置文件的默认保存目录，不设置则保存在系统临时目录下  os
        uploadDir: path.resolve(__dirname, '../static/uploads')
    },
    multipart: true // 开启文件上传，默认是关闭
}));

//开启静态文件访问
app.use(koaStatic(
    path.resolve(__dirname, '../static') 
));

//文件二次处理，修改名称
app.use((ctx) => {
    var file = ctx.request.files.f1;//得道文件对象
    var path = file.path;
    var fname = file.name;//原文件名称
    var nextPath = path+fname;
    if(file.size>0 && path){
        //得到扩展名
        var extArr = fname.split('.');
        var ext = extArr[extArr.length-1];
        var nextPath = path+'.'+ext;
        //重命名文件
        fs.renameSync(path, nextPath);
    }
    //以 json 形式输出上传文件地址
    ctx.body = `{
        "fileUrl":"${uploadHost}${nextPath.slice(nextPath.lastIndexOf('/')+1)}"
    }`;
});

/**
 * http server
 */
var server = http.createServer(app.callback());
server.listen(port);
console.log('demo1 server start ......   ');
```

扩展： [koa-body 文件上传自定义文件夹及文件名称](http://www.ptbird.cn/koa-body-diy-upload-dir-and-filename.html)

## 局部刷新-iframe

页面内放一个隐藏的 iframe，或者使用 js 动态创建，指定 form 表单的 target 属性值为iframe标签 的 name 属性值，这样 form 表单的 shubmit 行为的跳转就会在 iframe 内完成，整体页面不会刷新。

```html
   <iframe id="temp-iframe" name="temp-iframe" src="" style="display:none;"></iframe>
        <form method="post" target="temp-iframe" action="http://localhost:8100" enctype="multipart/form-data">
        选择文件(可多选):
            <input type="file" name="f1" id="f1" multiple/><br/> input 必须设置 name 属性，否则数据无法发送<br/>
<br/>
            标题：<input type="text" name="title"/><br/><br/><br/>

        <button type="submit" id="btn-0">上 传</button>

        </form>
        
        
<script>
var iframe = document.getElementById('temp-iframe');
iframe.addEventListener('load',function () {
      var result = iframe.contentWindow.document.body.innerText;
      //接口数据转换为 JSON 对象
      var obj = JSON.parse(result);
      if(obj && obj.fileUrl.length){
          alert('上传成功');
          
      }
      console.log(obj);
});
</script>

```
## 无刷新上传

无刷新上传文件肯定要用到XMLHttpRequest,在 ie 时代也有这个对象，单只 支持文本数据的传输，无法用来读取和上传二进制数据。
现在已然升级到了XMLHttpRequest2，较1版本有非常大的升级，首先就是可以读取和上传二进制数据，可以使用·FormData·对象管理表单数据。

```html
 <div>
        选择文件(可多选):
        <input type="file" id="f1" multiple/><br/><br/>
        <button type="button" id="btn-submit">上 传</button>
</div>


```

```js

<script>
    function submitUpload() {
        //获得文件列表，注意这里不是数组，而是对象
        var fileList = document.getElementById('f1').files;
        if(!fileList.length){
            alert('请选择文件');
            return;
        }
        var fd = new FormData();   //构造FormData对象
        fd.append('title', document.getElementById('title').value);

        //多文件上传需要遍历添加到 fromdata 对象
        for(var i =0;i<fileList.length;i++){
            fd.append('f1', fileList[i]);//支持多文件上传
        }

        var xhr = new XMLHttpRequest();   //创建对象
        xhr.open('POST', 'http://localhost:8100/', true);

        xhr.send(fd);//发送时  Content-Type默认就是: multipart/form-data; 
        xhr.onreadystatechange = function () {
            console.log('state change', xhr.readyState);
            if (this.readyState == 4 && this.status == 200) {
                var obj = JSON.parse(xhr.responseText);   //返回值
                console.log(obj);
                if(obj.fileUrl.length){
                    alert('上传成功');
                }
            }
        }

    }

    //绑定提交事件
    document.getElementById('btn-submit').addEventListener('click',submitUpload);
</script>


```
## 进度条

* 页面内增加一个用于显示进度的标签 div.progress
* js 内处理增加进度处理的监听函数xhr.upload.onprogress
* event.lengthComputable这是一个状态，表示发送的长度有了变化，可计算
* event.loaded表示发送了多少字节
* event.total表示文件总大小
* 根据event.loaded和event.total计算进度，渲染div.progress

```html
<h1>多文件上传 之 xhr formdata 上传进度条</h1>
  
  <div>选择文件:<input type="file" id="f1" multiple/></div>
  <div class="progress" id="progress">
    <span class="red"></span>
  </div>
  <button type="submit" id="btn-submit">上传</button>
  <script>
    

    function submit(){
      var progressBlock = document.getElementById('progress').firstElementChild;
      var fileList = document.getElementById('f1').files;

      progressBlock.style.width = '0';
      progressBlock.classList.remove('green');

      if(!fileList.length){
        alert('请选择文件');
        return;
      }
      console.log(fileList);
      var fd = new FormData();
      for(var i=0;i<=fileList.length;i++){
        fd.append('f1', fileList[i]);
      }
      

      var xhr = new XMLHttpRequest();
      xhr.open('POST', 'http://localhost:8100/upload', true);

      
      xhr.onreadystatechange = function(){
        console.log('state change', xhr.readyState);
        // console.log(this);
        if(this.readyState == 4 && this.status == 200) {
          var obj = JSON.parse(xhr.responseText);   //返回值
          console.log(obj);
          if(obj.fileUrl.length) {
            // alert('上传成功');
          }
        }
      }

      xhr.onprogress = updateProgress;
      xhr.upload.onprogress = updateProgress;

      function updateProgress(event) {
        console.log(event);
        if (event.lengthComputable){
          var completedPercent = (event.loaded / event.total * 100).toFixed(2);
          progressBlock.style.width = completedPercent + '%';
          progressBlock.innerHTML = completedPercent + '%';
          if(completedPercent>90){//进度条变色
            progressBlock.classList.add('green');
          }
          console.log('已上传', completedPercent);
        }
      }

      xhr.send(fd);//发送时  Content-Type默认就是: multipart/form-data; 
    }
    
    document.getElementById('btn-submit').addEventListener('click', submit);
    // document.getElementById('f1').addEventListener('change', function(event) {
    //   console.log(event);
    // });
  </script>
```

## 上传预览

* 为了预览的需要，我们这里选择上传图片文件，其他类型的也一样，只是预览不方便
* 页面内增加一个多图预览的容器div.img-box
* 根据选择的文件信息动态创建所属的预览区域和进度条以及取消按钮
* 为取消按钮绑定事件，调用`xhr.abort()`;终止上传
* 使用`window.URL.createObjectURL`预览图片，在图片加载成功后需要清除使用的内存`window.URL.revokeObjectURL(this.src)`;

## 大文件上传

**前端**

前端大文件上传网上的大部分文章已经给出了解决方案，核心是利用 `Blob.prototype.slice` 方法，和数组的 `slice` 方法相似，调用的 `slice` 方法可以返回原文件的某个切片

这样我们就可以根据预先设置好的切片最大数量将文件切分为一个个切片，然后借助 http 的可并发性，同时上传多个切片，这样从原本传一个大文件，变成了同时传多个小的文件切片，可以大大减少上传时间

另外由于是并发，传输到服务端的顺序可能会发生变化，所以我们还需要给每个切片记录顺序

```js
// 引入 spark-md5 库
const analyzeFile = (file) => {
  return new Promise((resolve, reject) => {
    var chunks = Math.ceil(file.size / chunkSize); // 获取切片的个数          
    var spark = new SparkMD5.ArrayBuffer();
    var reader = new FileReader();
    var currentChunk = 0;

    function loadNext() {
      var start = currentChunk * chunkSize;
      var end = start + chunkSize > file.size ? file.size : (start + chunkSize);
      reader.readAsArrayBuffer(blobSlice.call(file, start, end));
    };

    reader.onload = function (e) {
      const result = e.target.result;
      spark.append(result);
      currentChunk++;
      if (currentChunk < chunks) {
        loadNext();
        console.log(`第${currentChunk}分片解析完成，开始解析第${currentChunk + 1}分片`);
      } else {
        // const md5 = spark.end();
        const result = spark.end();
        // 如果单纯的使用result 作为hash值的时候, 如果文件内容相同，而名称不同的时候
        // 想保留两个文件无法保留。所以把文件名称加上。
        const sparkMd5 = new SparkMD5();
        sparkMd5.append(result);
        sparkMd5.append(file.name);
        const hexHash = sparkMd5.end();
        resolve(hexHash);
        console.log('解析完成', hexHash);
      }
    };
    reader.onerror = () => {
      console.warn('文件读取失败！');
    };
    loadNext();
  }).catch(err => {
    console.log(err);
  });
}
```

**服务端**

服务端需要负责接受这些切片，并在接收到所有切片后合并切片
这里又引伸出两个问题

何时合并切片，即切片什么时候传输完成
如何合并切片

第一个问题需要前端进行配合，前端在每个切片中都携带切片最大数量的信息，当服务端接受到这个数量的切片时自动合并，也可以额外发一个请求主动通知服务端进行切片的合并
第二个问题，具体如何合并切片呢？这里可以使用 nodejs 的 读写流（readStream/writeStream），将所有切片的流传输到最终文件的流里

```js
/**
 * 上传接口
 * 1、支持多文件上传————按日期区分文件夹
 * 2、支持分片上传————按hash值建立分片文件夹
 */
router.post('/file/upload', async (ctx) => {
  // var body = ctx.request.body;
  var files = ctx.request.files ? ctx.request.files.file : [];
  var fileSize = ctx.request.body.size || 0; // 文件大小
  var chunkCount = ctx.request.body.chunkCount || 0;  // 文件分片数量
  var fileIndex = ctx.request.body.index || 0;  // 文件索引
  var fileHash = ctx.request.body.hash || '';  // 文件hash值
  var result = [];
  var isChunk = chunkCount>0 && fileHash ? true : false; // 区分分片上传还是整体上传

  // 单文件处理
  if (!isChunk && files && !Array.isArray(files)) {
    files = [files];
  }  

  if (isChunk) {
    const chunksPath = path.join(uploadDir, fileHash, '/');
    if (!fs.existsSync(chunksPath)) fs.mkdirSync(chunksPath);
    fs.renameSync(files.path, chunksPath+fileIndex+'-'+fileHash);
    ctx.status = 200;
    ctx.body = `{
      "code": 0,
      "msg": "success"
    }`;
  } else {
    files.forEach(item => {
      if(item.size>0 && item.uploadpath) {
        result.push(uploadHost+item.uploadpath);
      }      
    });
    ctx.body = `{
      "code": 0,
      "msg": "success",
      "data": ${JSON.stringify(result)}
    }`;
  }
});

// 提取后缀名
const excludeExt = (fileName) => fileName.slice(fileName.lastIndexOf('.'), fileName.length);

// 返回已上传切片列表
const createUploadedList = async (fileHash) => fs.existsSync(path.resolve(uploadDir, fileHash)) ? await fs.readdir(path.resolve(uploadDir, fileHash)) : [];

router.post('/file/verify', async (ctx) => {
  var files = ctx.request.files ? ctx.request.files.file : [];
  var fileName = ctx.request.body.name || '';
  var hash = ctx.request.body.hash || '';

  var ext = excludeExt(fileName);
  var filePath = path.resolve(uploadDir, `${hash}${ext}`);
  var param = {
    shouldUpload: fs.existsSync(filePath) ? false : true,
    uploadedList: await createUploadedList(hash)
  }

  // console.log(await fs.readdir(path.resolve(uploadDir, hash)));

  ctx.status = 200;
  ctx.body = `{
    "code": 0,
    "msg": "success",
    "data": ${JSON.stringify(param)}
  }`;  
});

router.post('/file/merge', async (ctx) => {
  var fileName = ctx.request.body.name || ''; // 文件名字
  var fileSize = ctx.request.body.size || 0; // 文件大小
  var chunkCount = ctx.request.body.chunkCount || 0;  // 文件分片数量
  var fileHash = ctx.request.body.hash || '';  // 文件hash值

  var chunksPath = path.join(uploadDir, fileHash, '/');
  var extArr = fileName.split('.');
  var ext = extArr[extArr.length - 1];

  var writeStream = fs.createWriteStream(`${uploadDir}/${fileHash+'.'+ext}`);
  var cindex = 0;
  // 合并文件
  function mergeFile() {
    var fname = path.join(chunksPath, `${cindex}-${fileHash}`);
    var readStream = fs.createReadStream(fname);
    
    readStream.pipe(writeStream, { end: cindex + 1 == chunkCount ? true : false });
    readStream.on("end", function () {
      fs.unlink(fname, function (err) {
        if (err) {
          throw err;
        }
      });
      if (cindex + 1 < chunkCount) {
        cindex += 1;
        mergeFile();
      } else {
        fs.rmdirSync(chunksPath);
      }
    });
  }
  mergeFile();
  ctx.status = 200;
  ctx.body = `{
    "code": 0,
    "msg": 'success'
    "data": ${JSON.stringify(uploadHost + fileHash+'.'+ext)}
  }`
});
```

## 断点续传

生成 hash

无论是前端还是服务端，都必须要生成文件和切片的 hash，之前我们使用文件名 + 切片下标作为切片 hash，这样做文件名一旦修改就失去了效果，而事实上只要文件内容不变，hash 就不应该变化，所以正确的做法是根据文件内容生成 hash，所以我们修改一下 hash 的生成规则

这里用到另一个库 spark-md5，它可以根据文件内容计算出文件的 hash 值，另外考虑到如果上传一个超大文件，读取文件内容计算 hash 是非常耗费时间的，并且会引起 UI 的阻塞，导致页面假死状态，所以我们使用 web-worker 在 worker 线程计算 hash，这样用户仍可以在主界面正常的交互
由于实例化 web-worker 时，参数是一个 js 文件路径且不能跨域，所以我们单独创建一个 hash.js 文件放在 public 目录下，另外在 worker 中也是不允许访问 dom 的，但它提供了importScripts 函数用于导入外部脚本，通过它导入 spark-md5

```js
// /public/hash.js
self.importScripts("/spark-md5.min.js"); // 导入脚本

// 生成文件 hash
self.onmessage = e => {
  const { fileChunkList } = e.data;
  const spark = new self.SparkMD5.ArrayBuffer();
  let percentage = 0;
  let count = 0;
  const loadNext = index => {
    const reader = new FileReader();
    reader.readAsArrayBuffer(fileChunkList[index].file);
    reader.onload = e => {
      count++;
      spark.append(e.target.result);
      if (count === fileChunkList.length) {
        self.postMessage({
          percentage: 100,
          hash: spark.end()
        });
        self.close();
      } else {
        percentage += 100 / fileChunkList.length;
        self.postMessage({
          percentage
        });
        // 递归计算下一个切片
        loadNext(count);
      }
    };
  };
  loadNext(0);
};

```

**恢复上传**

之前在介绍断点续传的时提到使用第二种服务端存储的方式实现续传
由于当文件切片上传后，服务端会建立一个文件夹存储所有上传的切片，所以每次前端上传前可以调用一个接口，服务端将已上传的切片的切片名返回，前端再跳过这些已经上传切片，这样就实现了“续传”的效果
而这个接口可以和之前秒传的验证接口合并，前端每次上传前发送一个验证的请求，返回两种结果

服务端已存在该文件，不需要再次上传
服务端不存在该文件或者已上传部分文件切片，通知前端进行上传，并把已上传的文件切片返回给前端

所以我们改造一下之前文件秒传的服务端验证接口。

```js
$("#btn-submit").on("click", function () {
  const file = $("#f1")[0].files[0];
  if (!file) {
    alert('没有获取文件');
    return;
  }

  if (file.size < chunkSize) {
    let fd = new FormData();
    fd.append('file', file);
    axios.post('/file/upload', fd).then(res => {
      console.log(res.data);
    }).catch(e => {
      console.log(e);
    });
    return;
  }

  var chunks = Math.ceil(file.size / chunkSize); // 获取切片的个数     
  var axiosPromiseArray = [];  // axiosPromise数组
  var uploadedList = [];
  var willUploadList = [];
  var hash = '';

  // 上传切片
  function uploadChunks(list) {
    axios.all(list).then(() => {
      let fd = new FormData();
      fd.append('size', file.size);
      fd.append('name', file.name);
      fd.append('chunkCount', chunks);
      fd.append('hash', hash);
      axios.post('/file/merge', fd).then(res => {
        console.log('上传成功');
        console.log(res.data);
        console.log(file);
      }).catch(err => {
        console.log(err);
      })
    }).catch(e => {
      console.log(e);
    });
  }


  // 分析切片，验证切片
  analyzeFile(file).then(res => {
    // 验证文件是否存在
    hash = res;
    let fd = new FormData();
    fd.append('name', file.name);
    fd.append('hash', hash);
    return axios.post('/file/verify', fd);
  }).then(res => {
    let json = res.data;
    uploadedList = json.data.uploadedList || [];
    if (!json.data.shouldUpload) {
      alert('秒传:上传成功');
      return;
    } else {
      let CancelToken = axios.CancelToken;
      let cancel;
      for (let i = 0; i < chunks; i++) {
        let start = i * chunkSize;
        let end = Math.min(file.size, start + chunkSize);
        let fd = new FormData();
        fd.append('file', blobSlice.call(file, start, end));
        fd.append('name', file.name);
        fd.append('chunkCount', chunks);
        fd.append('index', i);
        fd.append('size', file.size);
        fd.append('hash', hash);
        let axiosOptions = {
          // onUploadProgress: e => {
          //   console.log(chunks, i, e, file);
          // },
          cancelToken: new CancelToken(function executor(c) {
            cancel = c;
          })
        };
        let fileHash = `${i}-${hash}`;
        // 过滤已上传的文件列表
        if (uploadedList.length > 0 && uploadedList.indexOf(fileHash) > -1) {
          continue;
        } else {
          // willUploadList.push({
          //   fd,
          //   option: axiosOptions,
          //   index: i
          // });   
          axiosPromiseArray.push(axios.post('/file/upload', fd, axiosOptions));
        }
        // 
      }

      uploadChunks(axiosPromiseArray);
      // willUploadList.filter(({ fd }) => {
      //   let hash = `${fd.index}-${fd.hash}`;
      //   console.log(hash);
      //   return !uploadedList.includes(hash);
      // });

      $("#btn-stop").on("click", function () {
        cancel && cancel('取消传输');
        $(this).hide();
        $("#btn-resume").show();
      });


    }
    console.log(json);
    // console.log(willUploadList);
    // return;
  });
});
```

## 网络请求并发控制

大文件`hash`计算后，一次发几百个http请求，计算哈希没卡，结果TCP建立的过程就把浏览器弄死了，而且我记得本身异步请求并发数的控制，本身就是头条的一个面试题

![头条](https://user-gold-cdn.xitu.io/2020/2/2/1700586de2c41042?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

思路其实也不难，就是我们把异步请求放在一个队列里，比如并发数是3，就先同时发起3个请求，然后有请求结束了，再发起下一个请求即可， 思路清楚，代码也就呼之欲出了

我们通过并发数max来管理并发数，发起一个请求max--，结束一个请求max++即可

```diff

+async sendRequest(forms, max=4) {
+  return new Promise(resolve => {
+    const len = forms.length;
+    let idx = 0;
+    let counter = 0;
+    const start = async ()=> {
+      // 有请求，有通道
+      while (idx < len && max > 0) {
+        max--; // 占用通道
+        console.log(idx, "start");
+        const form = forms[idx].form;
+        const index = forms[idx].index;
+        idx++
+        request({
+          url: '/upload',
+          data: form,
+          onProgress: this.createProgresshandler(this.chunks[index]),
+          requestList: this.requestList
+        }).then(() => {
+          max++; // 释放通道
+          counter++;
+          if (counter === len) {
+            resolve();
+          } else {
+            start();
+          }
+        });
+      }
+    }
+    start();
+  });
+}

async uploadChunks(uploadedList = []) {
  // 这里一起上传，碰见大文件就是灾难
  // 没被hash计算打到，被一次性的tcp链接把浏览器稿挂了
  // 异步并发控制策略，我记得这个也是头条一个面试题
  // 比如并发量控制成4
  const list = this.chunks
    .filter(chunk => uploadedList.indexOf(chunk.hash) == -1)
    .map(({ chunk, hash, index }, i) => {
      const form = new FormData();
      form.append("chunk", chunk);
      form.append("hash", hash);
      form.append("filename", this.container.file.name);
      form.append("fileHash", this.container.hash);
      return { form, index };
    })
-     .map(({ form, index }) =>
-       request({
-           url: "/upload",
-         data: form,
-         onProgress: this.createProgresshandler(this.chunks[index]),
-         requestList: this.requestList
-       })
-     );
-   // 直接全量并发
-   await Promise.all(list);
     // 控制并发  
+   const ret =  await this.sendRequest(list,4)

  if (uploadedList.length + list.length === this.chunks.length) {
    // 上传和已经存在之和 等于全部的再合并
    await this.mergeRequest();
  }
},

```

## demo

koa各种文件上传 **[koa-upload](https://gitee.com/bestmian/koa-upload)** 

## 参考

* [写给新手前端的各种文件上传攻略，从小图片到大文件断点续传](https://juejin.im/post/5da14778f265da5bb628e590#heading-0)
* [字节跳动面试官：请你实现一个大文件上传和断点续传](https://juejin.im/post/5dff8a26e51d4558105420ed#heading-0)
* [字节跳动面试官，我也实现了大文件上传和断点续传](https://juejin.im/post/5e367f6951882520ea398ef6#heading-0)
* [Node + js实现大文件分片上传基本原理及实践(一)](https://www.cnblogs.com/tugenhua0707/p/11246860.html)
