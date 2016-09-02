---
title: 利用php和ajax实现局部更新数据
tags: 
- PHP
- ajax
categories: PHP
---
基本思路就是ajax请求数据，获取成功后用jQuery更新DOM即可
下面是个很简单的实例
# 1.基本功能
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/php-ajax-1.png)
鼠标悬停在1，2,3,4的时候，下面的框中的数据会跟随改变
如在2的时候显示
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/php-ajax-2.png)
# 2.代码
**html**
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>利用ajax实现局部更新</title>
    <style>
        div{margin-left: 20px; }
        .divs div{width: 40px; height: 20px; color: white; background-color: #008B00; display: inline-block; }
        .divs div:hover{cursor: pointer; }
        .message{width: 400px; height: 100px; border: solid 2px #ccc; }
    </style>
</head>

<body>
    <div class="divs">
        <div class="cls1">1</div>
        <div class="cls2">2</div>
        <div class="cls3">3</div>
        <div class="cls4">4</div>
    </div>
    <div class="message"></div>
    <script src="http://cdn.bootcss.com/jquery/1.11.3/jquery.js"></script>
    <script type="text/javascript">
        $('.divs div').on({
            mouseenter:function(){
                var className = $(this).attr('class');
                $.ajax({
                    url: 'partRefresh.php',
                    type: 'POST',
                    data: {"className": className},
                })
                .done(function(data) {
                    console.log("success");
                    $('.message').html(data);
                })
                .fail(function() {
                    console.log("error");
                })
                .always(function() {
                    console.log("complete");
                });
                
            },
            mouseleave:function(){

            }
        });
    </script>
</body>

</html>
```
**php**
```php
<?php 
    $className = $_POST["className"];
    echo "className=<b>$className</b>";
    echo "<p>$className</p>"
 ?>
```