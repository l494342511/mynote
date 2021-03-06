# HTML5

## 1. 什么是html

html是一个标记语言，通过使用标签来描述网页

HTML指的是Hyper Text Markup Language

## 2. HTML的基本标签

| 标签                                | 描述                                              |
| ----------------------------------- | ------------------------------------------------- |
| \<html>                             | 定义html文档                                      |
| \<body>                             | 定义文档的主体                                    |
| \<h1>\</h1>                         | 通过\<h1> - \<h6> 标签来定义标题                  |
| \<p>\</p>                           | 定义一个段落                                      |
| \<br/>                              | 换行 可以在其他标签里面使用                       |
| \<hr/>                              | 创建水平线，用于分隔内容                          |
| \<a href="">\</a>                   | \<a>\</a>标签定义一个链接，href属性指定链接的地址 |
| \<img  src="" width="" height="" /> | 定义图片                                          |

html允许嵌套元素

## 3. HTML的基本属性

| 属性  | 描述                   |
| ----- | ---------------------- |
| class | 为html元素定义一个类名 |
| id    | 定义元素的id           |
| style | 定义元素的样式         |
| title | 描述了元素的额外信息   |

## 4. 文本格式

**文本格式标签**

| 标签                | 描述                        |
| ------------------- | --------------------------- |
| \<b>\</b>           | <b>用来加粗文本</b>         |
| \<em>\</em>         | <em>着重</em>               |
| \<small>\</small>   | <small>定义小号</small>     |
| \<strong>\</strong> | <strong>加重语气</strong>   |
| \<i>\</i>           | <i>斜体文本</i>             |
| \<code>\</code>     | <code>定义计算机代码</code> |
| \<sub>\</sub>       | <sub>下标</sub>             |
| \<sup>\</sup>       | <sup>定义上标</sup>         |
| \<ins>\</ins>       | <ins>定义插入字</ins>       |
| \<del>\</del>       | <del>定义删除字</del>       |

**计算机输出**

| 标签            | 描述                        |
| --------------- | --------------------------- |
| \<code>\</code> | <code>定义计算机代码</code> |
| \<kbd>\</kbd>   | <kbd>定义键盘码</kbd>       |
| \<samp>         | 定义计算机代码样本          |
| \<var>\</var>   | <var>定义变量</var>         |
| \<pre>\</pre>   | 定义预格式文本              |

**引文**

| 标签                        | 描述                                  |
| --------------------------- | ------------------------------------- |
| \<abbr>\</abbr>             | <abbr>定义缩写</abbr>                 |
| \<address>\</address>       | <address>定义地址</address>           |
| \<bdo>\</bdo>               | 定义文字方向                          |
| \<blockquote>\</blockquote> | <blockquote>定义长的引用</blockquote> |
| \<q>\</q>                   | <q>定义短语的引用语</q>               |
| \<cite>\</cite>             | <cite>定义引用、引证</cite>           |
| \<dfn>\</dfn>               | <dfn>定义一个定义项目</dfn>           |

## 5. HTML超链接

html使用\<a>\</a>标签定义超链接

```html
<a id="bilibili" href="http://www.bilibili.com" target="_blank" rel="noopener noreferrer">点击跳转到哔哩哔哩</a>
<a href="#bilibili">超链接的超链接</a>
<a href="http://www.bilibili.com"><img src="/HTML5-01/images/pikaqiu.png" width="500" height="449"/></a>
```

## 6. 头部head

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <base href="/HTML5-01/images/" />
    <style type="text/css">
        body {background-color: beige}
        p {color: brown}
    </style>
    <meta name="keywords" content="HTML">
    <meta name="description" content="我的第一个网页">
    <meta name="author" content="ljq">
    <meta http-equiv="refresh" content="30">
</head>
```

| 标签      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| \<base>   | 定义了页面中所有链接默认的根目录                             |
| \<title>  | 标签定义了页面的标题                                         |
| \<meta>   | 使用 \<meta> 元素来描述HTML文档的描述，关键词，作者，字符集等。 |
| \<link>   | 定义了html文档和外部资源之间的关系，常用来链接到样式表       |
| \<style>  | 定义了样式文件引用地址，也可以直接添加样式渲染HTML文档       |
| \<script> | 用于加载脚本文件                                             |

## 7. HTML使用CSS

HTML使用样式的方法有三种：

1. 内联样式：使用\<style>标签，当某个元素需要使用特定样式时，也可以使用内联样式

   ```html
   <p style="color: blue;margin-left: 20px;text-align: center;">使用内联样式的段落</p>
   ```

2. 内部样式表：在\<head>中使用\<style>元素来指定css

   ```html
   <!-- 内部样式表可以设置一个默认的样式 -->
   <head>   
   	<style type="text/css">
           body {background-color: beige}
           p {color: brown}
       </style>
   </head>
   ```

3. 使用外部css文件：需要适配很多页面时，最好使用外部样式表

   ```html
   <head>
       <link rel="stylesheet" type="text/css" href="/css/mystyle.css"> 
   </head>
   ```

   