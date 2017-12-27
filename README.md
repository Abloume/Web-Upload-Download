# Web-Upload-Download
关于Web上传下载的积累总结

# 前言

HTML5中提供的文件API在前端中有着丰富的应用，上传、下载、读取内容等在日常的交互中很常见。而且在各个浏览器的兼容也比较好，包括移动端，除了IE只支持IE10以上的版本。想要更好地掌握好操作文件的功能，先要熟悉每个API。

在Web浏览器上传文件一般有以下几种方式：

- FileList对象和File对象
- 原生JavaScript实现Ajax上传文件
- jQuery实现Ajax上传文件
- form+iframe上传文件

# 1. FileList对象和File对象

　　HTML5中的 `input[type="file"]` 标签有个 multiple 属性，允许用户选择多个文件，FileList对象则就是表示用户选择的文件列表。这个列表中的每一个文件，就是一个file对象。 

　　HTML5也添加了文件对象（File），可以方便的获得文件本身的一些信息（name、size、type），在上传前知道文件的大小，实现限制上传大小。

```
<input type="file" id="files" multiple accept="image/gif,image/jpeg,image/jpg,image/png">  // accept 属性，可以用来规定能够通过文件上传进行提交的文件类型。
<script>
    var elem = document.getElementById('files');
    elem.onchange = function (event) {
        var files = event.target.files;
        for (var i = 0; i < files.length; i++) {
            console.log(files[i].name);            
        }
    }
</script>
```

# 2. Blob 对象

Blob 对象相当于一个容器，可以用于存放二进制数据。它有两个属性，size 属性表示字节长度，type 属性表示 MIME 类型。

>一个Blob对象就是一个包含有只读原始数据的类文件对象。Blob对象中的数据并不一定得是JavaScript中的原生形式。File接口基于Blob，继承了Blob的功能,并且扩展支持了用户计算机上的本地文件。

## 如何创建

### 1）Blob 对象可以使用 Blob() 构造函数来创建。

```
var blob = new Blob(['hello'], {type:"text/plain"});
```

Blob 构造函数中的第一个参数是一个数组，存放数据可以是任意多个ArrayBuffer、ArrayBufferView、Blob或者 DOMString对象。

#### 1、创建一个装填DOMString对象的Blob对象

```
var s = "<div>Hello World!</div>";
var blob = new Blob([s],{type:"text/xml"});
```

#### 2、创建一个装填ArrayBuffer对象的Blob对象

```
var abf = new ArrayBuffer(8);
var blob = new Blob([abf],{type:"text/plain"});
```

#### 3、创建一个装填ArrayBufferView对象的Blob对象

```
var abf = new ArrayBuffer(8);
var abv = Int16Array(abf);
var blob = new Blog(abv,{type:"text/plain"});
```

### 2）Blob 对象可以通过 slice() 方法来返回一个新的 Blob 对象。

```
var newblob = blob.slice(0,5, {type:"text/plain"});
```

　　slice() 方法使用三个参数，均为可选。第一个参数代表要从Blob对象中的二进制数据的起始位置开始复制，第二个参数代表复制的结束位置，第三个参数为 Blob 对象的 MIME 类型。

### 3）canvas.toBlob() 也可以创建 Blob 对象。toBlob() 使用三个参数，第一个为回调函数，第二个为图片类型，默认为 image/png，第三个为图片质量，值在0到1之间。

```
var canvas = document.getElementById('canvas');
canvas.toBlob(function(blob){ console.log(blob); }, "image/jpeg", 0.5);
```

## 下载文件

Blod 对象可以通过 window.URL 对象生成一个网络地址，结合 a 标签的 download 属性来实现下载文件功能。

比如把 canvas 下载为一个图片文件。

```
createDownload("download.txt","download file");

function createDownload(fileName, content){
    var blob = new Blob([content]);
    var link = document.createElement("a");
    link.innerHTML = fileName;
    link.download = fileName;
    link.href = URL.createObjectURL(blob);
    document.getElementsByTagName("body")[0].appendChild(link);
}
```

执行后页面上会生成此Blob对象的地址，点击后可下载。

# 3.FileReader 对象

FileReader 对象主要用来把文件读入内存，并且读取文件中的数据。通过构造函数创建一个 FileReader 对象

```
var reader = new FileReader();
```

该对象有以下方法：

- abort：中断读取操作。
- readAsArrayBuffer：读取文件内容到ArrayBuffer对象中。
- readAsBinaryString：将文件读取为二进制数据。
- readAsDataURL：将文件读取为data: URL格式的字符串。
- readAsText：将文件读取为文本。

## 上传图片预览

在常见的应用就是在客户端上传图片之后通过 readAsDataURL() 来显示图片。

```
<input type="file" id="files" accept="image/jpeg,image/jpg,image/png">
<img src="blank.gif" id="preview">
<script>
    var elem = document.getElementById('files'),
        img = document.getElementById('preview');
    elem.onchange = function () {
        var files = elem.files,
            reader = new FileReader();
        if(files && files[0]){
            reader.onload = function (ev) {
                img.src = ev.target.result;
            }
            reader.readAsDataURL(files[0]);
        }
    }
</script>
```

## 数据备份与恢复

FileReader 对象的 readAsText() 可以读取文件的文本，结合 Blob 对象下载文件的功能，那就可以实现将数据导出文件备份到本地，当数据要恢复时，通过 input 把备份文件上传，使用 readAsText() 读取文本，恢复数据。

# 4. WebSocket上传

```
var url = "ws://localhost:8081/upload";
var ws = new WebSocket(url);
ws.binaryType = 'arraybuffer';
ws.onopen = function () {
    window.console.log('websocket connection success ...');
};

//...
ws.onerror = function (error) {
  window.console.log('WebSocket Error ' + error);
};

//...
function uploadFile(file){
   //实例化FileReader对象
   var fr = new FileReader();
   //定义文件加载完的监听事件，执行回调函数 
   fr.addEventListener("loadend", function() {
      ws.send(fr.result);
   });
   //把文件加载进ArrayBuffer中
   fr.readAsArrayBuffer(file);
}
```

实际使用中，浏览器websocket用做上传较少

websocket上传存在几个问题：

- 一般对于现有的上传服务，服务端需要单独开发接口
- 同样无法获得上传的进度信息（变通方式：必须使用分片来模拟进度）

# 5. XHR2上传

XHR即我们常说的ajax(Asynchronous JavaScript and XML)

XHR的特点：

- 可以设置HTTP请求的时限。
- 可以获取服务器端的（或向服务端发送）二进制数据。
- 可以使用FormData对象管理表单数据。
- 可以上传文件。xhr.upload(upload = XMLHttpRequestUpload)
- 可以获得数据传输的进度信息, xhr.upload.onprogess。
- 可以请求不同域名下的数据（跨域请求）。

```
function uploadFile(file){
   var xhr = new XMLHttpRequest();
   xhr.open('POST','/upload',true);
   var formData = new FormData();
   xhr.upload.onprogress = function(data){
      var per = Math.ceil((data.loaded/data.total)*100);
      //$('#'+file.uid+' .progress-bar').css('width',per+'%');
   }
   xhr.onreadystatechange = function() {
       if (xhr.readyState == 4 && xhr.status == 200) {
         // Every thing ok, file uploaded
           var res = JSON.parse(xhr.responseText);
           if(res.code ==200){
               // upload success
           }
       }
   };
   formData.append("upload_file", file);
   formData.append("filename",file.name);
   xhr.send(formData);
}
```

xhr2在结合H5的其他特性，可以实现上述flash上传的所以功能外，还可以实现拖拽上传功能。 
由于诸多HTML5特性（Blob ,xhr2,FileReader,ArrayBuffer等）在IE10+中才有效， 
所以xhr2上传更适合在chrome，firefox等高版本的浏览器或和移动端使用。
