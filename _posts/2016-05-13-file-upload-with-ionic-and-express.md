---
layout: post
title:  "使用Ionic和Express实现文件上传"
date:   2016-05-13
tags: [ionic, express]
categories: Full-Stack
---

使用Ionic搭建移动App的时候，会遇到要向服务器上传图片或者文件的情况。普通的做法是可以使用表单来实现。但是，ngCordova提供了文件操作的插件，在构建App的时候可以直接使用。

### 客户端

> 注意，以下步骤的前提是在App的模块中已经注入了`ngCordova`模块。

**1、安装插件**

首先，在客户端，要安装上ngCordova所提供的文件操作的插件`File Transfer`。

使用以下命令进行安装：

{% highlight bash %}
cordova plugin add cordova-plugin-file-transfer
{% endhighlight %}

**2、实现**

然后，按照ngCordova官方文档所提供的方法，可以像下面这样来进行文件的上传：

{% highlight javascript %}
app.controller(['$cordovaFileTransfer', function ( $cordovaFileTransfer ) {

	// 一定要将代码放在事件监听中
	// 表示设备准备好了之后才能执行里面的代码
    document.addEventListener('deviceready', function () {
        var uploadOptions = new FileUploadOptions(),
            server = 'your_server_api',
            imgPath = 'your_img_path';

        // 注意此处设置的fileKey，Express服务端中也需要这个
        uploadOptions.fileKey = 'file';
        uploadOptions.fileName = imgPath.substr(imgPath.lastIndexOf('/') + 1);
        uploadOptions.mimeType = 'image/jpeg';
        uploadOptions.chunkedMode = false;

        $cordovaFileTransfer.upload( server, imgPath, uploadOptions )
            .then(function ( result ) {
                // 上传成功
            }, function ( error ) {
                // 上传失败
            }, function ( progress ) {
                // 上传进度
            });

    }, false);
}]);
{% endhighlight %}

以上为客户端的文件上传实现代码。

### 服务端

服务端采用Express来实现，要接受来自客户端的文件，需要使用[multer](https://github.com/expressjs/multer)模块。可以使用`npm install --save multer`进行安装。

multer可接受单个文件的上传和多个文件的上传。此处只涉及单个文件上传。

在服务端，首先创建一个`fileUpload.js`：

{% highlight javascript %}
/* fileUpload.js */
var multer = require('multer'),
    storage = multer.diskStorage({
        destination: function ( req, file, callback ) {
		    // 注意，此处的uploads目录是从项目的根目录开始寻找
		    // 如果没有的话，需要手动新建此文件夹
            callback(null, './uploads');
        },
        filename: function ( req, file, callback ) {

	        // multer不会自动添加文件后缀名，需要手动添加
            callback(null, file.fieldname + '-' + Date.now() + '.' + file.mimetype.split('/')[1]);
        }
    }),

    upload = multer({
        storage: storage,
        limits: 1000000
    }).single('file');

// 将文件上传封装为一个模块，以供其他地方使用
function uploadFile( req, res ) {
    upload(req, res, function ( error ) {
        if ( error ) {
            console.error(JSON.stringify(error));
            return res.end('Error uploading file.');
        }
        console.log('Success!');
        res.end('File is uploaded');
    });
}

exports.uploadFile = uploadFile;
{% endhighlight %}

然后，创建一个`router.js`，通过路由来调用文件上传。

{% highlight javascript %}
/* router.js */
var router = require('express').Router(),
    uploadPhoto = require('/modules/path/fileUpload').uploadFile;

router.all('/photo_upload', function ( req, res ) {
    uploadFile(req, res);
});
{% endhighlight %}

以上就是一个简单的使用Ionic和Express实现的文件上传。

参考自[https://codeforgeek.com/2014/11/file-uploads-using-node-js/](https://codeforgeek.com/2014/11/file-uploads-using-node-js/)

本文代码在我的Gist里也有包含：[https://gist.github.com/Erichain/911c0d4a5f31e38b744b03cd65885ce6](https://gist.github.com/Erichain/911c0d4a5f31e38b744b03cd65885ce6)

更多关于[ngCordova](http://ngcordova.com)和[multer](https://github.com/expressjs/multer)可以查看其官方文档。