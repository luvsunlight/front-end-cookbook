> chrome控制台常用指令api

## 控制台console API

> 事实上，我们只需要在控制台里输入console即可看到它的所有参数

### 1. console.log

### 2. console.info

### 3. console.warning

### 4. console.error

### 5. console.group & console.groupEnd

### 6. console.assert(expression, description)

> 如果指定表达式返回false，就会将desc输出至控制台上

### 7. console.count

> 输出count在`同一行`用`同一个标签`调用的次数

```
login = () => console.count(“called”)

called 1
called 2
called 3

login = user => console.count(“called” + user)

called a 1
called b 1
called c 1

```

### 8. console.dir

输出指定对象的描述，如果记录对象是一个html元素，则输出其dom对象属性

### 9. console.time & console.timeEnd

### 10. console.profile & console.profileEnd

用于分析cpu的状态

### 11. console.timeLine & console.timeLineEnd

### 12. console.trace

### 13. console.clear

> 顾名思义，清空控制台

## 实用技巧

### 1. 快速寻找文件

`ctrl/cmd(mac) + p` => 快速寻找文件

### 2. 全局搜索

`cmd + opt + f` => 在全部加载的文件中全局搜索

### 3. css选择器

chrome调试器自带一个类jquery的类选择器

#### 3.1 $(querySelector)

> 返回`第一个`和css选择器匹配的元素

#### 3.2 $$(querySelector)

> 返回所有和css选择器匹配的元素

#### 3.3 $0-$4

依次返回五个最近在元素面板选择过的DOM元素的历史记录，$0是最新的记录，以此类推


## Refer

* [chrome 控制台常用指令api](https://blog.csdn.net/u598975767/article/details/75047436)
* [Chrome开发工具 控制台 API 参考](https://www.w3cschool.cn/chromedevtools/ntao1oeh.html)