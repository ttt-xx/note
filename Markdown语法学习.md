# Markdown语法学习

## 介绍以及软件

​	Markdown是一种轻量化的标记语言,能将写的内容转换成XHTML(亦或者是HTML)文档

​	语法简单明了,写熟悉了后用起来既有效率又舒服,我是开始接触GitHub博客才开始使用,最开始也就是现在,使用的软件为:```Typroa```

​	这个软件的好处就在于,支持```实时预览的Markdown文本编辑器```,而且是完全```免费```的

***

## 接下来是语法部分

### Headers标题

```` 标题
#  H1
##  H2
###  H3
####  H4
#####  H5
######  H6
要记住# 后面要打空格
另外H1和H2还可以
一级标题
===
二级标题
---
````
***
### Emphasis强调文本体

*斜体*

````
*斜体*
_斜体_
````

**加粗**

````
**加粗**
__加粗__
````

***粗斜体***

````
***粗斜体***
___粗斜体___
````

___如果* 和_ 两边逗游空白的,就会变成普通的符号___

如果要加入普通*&_,使用反斜线

````
\*转义符\*
````
***
### 列表

​	无序列表

````
* 无序列表
* 子项
* 子项

+ 无序列表
+ 子项
+ 子项

- 无序列表
- 子项
- 子项
````

	* 第一种实现效果

+ 第二种实现效果

- 第三种实现效果



​	有序列表

````
1. 第一行
2. 第二行
3. 第三行
 
1. 第一行
- 第二行
- 第三行
````

1. 第一种实现效果

2. 第一种实现效果

1. 第二种实现效果
2. 第二种实现效果

***
### 超链接

​	 Inline-style 内嵌方式： 

````
[link text](https://www.google.com "title text")
````

​	 Reference-style 引用方式： 

````
[link text][id]
[id]: https://www.mozilla.org "title text"
````

​	 Relative reference to a repository file 引用存储文件： 

````
[link text](../path/file/readme.text "title text")
````

​	 Email 邮件： 

````
<example@example.com>
````
***
### 加入图片

​	其实就是在超链接的基础上加上 !

​	Inline-style 内嵌方式：

```` 
![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "title text")
````

​	 Reference-style 引用方式： 

````
![alt text][logo]
[logo]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "title text"
````
***
### 高亮

```包裹起来```

````
`` `包裹起来` ``
````
***
### 注释

> 注释

````
>注释
````
***
### 水平分割线

例如:

***

````
***
````

### 代码块

````
​```` 加上内容
````

