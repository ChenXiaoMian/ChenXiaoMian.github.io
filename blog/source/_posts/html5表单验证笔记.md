---
title: html5表单验证笔记
date: 2018-05-30 14:29:42
categories: html5
tags: [html5,表单验证]
---

## html5中valid、invalid、required的定义

### `:required`
必须，那input不能为空的意思。

### `:valid`
有效，即当填写的内容符合要求的时候触发。

### `:invalid`
无效，即当填写的内容不符合要求的时候触发。

```css
input:required:invalid, input:focus:invalid, textarea:required:invalid, textarea:focus:invalid{box-shadow: none;}
```

## autocomplete属性和novalidate属性

`autocomplete属性`控制自动完成功能的开启和关闭。

```html
<form action="#" method="get" autocomplete="on">
  请输入：<input type="text"  name="txt" /><br/>
  <input type="submit" />
</form>
```

`novalidate属性`用于在提交表单时不对form或input进行验证。（默认浏览器行为）

## 新增input属性

1、`required属性`，用户必须填写内容才能提交，为空时提交不上。

2、`placeholder属性`，提示需要输入的内容。

3、`autofocus属性`，用于自动获取焦点。

4、`form属性`，用于设置input元素属于哪个表单。在html4中，表单中的所以元素都必须在这个表单的开始标签和结束标签之间，
而在html5中，如果要将表单开始和结束标签之外的元素归属到该表单，只需要设置`form属性`。

```html
<form action="#" method="get" id="myForm">
    常用地址：<input type="text" name="ftxt" />
    <input type="submit" />
</form>
临时地址：<input type="text" name="ltxt" form="myForm" />
```

5、`override属性`，用于重写表单元素的某些属性，在html5中，可以重写的表单属性有`formaction`、`formmethod`、`formenctype`、`formnovalidate`和`formtarget`，这些属性分别用于重写表单的action、enctype、method、novalidate和target属性。

```html
<form action="a.jsp" method="get">
    用户名：<input type="text" name="fname" /><br />
    <input type="submit" value="张三的提交" /><br/>
    <input type="submit" formaction="b.jsp" value="李四的提交" />
</form>
```

6、`list属性`，用于设置输入域的datalist元素，为list属性设置datalist的id属性值，可以将datalist元素与input元素相关联。（谷歌浏览器可用）
list属性适应于以下类型的input元素：text、search、url、telephone、email、date、 pickers、number、range和color。

```html
<input type="url" list="url_list" name="myUrl" />
<datalist id="url_list">
  <option label="Microsoft" value="http://www.microsoft.com" />
  <option label="Google" value="http://www.google.com" />
  <option label="百度" value="http://www.baidu.com" />
</datalist>
```

7、`multiple属性`，用于设置input元素是否可以有多个值。该属性只适用于email和file类型的input元素。
如果给email类型的input元素设置multiple属性，那么在输入框中可以输入多个email地址，多个email地址之间用逗号隔开。
如果给file类型的input元素设置multiple属性，那么在打开的选择文件对话框中就可以选择对个文件。

```html
E-mail：<input type="email" name="myEmail" multiple />
File：<input type="file" name="myFile" multiple />
```

8、`pattern属性`，正则表达式由一系列字符和数字组成，用于匹配某个句法规则。
该属性适应于text、search、url、telephone、email和password类型的input元素。
```html
<form>
    <input type="text" name="myName" pattern="[a-zA-Z]\w{5,15}$">
    <html>以字母开头，6-16位</html>
    <input type="submit" value="提交">
</form>
```

## checkValidity方法

HTML5客户端校验：`checkValidity`方法可以用于检验你的输入是否合法，合法时返回true,否则返回false。
