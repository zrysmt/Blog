---
title: 利用File Input控件修改name属性
tags: 
- FE
- javascript
categories: 前端技术
---
接着：【FE】File Input多次添加文件，动态删除文件，用来实现上传等操作 一文
### 1.想方设法  
我们首先查阅资料后发现 `fileList`的`name`属性是只读的[MDN][1]
修改只读属性：  
```javascript
	Object.defineProperty(fileLists[0], 'name', {
            writable: true
     });
```
这样就可以修改fileLists的name属性了，注：`writable: false`是不可逆的。

### 2.付诸行动
```javascript
	for (var i = 0; i < fileLists.length; i++) {
        var initialFileName = fileLists[i].name;
        Object.defineProperty(fileLists[i], 'name', {
               writable: true //设置属性为可写
        });
        fileLists[i].name = timestamp+'-'+initialFileName;//在名字中加入时间戳，可以是任意的字符串
        filename += fileLists[i].name + ";";
     }
     filename = filename.slice(0, -1);
```

**参考资料**  
[1]: https://developer.mozilla.org/zh-CN/docs/Web/API/File 
1. 【MDN】https://developer.mozilla.org/zh-CN/docs/Web/API/File

