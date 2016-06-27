---
title: File Input多次添加文件，动态删除文件，用来实现上传等操作
tags: FE
categories: 前端技术
---
### 1.需求图示
![实现](https://raw.githubusercontent.com/zrysmt/mdPics/master/file%20input.png)

### 2.按图索骥  
   - 添加 实际上，添加附件就是`<input type="file" id="myFile">`的控件，`var fileList = getElementById(myFile).files`就可以得到选择的文件的FileList对象，这个对象是类数组的对象(含义有点像函数参数arguments)。记住这一点很重要。

   - 显示 下面的显示文件名的面板根据上传的文件名`name`显示

### 3.刨根问底
- FileList类数组对象
   `console.log(fileList)`打印出来的结果显示：   
   ``` bash  
  	FileList  
		0:File  
		 lastModified:1446204650848  
		 lastModifiedDate:Fri Oct 30 2015 19:30:50 GMT+0800 (中国标准时间)   
		 name:"CCGIS.png"    
		 size:809542   
		 type:"image/png"   
		 webkitRelativePath:""   
	    __proto__:File   
	   length:1  
	 __proto__:FileList  
   ```
   思考：我们只需要能动态修改fileList即可，第一想法是将它转化为数组进行操作。
   `` files = Array.prototype.slice.call(files); ``

### 4.付诸行动
动手编程吧：  
	html很简单，省略  
	逻辑代码
``` javascript
	        var fileInput = document.getElementById('myFile');
            var files = fileInput.files; //filelist

            $('#myFile').on('change', function(event) {

                files = fileInput.files; //应该重新获取
                console.log(files);
                
                files = Array.prototype.slice.call(files); //全部转化为数组
                fileLists = fileLists.concat(files);
                //显示文件名面板
                if (files.length !== 0) {
                    var html = '';
                    for (var i = 0; i < files.length; i++) {
                        html += "<p>" + files[i].name + "&nbsp&nbsp<img class='icon-remove'></p>";
                    }
                    $('.upfile-list-mes').append(html);
                }
            });

           /*点击叉号可以删除要上传的文件*/
            $('.upfile-list-mes').on('click', '.icon-remove', function(event) {
                var ind = $(this).parent().index();
                $(this).parent().css('display', 'none');
                fileLists.splice(ind, 1);//修改fileLists
                console.log(fileLists);
            });
```

