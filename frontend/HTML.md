# 特殊标签

## \<!DOCTYPE\>

只有在html文档的开头使用适当的doctype才能确保浏览器解析页面的时候启用标准模式（Standards mode），以此才能确保在不同浏览器中得到一个统一的css样式。

HTML5中只定义了一种声明：

`<!DOCTYPE html>`

## \<meta\>

1. 指定html文档的元信息(字符集, 描述, 搜索关键字, 作者)
2. 响应式布局中指定视窗宽度, 缩放比例
3. 指定自动刷新页面

```html
<head>
  <meta charset="UTF-8">
  <meta name="description" content="Free Web tutorials">
  <meta name="keywords" content="HTML, CSS, JavaScript">
  <meta name="author" content="John Doe">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="refresh" content="30">
</head>
```

## link

导入外部文件

* rel: 导入的文件和html文档的关系
* hrel: 要导入的文件的url

```
<link rel="stylesheet" href="styles.css">
<link rel="icon" href="favicon.png">
```

> favicon.ico文件应置于根目录下时多数浏览器可以直接识别为图标

# 常用标签

## a页面基本元素

```html
    <!-- inline元素 href表示链接URL target表示打开链接的方式-->
    <a href="#top" target="_blank">test</a>
    <!-- href=URL/ #(当前页面元素的id)/相对路径 -->
```

* a的默认样式一般很难看, 使用css消除

```css
a{
    text-decoration: none;
    color: #333;
}
```

* 属性href

href: 包含超链接指向的 URL 或 URL 片段。URL 片段是哈希标记（#）前面的名称，哈希标记指定当前文档中的内部目标位置（HTML 元素的 ID）。URL 不限于基于 Web（HTTP）的文档，也可以使用浏览器支持的任何协议。例如，在大多数浏览器中正常工作的file: ftp:和 mailto：

```
href = "http://www.baidu.com"
href = "file://e://test.png"
href = "./test.png"
```

## p文章相关

* p 元素会自动在其前后创建一些空白(缩进和段间距)。浏览器会自动添加这些空间，您也可以在样式表中规定。
* i定义与文本中其余部分不同的部分，并把这部分文本呈现为斜体文本。

## table

```html
    <table>
        <caption>Caption</caption>
        <thead>
            <tr>
                <td>标题A</td>
                <td>标题B</td>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>a</td>
                <td>b</td>
            </tr>
        </tbody>
    </table>
```

table默认只有对齐, 需要其他样式可以设置css

```css
/* table默认无边框 */
table{
    border: 3px solid rgb(95, 219, 46);
    border-collapse: collapse;
}
/* 单元格也是默认无边框 */
td{
    text-align: center;
    border: 2px solid rgb(42, 100, 19);
}
```

# canvas

html5原生支持canvas绘图
