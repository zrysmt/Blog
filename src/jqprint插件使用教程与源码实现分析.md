---
title: jqprint插件使用教程与源码实现分析
tags: 
- FE
- javascript
- 插件
- 实践
categories: 前端技术
---
**jqprint**是可以实现局部打印的一个轻量型的，基于jquery的js插件。
本博客从jqprint的使用教程和源码解析两个方面介绍这个小插件。
# 1.jqprint使用教程全解
## 1.1 jqrint基本使用教程

- 1.引入jquery和jqprint

```javascript
<script language="javascript" src="jquery-1.4.4.min.js"></script>
<script language="javascript" src="jquery.jqprint-0.3.js"></script>
```
- 2.html文件

```html
<div id="table1">
    <table>
        <tr>
            <td>test</td>
            <td>test</td>
        </tr>
    </table>
</div>
```
- 3.js文件

```javascript
$("#table1").jqprint();
```
或者可以设置options

```javascript
$("#table1").jqprint({
     debug: false, //如果是true则可以显示iframe查看效果
     //（iframe默认高和宽都很小，可以再源码中调大），默认是false
     importCSS: true, //true表示引进原来的页面的css，默认是true。
     //（如果是true，先会找$("link[media=print]")，若没有会去找$("link")中的css文件）
     printContainer: true, //表示如果原来选择的对象必须被纳入打印
     //（注意：设置为false可能会打破你的CSS规则）。
     operaSupport: true//表示如果插件也必须支持歌opera浏览器，在这种情况下，
     //它提供了建立一个临时的打印选项卡。默认是true
});
```

## 1.2 jqprint打印的div中有input或者textarea等
div中有input或者textarea等DOM的时候，直接用上面的方法是不能正确打印的（不会显示），所以我们就需要另外的途径解决问题。
**思考解决方案：**打印的时候重新生成一个html片段，打印结束的时候隐藏。
- 生成的html片段我们可以写在原来的html中，设置其不可见即可。
- 或者用模板引擎 demo如下,使用juicer前端渲染模板引擎。

```javascript
var data = {
   tab1: $('.tab-td1 input').val(),
};
var tpl = require('document/printTable.tpl');
var html = juicer(tpl, data);
$('.hidden-print-layer').html(html);
//原来html中有div.hidden-print-layer
docCommon.printTable("table1", "", "none", function() {
   $('#table1').find('th').css('font-weight', 'bold');
   //开始时候回掉函数
}, function() {
   $('#table1').css('display', 'none');
   //结束后回掉函数
});

function printTable(id, text, hide, codebf, codeaft) {
    var titleDiv = document.createElement('div');
    titleDiv.innerHTML = text;
    titleDiv.id = 'print_title';
    titleDiv.style.position = "relative";
    document.getElementById(id).appendChild(titleDiv);
    if (codebf) codebf();
    $("#" + id).jqprint({
       debug: false,
       importCSS: true, 
       printContainer: true,。
       operaSupport: true
    });
    document.getElementById(id).removeChild(titleDiv);
    if (codeaft) codeaft();
},
```
注意在`printTable.tpl`中，juicer变量是个DOM字符串时候，一定要用两个`$$`，如`$${tab1}`，可以避免模板引擎转义。对于其他模板引擎也是这样。
## 1.3 jqprint如何引入外部css

```css
<link media="print" rel="stylesheet" type="text/css" href="print.css">
```
# 2.源码分析
## 2.1 本质原理
最核心的一句是

```javascript
ifrm.print();//只打印iframe里面的内容
```
iframe.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>iframe Demo</title>
    <link rel="stylesheet" href="iframe.css">
</head>
<body>
    <html>
    <body>
        <p>这是iframe外面的文字</p>
        <iframe id="myIframe">
            <div>A</div>
            <div>B</div>
            <div>C</div>
        </iframe>
        <script>
            var ifrm = document.getElementById('myIframe');
            ifrm = ifrm.contentWindow || ifrm.contentDocument.document || ifrm.contentDocument;
            ifrm.document.open();
            ifrm.document.write('Hello World!');
            ifrm.document.write("<link type='text/css' rel='stylesheet' href='iframe.css'>");
            //将css作用到iframe上，没有这句css不会作用到iframe上
            ifrm.document.close();
            ifrm.print();//只打印iframe里面的内容
        </script>
    </body>
    </html>
</body>
</html>
```
iframe.css
```css
*{color: blue;}
```
## 2.2注释过的源码

```javascript
(function($) {
    var opt;
    var matched, browser;
    /*浏览器检测方法*/
    jQuery.uaMatch = function(ua) {
        ua = ua.toLowerCase();

        var match = /(chrome)[ \/]([\w.]+)/.exec(ua) ||
            /(webkit)[ \/]([\w.]+)/.exec(ua) ||
            /(opera)(?:.*version|)[ \/]([\w.]+)/.exec(ua) ||
            /(msie) ([\w.]+)/.exec(ua) ||
            ua.indexOf("compatible") < 0 && /(mozilla)(?:.*? rv:([\w.]+)|)/.exec(ua) || [];

        return {
            browser: match[1] || "",
            version: match[2] || "0"
        };
    };

    matched = jQuery.uaMatch(navigator.userAgent);
    browser = {};

    if (matched.browser) {
        browser[matched.browser] = true;
        browser.version = matched.version;
    }

    // Chrome is Webkit, but Webkit is also Safari.
    if (browser.chrome) {
        browser.webkit = true;
    } else if (browser.webkit) {
        browser.safari = true;
    }

    jQuery.browser = browser;
    $.fn.jqprint = function(options) {
            opt = $.extend({}, $.fn.jqprint.defaults, options);

            var $element = (this instanceof jQuery) ? this : $(this);

            if (opt.operaSupport && $.browser.opera) //opera浏览器支持
            {
                var tab = window.open("", "jqPrint-preview");
                tab.document.open();

                var doc = tab.document;
            } else {
                var $iframe = $("<iframe  />");

                if (!opt.debug) { $iframe.css({ position: "absolute", width: "0px", height: "0px", left: "-600px", top: "-600px" }); }

                $iframe.appendTo("body");
                var doc = $iframe[0].contentWindow.document; //通过iframe获得document
            }

            if (opt.importCSS) //引入打印CSS的话
            {
                if ($("link[media=print]").length > 0) { //找media=print link引进来的css文件
                    $("link[media=print]").each(function() {
                        doc.write("<link type='text/css' rel='stylesheet' href='" + $(this).attr("href") + "' media='print' />");
                        //直接引进来没作用，必须执行上面这一句
                    });
                } else {
                    $("link").each(function() {
                        doc.write("<link type='text/css' rel='stylesheet' href='" + $(this).attr("href") + "' />");
                    });
                }
            }
            //true表示如果原来选择的对象必须被纳入打印
            //false 可能会打破你的CSS规则
            if (opt.printContainer) { doc.write($element.outer()); } else { $element.each(function() { doc.write($(this).html()); }); }

            doc.close();

            (opt.operaSupport && $.browser.opera ? tab : $iframe[0].contentWindow).focus(); //获得焦点
            setTimeout(function() {
                (opt.operaSupport && $.browser.opera ? tab : $iframe[0].contentWindow).print();//打印
                if (tab) { tab.close(); } }, 1000);
        }
        //默认的配置
    $.fn.jqprint.defaults = {
        debug: false,
        importCSS: true,
        printContainer: true,
        operaSupport: true
    };

    // Thanks to 9__, found at http://users.livejournal.com/9__/380664.html
    jQuery.fn.outer = function() {
        return $($('<div></div>').html(this.clone())).html();
        //将DOM放进iframe中
    }
})(jQuery);
```


**参考阅读：**
- [w3school--iframe](http://www.w3school.com.cn/jsref/dom_obj_iframe.asp)
