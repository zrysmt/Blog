---
title: 一步一步DIY zepto库，研究zepto源码4 -- ajax模块
tags:    
- FE
- zepto
- 源码
- js原生实现库    
categories: 前端技术
---
上面的博文介绍的都是源码src下的基础模块`zepto.js`文件和事件模块`event.js`，下面接着看另外一个独立的模块--ajax模块`ajax.js`

> 代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，对应文件夹v0.4.1。
> https://github.com/zrysmt/DIY-zepto

# 1.ajax的过程
- 当global: true时。在Ajax请求生命周期内，以下这些事件将被触发。
- ajaxStart (global)：如果没有其他Ajax请求当前活跃将会被触发。
- ajaxBeforeSend (data: xhr, options)：再发送请求前，可以被取消。
- ajaxSend (data: xhr, options)：像 ajaxBeforeSend，但不能取消。
- ajaxSuccess (data: xhr, options, data)：当返回成功时。
- ajaxError (data: xhr, options, error)：当有错误时。
- ajaxComplete (data: xhr, options)：请求已经完成后，无论请求是成功或者失败。
- ajaxStop (global)：如果这是最后一个活跃着的Ajax请求，将会被触发。

下面我们就首先来看这些过程的源码：
- 实现逻辑函数：

```javascript
// trigger a custom event and return false if it was cancelled
function triggerAndReturn(context, eventName, data) {
    var event = $.Event(eventName); //包装成事件
    $(context).trigger(event, data); //触发
    return !event.isDefaultPrevented();
}
// trigger an Ajax "global" event
function triggerGlobal(settings, context, eventName, data) {
    if (settings.global) return triggerAndReturn(context || document, eventName, data);
}
// Number of active Ajax requests
// 发送中的ajax请求个数
$.active = 0;
//如果没有其他Ajax请求当前活跃将会被触发
```
-  `ajaxStart`

```javascript
//如果没有其他Ajax请求当前活跃将会被触发
function ajaxStart(settings) {
   if (settings.global && $.active++ === 0) triggerGlobal(settings, null, 'ajaxStart');
}
```
- `ajaxBeforeSend`

```javascript
// triggers an extra global event "ajaxBeforeSend" that's like "ajaxSend" but cancelable
// 触发选项中beforeSend回调函数和触发ajaxBeforeSend事件
// 上述的两步中的回调函数中返回false可以停止发送ajax请求，否则就触发ajaxSend事件
function ajaxBeforeSend(xhr, settings) {
    var context = settings.context;
    if (settings.beforeSend.call(context, xhr, settings) === false ||
        triggerGlobal(settings, context, 'ajaxBeforeSend', [xhr, settings]) === false)
        return false;

    triggerGlobal(settings, context, 'ajaxSend', [xhr, settings]);
}
```
- `ajaxSuccess`

```javascript
function ajaxSuccess(data, xhr, settings, deferred) {
    var context = settings.context,
        status = 'success';
    settings.success.call(context, data, status, xhr);
    if (deferred) deferred.resolveWith(context, [data, status, xhr]);
    triggerGlobal(settings, context, 'ajaxSuccess', [xhr, settings, data]);
    ajaxComplete(status, xhr, settings);
}
```
- `ajaxError`

```javascript
// type: "timeout", "error", "abort", "parsererror"
function ajaxError(error, type, xhr, settings, deferred) {
    var context = settings.context;
    settings.error.call(context, xhr, type, error);
    if (deferred) deferred.rejectWith(context, [xhr, type, error]);
    triggerGlobal(settings, context, 'ajaxError', [xhr, settings, error || type]);
    ajaxComplete(type, xhr, settings);
}
```
- `ajaxComplete`

```javascript
// status: "success", "notmodified", "error", "timeout", "abort", "parsererror"
function ajaxComplete(status, xhr, settings) {
    var context = settings.context;
    settings.complete.call(context, xhr, status);
    triggerGlobal(settings, context, 'ajaxComplete', [xhr, settings]);
    ajaxStop(settings);
}

```
- `ajaxStop`

```javascript
// 所有ajax请求都完成后才触发
function ajaxStop(settings) {
    if (settings.global && !(--$.active)) triggerGlobal(settings, null, 'ajaxStop');
}
```
我们注意到，我们只是自定义了ajaxXXX事件，并没有实际的意义，这时候我们就需要将这些事件的逻辑实现。这部分逻辑放在了`$.ajax`中，在适当的时候触发事件，这些很值得我们去思考的它的巧妙。

# 2.`$.ajax`

## 2.1 全局变量,工具函数定义：

```javascript
var jsonpID = +new Date(),
    document = window.document,
    key,
    name,
    rscript = /<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, //用来除掉html代码中的<script>标签
    scriptTypeRE = /^(?:text|application)\/javascript/i,
    xmlTypeRE = /^(?:text|application)\/xml/i, //用来判断是不是js的mime
    jsonType = 'application/json',
    htmlType = 'text/html',
    blankRE = /^\s*$/,
    originAnchor = document.createElement('a');
originAnchor.href = window.location.href;

// Empty function, used as default callback
// 空函数，被用作默认的回调函数
function empty() {}
function mimeToDataType(mime) {
    if (mime) mime = mime.split(';', 2)[0];
    return mime && (mime == htmlType ? 'html' :
        mime == jsonType ? 'json' :
        scriptTypeRE.test(mime) ? 'script' :
        xmlTypeRE.test(mime) && 'xml') || 'text';
}
//把参数添加到url上
function appendQuery(url, query) {
    if (query == '') return url;
    return (url + '&' + query).replace(/[&?]{1,2}/, '?');
    //将&、&&、&?、?、?、&?&? 转化为 ?
}
```
## 2.2 `$.ajaxSetting`

```javascript
$.ajaxSettings = {
    type: 'GET',
    beforeSend: empty,
    success: empty,
    error: empty,
    complete: empty,
    context: null,
    global: true,
    xhr: function() {
        return new window.XMLHttpRequest();
    },
    accepts: {
        script: 'text/javascript, application/javascript, application/x-javascript',
        json: jsonType,
        xml: 'application/xml, text/xml',
        html: htmlType,
        text: 'text/plain'
    },
    crossDomain: false,
    timeout: 0,
    processData: true,
    cache: true,
    dataFilter: empty
};
```
## 2.3 序列化
```javascript
// 列化data参数，并且如果是GET方法的话把参数添加到url参数上
function serializeData(options) {
    //options.data是个对象
    if (options.processData && options.data && $.type(options.data) != "string")
        console.info(options.data);
    options.data = $.param(options.data, options.traditional);

    // 请求方法为GET，data参数添加到url上
    if (options.data && (!options.type || options.type.toUpperCase() == 'GET' || 'jsonp' == options.dataType))
        options.url = appendQuery(options.url, options.data), options.data = undefined;
}
```
```javascript
var escape = encodeURIComponent;
//序列化
//在Ajax post请求中将用作提交的表单元素的值编译成 URL编码的 字符串
function serialize(params, obj, traditional, scope) {
    var type, array = $.isArray(obj),
        hash = $.isPlainObject(obj);
    // debugger;

    $.each(obj, function(key, value) {
        type = $.type(value);

        if (scope) {
            key = traditional ? scope :
                scope + '[' + (hash || type == 'object' || type == 'array' ? key : '') + ']';
        }
        // handle data in serializeArray() format
        if (!scope && array) { //obj是个数组
            params.add(value.name, value.value);
        }
        // obj的value是个数组/对象
        else if (type == "array" || (!traditional && type == "object")) {
            serialize(params, value, traditional, key);
        } else {
            params.add(key, value);
        }
    });
}
$.param = function(obj, traditional) {
    var params = [];
    //serialize函数使用add
    params.add = function(key, value) {
        if ($.isFunction(value)) value = value();
        if (value == null) value = "";
        this.push(escape(key) + '=' + escape(value));
    };
    serialize(params, obj, traditional); //处理obj
    return params.join('&').replace(/%20/g, '+');
};
```
## 2.4 [XMLHttpRequest解释](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
常用方法：

| 函数/属性  |作用   |
| ------------ | ------------ |
|setRequestHeader()|给指定的HTTP请求头赋值.在这之前,你必须确认已经调用 [`open()`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest#open) 方法打开了一个url|
|overrideMimeType() |重写由服务器返回的MIME type   |
|onreadystatechange   |readyState属性改变时会调用它   |
|open()   |初始化一个请求   |
|send()   |发送请求. 如果该请求是异步模式(默认),该方法会立刻返回. 相反,如果请求是同步模式,则直到请求的响应完全接受以后,该方法才会返回   |

**readyState的状态：**

| 值 | 状态 | 描述 |
| ------------ | ------------ |
| `0` | `UNSENT `(未打开) | `open()`方法还未被调用. |
| `1` | `OPENED`  (未发送) | `send()`方法还未被调用. |
| `2` | `HEADERS_RECEIVED (已获取响应头)` | `send()`方法已经被调用, 响应头和响应状态已经返回. |
| `3` | `LOADING (正在下载响应体)` | 响应体下载中; `responseText`中已经获取了部分数据. |
| `4` | `DONE (请求完成)` | 整个请求过程已经完毕. |
## 2.5 `$.ajax`实现

```javascript
$.ajax = function(options) {
    var settings = $.extend({}, options || {}),
        deferred = $.Deferred && $.Deferred(),
        urlAnchor, hashIndex;
    for (key in $.ajaxSettings)
        if (settings[key] === undefined) settings[key] = $.ajaxSettings[key];

    ajaxStart(settings); //---------------@开始
    // 如果没有传入crossDomain参数，就通过检测setting.url和网址的protocol、host是否一致判断该请求是否跨域
    if (!settings.crossDomain) {
        // 通过设置a元素的href就可以很方便的获取一个url的各组成部分
        urlAnchor = document.createElement('a');
        urlAnchor.href = settings.url;
        // cleans up URL for .href (IE only), see https://github.com/madrobby/zepto/pull/1049
        urlAnchor.href = urlAnchor.href;
        settings.crossDomain = (originAnchor.protocol + '//' + originAnchor.host) !== (urlAnchor.protocol + '//' + urlAnchor.host);
    }
    // 没有传入url参数，使用网站的网址为url参数
    // window.location.toString() 等于 window.location.href
    if (!settings.url) settings.url = window.location.toString();
    //去掉url上的hash部分
    if ((hashIndex = settings.url.indexOf('#')) > -1) settings.url = settings.url.slice(0, hashIndex);
    serializeData(settings); // 序列化data参数，并且如果是GET方法的话把参数添加到url参数上
    var dataType = settings.dataType,
        hasPlaceholder = /\?.+=\?/.test(settings.url); // 判断url参数是否包含=?
    if (hasPlaceholder) dataType = 'jsonp'; //jsonp url 举例http://www.xxx.com/xx.php?callback=?

    // 设置了cache参数为false，或者cache参数不为true而且请求数据的类型是script或jsonp，就在url上添加时间戳防止浏览器缓存
    // (cache设置为true也不一定会缓存，具体要看缓存相关的http响应首部)
    if (settings.cache === false || (
            (!options || options.cache !== true) &&
            ('script' == dataType || 'jsonp' == dataType)
        ))
        settings.url = appendQuery(settings.url, '_=' + Date.now());

    // jsonp调用$.ajaxJSONP实现
    if ('jsonp' == dataType) {
        if (!hasPlaceholder)
            settings.url = appendQuery(settings.url,
                settings.jsonp ? (settings.jsonp + '=?') : settings.jsonp === false ? '' : 'callback=?');
        return $.ajaxJSONP(settings, deferred);
    }

    // 下面代码用来设置请求的头部、相应的mime类型等
    var mime = settings.accepts[dataType],
        headers = {},
        setHeader = function(name, value) { headers[name.toLowerCase()] = [name, value]; },
        protocol = /^([\w-]+:)\/\//.test(settings.url) ? RegExp.$1 : window.location.protocol,
        xhr = settings.xhr(), //XMLHttpRequest
        nativeSetHeader = xhr.setRequestHeader,
        abortTimeout;

    if (deferred) deferred.promise(xhr);

    //不跨域
    if (!settings.crossDomain) setHeader('X-Requested-With', 'XMLHttpRequest');
    setHeader('Accept', mime || '*/*');
    if (mime = settings.mimeType || mime) {
        if (mime.indexOf(',') > -1) mime = mime.split(',', 2)[0];
        //重写由服务器返回的MIME type  注意，这个方法必须在send()之前被调用
        xhr.overrideMimeType && xhr.overrideMimeType(mime);
    }
    //设置contentType
    if (settings.contentType || (settings.contentType !== false && settings.data && settings.type.toUpperCase() != 'GET'))
        setHeader('Content-Type', settings.contentType || 'application/x-www-form-urlencoded');
    //如果配置中有对headers内容
    if (settings.headers)
        for (name in settings.headers) setHeader(name, settings.headers[name]);
    xhr.setRequestHeader = setHeader; //设置头信息

    xhr.onreadystatechange = function() {
        if (xhr.readyState == 4) { //请求完成
            xhr.onreadystatechange = empty;
            clearTimeout(abortTimeout);
            var result, error = false;
            //请求成功
            //在本地调动ajax，也就是请求url以file开头，也代表请求成功
            if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304 || (xhr.status == 0 && protocol == 'file:')) {
                dataType = dataType || mimeToDataType(settings.mimeType || xhr.getResponseHeader('content-type'));

                // 根据xhr.responseType和dataType处理返回的数据
                if (xhr.responseType == 'arraybuffer' || xhr.responseType == 'blob') {
                    result = xhr.response;
                } else {
                    result = xhr.responseText;

                    try {
                        // http://perfectionkills.com/global-eval-what-are-the-options/
                        // (1,eval)(result) 这样写还可以让result里面的代码在全局作用域里面运行
                        result = ajaxDataFilter(result, dataType, settings);
                        if (dataType == 'script') {
                            (1, eval)(result);
                        } else if (dataType == 'xml') {
                            result = xhr.responseXML;
                        } else if (dataType == 'json') {
                            result = blankRE.test(result) ? null : $.parseJSON(result);
                        }
                    } catch (e) { error = e; }

                    if (error) return ajaxError(error, 'parsererror', xhr, settings, deferred);//---------------@
                }

                ajaxSuccess(result, xhr, settings, deferred);//---------------@
            } else {
                ajaxError(xhr.statusText || null, xhr.status ? 'error' : 'abort', xhr, settings, deferred);//---------------@
            }
        }
    };
    //必须send()之前//---------------@
    if (ajaxBeforeSend(xhr, settings) === false) {
        xhr.abort();
        ajaxError(null, 'abort', xhr, settings, deferred);
        return xhr;
    }

    var async = 'async' in settings ? settings.async : true;
    /**
     * void open(
           DOMString method,
           DOMString url,
           optional boolean async,
           optional DOMString user,
           optional DOMString password
        );
     */
    xhr.open(settings.type, settings.url, async, settings.username, settings.password);

    if (settings.xhrFields)
        for (name in settings.xhrFields) xhr[name] = settings.xhrFields[name];

    for (name in headers) nativeSetHeader.apply(xhr, headers[name]);

    // 超时丢弃请求
    if (settings.timeout > 0) abortTimeout = setTimeout(function() {
        xhr.onreadystatechange = empty;
        xhr.abort();
        ajaxError(null, 'timeout', xhr, settings, deferred);
    }, settings.timeout);
    
    // avoid sending empty string (#319)
    xhr.send(settings.data ? settings.data : null);
    return xhr;
};
```
## 2.6 示例Demo
ajax读取本地的json，安装个小服务器,或者使用其他服务器，如apache，tomcat等。
```bash
npm install anywhere -g      //安装
anywhere 8860                //开启端口（任意没被占用的）为服务器使用
```
```javascript
$.ajax({
    type: 'GET',
    url: '/projects.json',
    data: {
        name: 'Hello'
    },
    dataType: 'json',
    timeout: 300,
    success: function(data) {
        console.log(data);
    },
    error: function(xhr, type) {
        alert('Ajax error!');
    }
})
```
# 3.jsonp
使用jsonp跨域的核心思想是：

```javascript
script = document.createElement('script')
script.src = options.url.replace(/\?(.+)=\?/, '?$1=' + callbackName);
document.head.appendChild(script)
```
具体实现的代码：

```javascript
$.ajaxJSONP = function(options, deferred) {
    // 没有type选项，调用$.ajax实现
    if (!('type' in options)) return $.ajax(options);

    var _callbackName = options.jsonpCallback,
        //options配置写了jsonpCallback，那么回调函数的名字就是options.jsonpCallback
        //没有就是'Zepto' + (jsonpID++)
        callbackName = ($.isFunction(_callbackName) ?
            _callbackName() : _callbackName) || ('Zepto' + (jsonpID++)),
        script = document.createElement('script'),
        originalCallback = window[callbackName],
        responseData,
        abort = function(errorType) {
            $(script).triggerHandler('error', errorType || 'abort');
        },
        xhr = { abort: abort },
        abortTimeout;

    if (deferred) deferred.promise(xhr);
    // 加载成功或者失败触发相应的回调函数
    // load error 在event.js
    $(script).on('load error', function(e, errorType) {
        clearTimeout(abortTimeout);
        // 加载成功或者失败都会移除掉添加到页面的script标签和绑定的事件
        $(script).off().remove();

        if (e.type == 'error' || !responseData) { //失败
            ajaxError(null, errorType || 'error', xhr, options, deferred);
        } else { //成功
            ajaxSuccess(responseData[0], xhr, options, deferred);
        }

        window[callbackName] = originalCallback;
        if (responseData && $.isFunction(originalCallback))
            originalCallback(responseData[0]);

        originalCallback = responseData = undefined;
    });
    // 在beforeSend回调函数或者ajaxBeforeSend事件中返回了false，取消ajax请求
    if (ajaxBeforeSend(xhr, options) === false) {
        abort('abort');
        return xhr;
    }

    window[callbackName] = function() {
        responseData = arguments;
    };
    // 参数中添加上变量名
    script.src = options.url.replace(/\?(.+)=\?/, '?$1=' + callbackName);
    document.head.appendChild(script);
    //超时处理
    if (options.timeout > 0) abortTimeout = setTimeout(function() {
        abort('timeout');
    }, options.timeout);

    return xhr;
};
```
# 4.get、post、getJSON、load 方法
篇幅所限，这里就不再单列出来了，他们都是处理好参数，调用`$.ajax`方法。详细的实现方式去我的github 下载。

> 全部代码挂在我的[github](https://github.com/zrysmt/DIY-zepto)上，本博文对应文件夹v0.4.x。
> https://github.com/zrysmt/DIY-zepto


参考阅读：
- [Zepto ajax 模块 源码分析](https://github.com/oadaM92/zepto/tree/master/oadaM92/ajax)
- [XMLHttpRequest解释--MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
