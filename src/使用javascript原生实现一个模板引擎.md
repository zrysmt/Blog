---
title: 使用javascript原生实现一个模板引擎
tags: 
- FE
- javascript
- 模板引擎
- js原生实现库
categories: 前端技术
---
模板引擎分为**前端**和**后端**的，前端常用的模板引擎如artTemplate,[juicer](http://www.juicer.name/docs/docs_zh_cn.html)渲染是在客户端完成的；后端的模板引擎如基于PHP的[smarty](http://www.smarty.net/docs/zh_CN/),渲染是服务器完成的。

前两天看到一篇博客挺好的是用了不到20行代码实现一个前端的模板引擎，感觉挺有趣的，今天就来实现下

# 1.简单的例子
**逻辑**

```javascript
var tplEngine = function(tpl, data) {
        var re = /<%([^%>]+)?%>/g;
        while (match = re.exec(tpl)) {
            tpl = tpl.replace(match[0], data[match[1]]);
        }
        return tpl;
};
```
就是把`<%name%>`替换成`data.name`即可
**测试**
```javascript
var template1 = '<p>Hello, my name is <%name%>. I\'m <%age%> years old.</p>';
console.log(tplEngine(template1, {
    name: "Tom",
    age: 29
}));
```
# 2. data属性复杂点

```javascript
    var tplEngine = function(tpl, data) {
        var re = /<%([^%>]+)?%>/g;
        var code = 'var r=[];\n',
            cursor = 0;//辅助变量
        var add = function(line, js) {//针对变量还是普通的片段分别处理
            js ? code += 'r.push(' + line + ');\n' :
                code += 'r.push("' + line.replace(/"/g, '\\"') + '");\n';
        };
        while (match = re.exec(tpl)) {
            add(tpl.slice(cursor, match.index));
            add("this."+match[1],true);//要替换的变量
            cursor = match.index + match[0].length;
        }
        add(tpl.substr(cursor, tpl.length - cursor));
        code += 'return r.join("");'; // <-- return the result
        console.info(code);

        return new Function(code.replace(/[\r\t\n]/g,'')).apply(data);
    };
```
我们研究下`new Function`
**构造函数**
```
new Function ([arg1[, arg2[, ...argN]],] functionBody)
```
argN是传入的参数，当然可以省略
函数体是`code.replace(/[\r\t\n]/g,'')`，apply将函数体的上下文环境（this）指向了data
测试

```javascript
var template2 = '<p>Hello, my name is <%name%>. I\'m <%profile.age%> years old.</p>';
    console.log(tplEngine(template2, {
        name: "Kim",
        profile: {
            age: 29
        }
}));
```

# 3.加入for if循环和判断语句
按照上面的测试

```javascript
var template3 =
        'My skills:' +
        '<%for(var index in this.skills) {%>' +
        '<a href="#"><%skills[index]%></a>' +
        '<%}%>';
    console.log(tplEngine(template3, {
        skills: ["js", "html", "css"]
}));
```
报错：
```
Uncaught SyntaxError: Unexpected token for
```
打印结果`r.push(for(var index in this.skills) {);`是有问题的。
```
var r=[];
r.push("My skills:");
r.push(for(var index in this.skills) {);
r.push("<a href=\"#\">");
r.push(this.skills[index]);
r.push("</a>");
r.push(this.});
r.push("");
return r.join("");
```
修改

```javascript
var tplEngine = function(tpl, data) {
        var re = /<%([^%>]+)?%>/g,
            re2 = /(^( )?(if|for|else|switch|case|break|{|}))(.*)?/g;
        var code = 'var r=[];\n',
            cursor = 0;
        var add = function(line, js) {
            js ? code += line.match(re2) ? line + '\n' : 'r.push(' + line + ');\n' :
                code += 'r.push("' + line.replace(/"/g, '\\"') + '");\n';
        };
        while (match = re.exec(tpl)) {
            add(tpl.slice(cursor, match.index));
            re2.test(match[1]) ? add(match[1], true) : add("this." + match[1], true);
            cursor = match.index + match[0].length;
        }
        add(tpl.substr(cursor, tpl.length - cursor));
        code += 'return r.join("");'; 
        console.info(code);

        return new Function(code.replace(/[\r\t\n]/g, '')).apply(data);
};
```

我们可以打印`code`看看
```
code+='console.log(r)；\n';
```
```
["My skills:", "<a href="#">", "js", "</a>", "<a href="#">", "html", "</a>", "<a href="#">", "css", "</a>", ""]
```
最终的结果
```
var r=[];
r.push("My skills:");
for(var index in this.skills) {
r.push("<a href=\"#\">");
r.push(this.skills[index]);
r.push("</a>");
}
r.push("");
console.log(r);
return r.join("");
```
解析结果
```
My skills:<a href="#">js</a><a href="#">html</a><a href="#">css</a>
```

参考阅读：
- [只有20行Javascript代码！手把手教你写一个页面模板引擎](http://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551309&idx=1&sn=93bf90d6f2c63fea0e5f3ec488a1431f&chksm=8025a18cb752289af6ad341cdd20daa4b0d0dc6fd5e1f8cd6d8910afa8ee1b6400b0503ad382&mpshare=1&scene=1&srcid=0926c4H6Se5FCKfQBRO2uDjd#rd)
