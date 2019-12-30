## 1. 背景

平时我们对html只关注其标签，但是我们偶尔也会隐隐约约感觉到html和js这样的语言有一些本质的不同

实际上，js是图灵完备的，即可以理解成”包含了表达一切逻辑的能力“，像html这样的语言，我们称之为标记语言，它是纯文本的一种升级

html的设计是根据一种古老的标记语言`SGML`来的，html是SGML中规定的一种格式，但是实际的浏览器没有任何一个是通过SGML引擎来解析HTML的

接下来，我们就开始介绍SGML留给html的重要遗产：基本语法和DTD

## 2. 基本语法

首先html作为SGML的子集，它遵循着SGML的基本语法，包含标签和转义

* 基本语法
    * 标签<tag></tag>
    * 文本
        * text
        * <![CDATA[text]]> 在CDATA节点内，不需要考虑多数的转义情况，可以处理字符中存在大量的<&等转义字符的情况
    * 注释<!--comments-->
    * DTD<!Doctype html>
    * 处理信息ProcessingInstruction语法 多数情况下，该语法是给及其看的

## 3. DTD

DTD全称Document Typed Defination，也就是文档类型定义，SGML用DTD来定义每一种文档类型，在h5之前，html都是使用符合SGML规定的DTD

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
```

html4.01有三种DTD

* 严格模式 规定了html4.01中需要的标签
* 过渡模式 除了上述标签，还包含一些被贬斥的标签，这些标签已经不再推荐使用了，但是在这个模式中还保留了
* frameset模式 现在很少见了，它使用frameset标签将几个网页组合到一起

众所周知，html中允许一些标签不闭合的用法，实际上这些都是符合SGML规定的，并且在DTD中规定好了的，但是一些程序员喜欢严格遵守XML语法，保证标签闭合性，所以HTML4.01又规定了XHTML语法

其实这些复杂的DTD语法并没有什么实际作用，因为浏览器根本不会用SGML引擎解析它们，所以到了h5时代，规定了一个简单的大家都能记住的DTD

```
<!DOCTYPE html>
```