## 1. 定义

第一次听说AST(Abstract syntax tree)抽象语法树的时候，是在你不知道的js里，里面说传统的编译语言编译过程分为三步

* 词法分析，将我们输入的代码分解成有意义的代码块，这些代码块被称为`词法单元`
* 根据词法单元，构建AST抽象语法树，即将词法块整合起来
* 代码生成，即根据AST转化成可执行代码的过程

那个时候就觉得AST真是高大上啊，感觉只有那种会手写编译器的大牛才会做的事，但是没有想到AST竟然在日常代码里运用得如此广泛，很多构建工具如babel，eslint，框架如vue，react等都在广泛使用这一项技术（我曾经还很天真地以为是用正则去匹配）

AST的应用范围可太广了

* 代码语法检查
* 代码风格检查
* 高亮
* 自动补全
* 代码压缩
* 代码打包
* 模块化
* 超集语言
* ...，

了解了基本定义之后，我们就从方法入门

esprima是一个js-parser解析器，它可以将js源码转化为抽象的语法库，但是它过于底层而且只提供代码解析功能，即它只能将代码转化成AST，但是不能将AST重新转化为代码，如果想要这个功能，我们还需要`escodegen`来根据AST生成代码

[jscodeshift](https://github.com/facebook/jscodeshift)是一种` codemod toolkit`工具，它是recast的wrapper，recast是esprima的wrapper。它的作用就是解析js，将js解析成AST，然后提供一些遍历的操作接口，我们可以做一些比如垫片或者babel，又或者是代码自动升级工具。要使用jscodeshift，就必须要使用它的配套网站[AST Explorer](https://astexplorer.net/)，虽然看起来很简单，但是功能一点都不漏下。

## 2.基本操作

我们设定要操作的对象代码段为

```

let a = 1
a = 2
app.say = function(test) {
  console.log(test);
}

app.get('/api/config/save', checkConfigHighRiskPermission, function() {
  console.log('cool')
});

app.say('123')
```

### 查

我们在上述网站中，编辑自己写的代码，随意点击一块区域，就会出现其在AST中对应的节点位置，用这个功能，我们可以很快定位我们想要修改的代码部分。

每一个可修改的节点都存在一个type属性，我们就要好好利用该属性来定位节点。比如`a = 0`,我们想要将`=`修改成`-=`,我们首先找到=所在的位置，=自己是一个操作符，并不是具体的节点对象，我们找到它的父节点`AssignmentExpression`赋值表达式，它具有属性`type:AssignmentExpression,operator:"="`,于是


```
export default function transformer(file, api) {
  const j = api.jscodeshift;

  return j(file.source)
    .find(j.AssignmentExpression, { operator: "=" })
    .toSource();
}
```

我们通过如上代码可以定位到几乎每一个元素的位置，查的功能就此完结

### 改

改分为两种，一种是修改属性，一种是全部修改，这里举两个例子辅助理解。第一种修改属性

```
return j(file.source)
    .find(j.AssignmentExpression, { 
        operator: "=",
        left: { type: "Identifier" }
     })
    .forEach(path => {
        path.node.operator = ">"
    })
    .toSource();
```

path.node指向的就是当前父节点即`a = 2`这样一个赋值表达式,我们直接修改其operator值就可以达到目的

我们还可以整个修改，不过那样的话方法就有所不一样

```
// Press ctrl+space for code completion
export default function transformer(file, api) {
  const j = api.jscodeshift;

  return j(file.source)
    .find(j.AssignmentExpression, {
      operator: "=",
      left: { type: "Identifier" }
    })
    .forEach(path => {
      j(path).replaceWith(
        j.assignmentExpression("-=", path.value.left, path.value.right)
      );
    })
    .toSource();
}
```

要注意这里的整体替换时的构造函数

* 名称小写
* 一般没有loc，range和type，其他属性都要带上

### 增

提供了insertAfter和insertBefore方法

```
ast.find(j.FunctionExpression)
    .forEach(path => {
        j(path).insertAfter(
            j.arrowFunctionExpression(
                path.value.params,   // 方法参数
                path.value.body,     // 方法体
                false                // expression
            )
        )
    })
```

### 删

删更简单，直接将一项替换成空即可

```
j(path).insertAfter(
    j(`console.log('123123')`).find(j.ExpressionStatement).__paths[0].value
)
```

## Refer

* [jscodeshift简易教程](https://www.cnblogs.com/axes/p/7694041.html)
* [手把手教你写几个AST插件](https://mp.weixin.qq.com/s/GJ1H6gMoB1BEYfQbF3RQpQ)
* [代码重构利器:jscodeshift](https://cloud.tencent.com/developer/article/1009072)