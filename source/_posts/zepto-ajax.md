---
title: Zepto.js全局设置headers
date: 2017-08-10 17:53:29
tags:
- Zepto.js
- Ajax
- Laravel
categories:
- PHP
- laravel
---

### 最后总结的测试代码：

```javascript
<?php
if(!empty($_POST)){
	echo $_POST['name'];
	exit;
}
?>

<!DOCTYPE html>
<html>

<head>
    <meta name="csrf-token" content="dZhN3E5hVsgrbxr3aZiwUT1WIA0MeZbrJtK62yyK">
    <script type="text/javascript" src="http://zeptojs.com/zepto.min.js"></script>
    <script type="text/javascript">
    /*       
	// 第一种方式
    $(document).on('ajaxBeforeSend', function(e, xhr, options) {
        var token = $('meta[name="csrf-token"]').attr('content');
        if (token) {
            xhr.setRequestHeader('X-CSRF-TOKEN', token)
        }
    })
    */

    //第二种方式
    var token = $('meta[name="csrf-token"]').attr('content');
    $.ajaxSettings.headers = { 'X-CSRF-TOKEN': token };

    </script>
    <title>zeptojs测试</title>
    <button onclick="zeptojs()">ajax</button>
</head>

<body>
    <script type="text/javascript">
    function zeptojs() {
        $.ajax({
            url: "zepto.php",
            type: "post",
            data: { name: 'Zepto.js' },
            success: function(data) {
                console.log(data);
            },
        })
    }
    </script>
</body>

</html>
```
<!-- more -->
### Zepto简介

　　Zepto的设计目的是提供Zepto是一个轻量级的针对现代高级浏览器的JavaScript库， 它与jquery有着类似的api。 如果你会用jquery，那么你也会用zepto。
　　Zepto的设计目的是提供 jQuery 的类似的API，但并不是100%覆盖jQuery。Zepto设计的目的是有一个5-10k的通用库、下载并快速执行、有一个熟悉通用的API，所以你能把你主要的精力放到应用开发上。

#### 浏览器支持

  **初级 (100% 支持)**

  - Safari 6+ (Mac)

  - Chrome 30+ (Windows, Mac, Android, iOS, Linux, Chrome OS)

  - Firefox 24+ (Windows, Mac, Android, Linux, Firefox OS)

  - iOS 5+ Safari

  - Android 2.3+ Browser

  - Internet Explorer 10+ (Windows, Windows Phone)

#### 次要目标（完全或大部分支持）
  - iOS 3+ Safari

  - Chrome <30

  - Firefox 4+

  - Safari <6

  - Android Browser 2.2

  - Opera 10+

  - webOS 1.4.5+ Browser

  - BlackBerry Tablet OS 1.0.7+ Browser

  - Amazon Silk 1.0+

  - Other WebKit-based browsers/runtimes

　　需要注意的是Zepto的一些可选功能是专门针对移动端浏览器的；因为它的最初目标在移动端提供一个精简的类似jquery的js库。
　　在浏览器上(Safari、Chrome和Firefox)上开发页面应用或者构建基于html的web-view本地应用，你如PhoneGap，使用Zepto是一个不错的选择。
　　总之，Zepto希望在所有的现代浏览器中作为一种基础环境来使用。Zepto不支持旧版本的Internet Explorer浏览器(<10)。

### Zepto.js的 Ajax 事件

当global: true时。在Ajax请求生命周期内，以下这些事件将被触发。

- ajaxStart (global)：如果没有其他Ajax请求当前活跃将会被触发。

- ajaxBeforeSend (data: xhr, options)：再发送请求前，可以被取消。

- ajaxSend (data: xhr, options)：像 ajaxBeforeSend，但不能取消。

- ajaxSuccess (data: xhr, options, data)：当返回成功时。

- ajaxError (data: xhr, options, error)：当有错误时。

- ajaxComplete (data: xhr, options)：请求已经完成后，无论请求是成功或者失败。

- ajaxStop (global)：如果这是最后一个活跃着的Ajax请求，将会被触发。

默认情况下，Ajax事件在document对象上触发。然而，如果请求的 `context` 是一个DOM节点，该事件会在此节点上触发然后再DOM中冒泡。唯一的例外是 `ajaxStart` & `ajaxStop`这两个全局事件。

```javascript
$(document).on('ajaxBeforeSend', function(e, xhr, options){
  // This gets fired for every Ajax request performed on the page.
  // The xhr object and $.ajax() options are available for editing.
  // Return false to cancel this request.
})

$.ajax({
  type: 'GET',
  url: '/projects',
  // data to be added to query string:
  data: { name: 'Zepto.js' },
  // type of data we are expecting in return:
  dataType: 'json',
  timeout: 300,
  context: $('body'),
  success: function(data){
    // Supposing this JSON payload was received:
    //   {"project": {"id": 42, "html": "<div>..." }}
    // append the HTML to context object.
    this.append(data.project.html)
  },
  error: function(xhr, type){
    alert('Ajax error!')
  }
})

// post a JSON payload:
$.ajax({
  type: 'POST',
  url: '/projects',
  // post payload:
  data: JSON.stringify({ name: 'Zepto.js' }),
  contentType: 'application/json'
})
```

- 根据以上可以实现第一种方法。

```javascript
$(document).on('ajaxBeforeSend', function(e, xhr, options){
    setting.setRequestHeader('name','value')
})
```

- 变种：

```javascript
$(document).on('ajaxSend', function(e, xhr, options){
    setting.setRequestHeader('name','value')
})
```



### $.ajaxSettings

一个包含Ajax请求的默认设置的对象。大部分的设置在 [$.ajax](http://www.runoob.com/manual/zeptojs.html#$.ajax)中已经描述。以下设置为全局非常有用：

- `timeout` (默认： `0`)：对Ajax请求设置一个非零的值指定一个默认的超时时间，以毫秒为单位。

- `global` (默认： true)：设置为false。以防止触发Ajax事件。

- `xhr` (默认：XMLHttpRequest factory)：设置为一个函数，它返回XMLHttpRequest实例(或一个兼容的对象)

- `accepts`  : 从服务器请求的MIME类型，指定

- `dataType`  值：

  - script: “text/javascript, application/javascript”

  - json: “application/json”

  - xml: “application/xml, text/xml”

  - html: “text/html”

  - text: “text/plain”

以上只列出了部分参数。查看源码：
```javascript
 $.ajaxSettings = {
    // Default type of request
    type: 'GET',
    // Callback that is executed before request
    beforeSend: empty,
    // Callback that is executed if the request succeeds
    success: empty,
    // Callback that is executed the the server drops error
    error: empty,
    // Callback that is executed on request complete (both: error and success)
    complete: empty,
    // The context for the callbacks
    context: null,
    // Whether to trigger "global" Ajax events
    global: true,
    // Transport
    xhr: function () {
      return new window.XMLHttpRequest()
    },
    // MIME types mapping
    // IIS returns Javascript as "application/x-javascript"
    accepts: {
      script: 'text/javascript, application/javascript, application/x-javascript',
      json:   jsonType,
      xml:    'application/xml, text/xml',
      html:   htmlType,
      text:   'text/plain'
    },
    // Whether the request is to another domain
    crossDomain: false,
    // Default timeout
    timeout: 0,
    // Whether data should be serialized to string
    processData: true,
    // Whether the browser should be allowed to cache GET responses
    cache: true,
    //Used to handle the raw response data of XMLHttpRequest.
    //This is a pre-filtering function to sanitize the response.
    //The sanitized response should be returned
    dataFilter: empty
  }
```
没有相关的参数可以设定 ajax请求 的 `headers` 参数。

继续搜索关键词 `“ajaxSettings”` 查找源码，可以看到以下内容：

```javascript
$.ajax = function(options){
    var settings = $.extend({}, options || {}),
        deferred = $.Deferred && $.Deferred(),
        urlAnchor, hashIndex
    for (key in $.ajaxSettings) if (settings[key] === undefined) settings[key] = $.ajaxSettings[key]

    ajaxStart(settings)
    ...
```

在执行 `$.ajax` 时会循环查找 `ajaxSettings` 的参数，当参数未定义时，会获取 `$.ajaxSettings` 相关属性的值，并使用 `ajaxStart` 方法设置参数。那么我们可以在参数中追加 `$.ajaxSettings` 属性。

第二种方法：
```javascript
$.ajaxSettings.headers = {'name' : 'value' };
```
或者：
```javascript
$.ajaxSettings = $.extend($.ajaxSettings, {
    headers: {'name' : 'value' };
});
```

