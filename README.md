# JF图片服务 PHP SDK

## 说明
http 请求使用 Guzzle 三方库（v5.3），具体文档可参考：[Guzzle v5.3 Doc](http://guzzle.readthedocs.io/en/5.3/)

## 安装
```
composer require jufeng/phpsdk
```
由于本项目未发布到[packagist](https://packagist.org)，因此本地安装时需要在项目的`composer.json`文件中加入如下内容：
```
"repositories": [
    {
        "type": "git",
        "url": "git@git.20hn.cn:developer/img-server-sdk.git"
    }
]
```
之后运行`composer install`即可。

## 使用方法
```php
<?php

require 'vendor/autoload.php';

// 实例化客户端
$client = new JF\FileManager('ak', 'sk', 'http://img-upload.20hn.cn/v1');
// 设置上传策略，无特殊需求可省略。参考：参考：http://git.20hn.cn/developer/img-server/wikis/api-des
// $client->setPolicy(['deadline' => time() + 3600, 'autoCompress' => 1]);
// 获取token
$token = $client->getToken();

try {
    // 文件上传
    $response = $client->upload($filePath = '/tmp/test.jpeg', $token);
    // 远程图片采集
    // $response = $client->collect('http://9.pic.pc6.com/thumb/up/2015-4/14301352575151701_600_0.jpg', $token);

    // http 请求返回内容
    echo $response->getBody();
    // http 响应状态码
    echo $response->getStatusCode();
} catch (GuzzleHttp\Exception\BadResponseException $e) {
    // 4xx 或 5xx 错误
    $response = $e->getResponse();
    echo $response->getStatusCode();
} catch (Exception $e) {
    echo $e->getMessage();
}
```

# JF图片服务 JS SDK

推荐使用的三方上传组件：[uploadify](http://www.uploadify.com/)、[plupload](http://www.plupload.com/)。相对于`uploadify`，`plupload`功能更为全面，支持html5、flash、silverlight等多种上传方式。上传组件的具体使用请参考其官方文档。

相对于传统的上传方式，使用图片服务的不同之处在于上传之前需要到服务端获取`token`，然后上传文件时带上`token`参数即可。以`uploadify`为例，代码如下所示：

```javascript
$('#file_upload').uploadify({
    'swf'      : 'uploadify.swf',
    'uploader' : '/file/upload', // 图片服务url
    'buttonText':'点击上传',
    'buttonImage':'btnbg.png',
    'multi'    : false,
    'formData' : {token: 'upload_token'}, // 从服务端获取的token值
    'onUploadSuccess': function (file, data, response) {
        // 上传成功回调
    },
});
```

`plupload`的使用方法类似：

```javascript
var uploader = new plupload.Uploader({
    runtimes : 'html5,flash,silverlight,html4',
    flash_swf_url : 'plupload-2.3.1/js/Moxie.swf',
    silverlight_xap_url : 'plupload-2.3.1/js/Moxie.xap',
    filters : {
        max_file_size : '2mb', //文件上传大小
        mime_types: [
            {title : "Image files", extensions : "jpg,jpeg,gif,png"},
        ], // 文件上传类型
    },
    drop_element : 'dropElementId', // 拖拽上传
    browse_button : 'browseButtonId', // 浏览上传
    url : 'file/upload', // 图片服务url
    multipart_params : {token: 'upload_token'} // 从服务端获取的token值
});

uploader.init();

uploader.bind('FilesAdded', function(up, files) {});
uploader.bind('UploadProgress', function(up, file) {});
uploader.bind('Error', function(up, err) {});
```