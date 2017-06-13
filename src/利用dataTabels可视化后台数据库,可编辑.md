---
title: 利用dataTabels可视化后台数据库,可编辑
tags: 
- FE
- javascript
- js库
categories: 前端技术
---
# 1.基本描述
## 1.1 功能
基本功能如下所示
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/datatables1.png)
修改模块，鼠标单击要修改的内容，输入文字即可；
或者选择修改的行（每行第一列 多选框），单击左上角的Edit，Delete按钮，利用弹出窗进行修改，或者删除。可以进行批量操作。
## 1.2 运行
源码放在[github](https://github.com/zrysmt/editService)里面，配置好editor/php/config.php 中的数据库信息，在php环境中即可运行。
数据库信息：
```
"user" => "",       // Database user name
"pass" => "",       // Database password
"host" => "",       // Database host
"port" => "",       // Database connection port (can be left empty for default)
"db"   => "",       // Database name
```
# 2.Demo代码
html和css
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>修改房源信息</title>
    <!-- DataTables CSS -->
    <link rel="stylesheet" type="text/css" href="http://cdn.datatables.net/1.10.7/css/jquery.dataTables.css">
    <link rel="stylesheet" href="https://cdn.datatables.net/buttons/1.2.1/css/buttons.dataTables.min.css">
    <link rel="stylesheet" href="https://cdn.datatables.net/select/1.2.0/css/select.dataTables.min.css">
    <link rel="stylesheet" href="editor/editor.dataTables.min.css">
    <style type="text/css">
    body {
        margin: 0;
        padding: 0;
        font-family: "Microsoft YaHei"
    }
    /*dataTable*/ 
    .maintenance {
        background-color: #fff;
        width: 91%;
        padding: 15px;
        margin: 0 auto;
        margin-top: 10px;
        min-height: 460px;
    }
    #myTable {
        width: 100%;
    }
    #myTable th,
    #myTable tr {
        font-size: 14px;
        text-align: center;
    }
    .dt-buttons {
        margin-right: 20px;
        margin-top: -5px;
    }
    </style>
</head>

<body>
    <div class='maintenance'>
        <table id="myTable" class="display">
        </table>
    </div>
    <!-- jQuery -->
    <script type="text/javascript" charset="utf8" src="http://code.jquery.com/jquery-1.10.2.min.js"></script>
    <!-- DataTables -->
    <script type="text/javascript" charset="utf8" src="http://cdn.datatables.net/1.10.7/js/jquery.dataTables.js"></script>
    <script type="text/javascript" src="https://cdn.datatables.net/buttons/1.2.1/js/dataTables.buttons.min.js"></script>
    <script type="text/javascript" src="https://cdn.datatables.net/select/1.2.0/js/dataTables.select.min.js"></script>
    <script type="text/javascript" src="editor/dataTables.editor.js"></script>
</body>
</html>
```
注意要使用的dataTables和它的扩展库的路径。

js文件
```javascript
    var editor;
    $(document).ready(function() {
        var thName = [{
            data: null,
            defaultContent: '',
            className: 'select-checkbox',
            orderable: false
        }, {
            title: "序号",
            data: "id"
        }, {
            title: "省",
            data: "province"
        }, {
            title: "城市",
            data: "city"
        }, {
            title: "区",
            data: "district",
            editField: "district"
        }, {
            title: "镇",
            data: "town",
            editField: "town"
        }, {
            title: "地址",
            data: "address",
            editField: "address"
        }, {
            title: "出租信息",
            data: "rent",
            editField: "rent"
        }, {
            title: "位置",
            data: "location",
            editField: "location"
        }, {
            title: "面积",
            data: "area",
            editField: "area"
        }, {
            title: "价格",
            data: "sellingPrice",
            editField: "sellingPrice"
        }];

        editor = new $.fn.dataTable.Editor({
            ajax: "editDB.php",
            table: "#myTable",
            // idSrc: "id",
            fields: [{
                label: "District:",
                name: "district"
            }, {
                label: "Town:",
                name: "town"
            }, {
                label: "Address:",
                name: "address"
            }, {
                label: "Rent:",
                name: "rent"
            }, {
                label: "Location",
                name: "location"
            }, {
                label: "Area:",
                name: "area",
                // type: "select"
            }, {
                label: 'SellingPrice',
                name: "sellingPrice"
            }]
        });

        // Activate an inline edit on click of a table cell
        $('#myTable').on('click', 'tbody td:not(:first-child)', function(e) {
            editor.inline(this);
        });
        var myTbl = $('#myTable').DataTable({
            ajax: "editDB.php",
            //当处理大数据时，延迟渲染数据，有效提高Datatables处理能力
            deferRender: true,
            // destory:true,
            language: {
                "sProcessing": "处理中...",
                "sLengthMenu": "显示 _MENU_ 项结果",
                "sZeroRecords": "没有匹配结果",
                "sInfo": "显示第 _START_ 至 _END_ 项结果，共 _TOTAL_ 项",
                "sInfoEmpty": "显示第 0 至 0 项结果，共 0 项",
                "sInfoFiltered": "(由 _MAX_ 项结果过滤)",
                "sInfoPostFix": "",
                "sSearch": "搜索:",
                "sUrl": "",
                "sEmptyTable": "表中数据为空",
                "sLoadingRecords": "载入中...",
                "sInfoThousands": ",",
                "oPaginate": {
                    "sFirst": "首页",
                    "sPrevious": "上页",
                    "sNext": "下页",
                    "sLast": "末页"
                },
                "oAria": {
                    "sSortAscending": ": 以升序排列此列",
                    "sSortDescending": ": 以降序排列此列"
                }
            },
            dom: 'Bflrtip', //B指的是button
            columns: thName,
            // serverSide:true,//服务器端渲染
            select: {
                style: 'os',
                selector: 'td:first-child'
            },
            buttons: [/*{
                extend: "create",
                editor: editor
            },*/ {
                extend: "edit",
                editor: editor
            }, {
                extend: "remove",
                editor: editor
            }]
        });

        myTbl.select();//官网上例子没这个是错误的
    });
```
注意这一句
```
myTbl.select();//官网上例子没这个是错误的
```
php文件
```php
<?php 
    include("editor/php/DataTables.php");

    // Alias Editor classes so they are easy to use
use
    DataTables\Editor,
    DataTables\Editor\Field,
    DataTables\Editor\Format,
    DataTables\Editor\Mjoin,
    DataTables\Editor\Upload,
    DataTables\Editor\Validate;

    //第二个参数：表名
   Editor::inst( $editDb, '0test' )
    ->fields(
        Field::inst( 'id' )->validator( 'Validate::notEmpty' ),
        Field::inst( 'province' )->validator( 'Validate::notEmpty' ),
        Field::inst( 'city' )->validator( 'Validate::notEmpty' ),
        Field::inst( 'district' )->validator( 'Validate::notEmpty' ),
        Field::inst( 'town' )->validator( 'Validate::notEmpty' ),
        Field::inst( 'address' )->validator( 'Validate::notEmpty' ),
        Field::inst( 'rent' ),
        Field::inst( 'location' ),
        Field::inst( 'area' )
            ->validator( 'Validate::numeric' )
            ->setFormatter( 'Format::ifEmpty', null ),
        Field::inst( 'sellingPrice' )
            ->validator( 'Validate::numeric' )
            ->setFormatter( 'Format::ifEmpty', null )
    )
    ->process( $_POST )
    ->json();

?>
```
# 3.新增功能：记录每条记录的修改时间
![](https://raw.githubusercontent.com/zrysmt/mdPics/master/datatables2.png)
最后一个字段记录更新时间
```javascript
//获取当前格式化时间
function formatDateTime() {
    var time = new Date();
    var y = time.getFullYear();
    var m = time.getMonth() + 1;
    var d = time.getDate();
    m = parseInt(m) < 10 ? '0' + m : m;
    d = parseInt(d) < 10 ? '0' + d : d;
    var h = time.getHours();
    var min = time.getMinutes();
    var s = time.getSeconds();
    var ms = time.getMilliseconds();
    var msi = parseInt(ms);
    h = parseInt(h) < 10 ? '0' + h : h;
    min = parseInt(min) < 10 ? '0' + min : min;
    s = parseInt(s) < 10 ? '0' + s : s;
    
    return '' + y + '-' + m + '-' + d + ' ' + h + ':' + min + ':' + s ;
}
```
```javascript
editor.on('preSubmit', function(e, json, data) {
    var time = formatDateTime();
    for (var rId in json.data) {
        json.data[rId].updateDate = time;
    }
});
```
```php
//php文件增加一个字段
Field::inst( 'updateDate' )
```
在提交之前，修改要提交的json的修改时间字段
for-in循环是为了能够支持批量操作。









