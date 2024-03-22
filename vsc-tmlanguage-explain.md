# VSCode Tmlanguage Explain

VSCode使用syntaxes来描述一门语言。syntaxes一般使用 **JSON格式** ，但是因为 **JSON格式** 比较啰嗦，同时不能加注释，作为配置文件，在代码较多的情况下，会显得非常难以阅读。

这里（包括官方）推荐使用 **YAML格式** 来作为源代码配置，然后再手动或者通过自动化转化为 **JSON格式** 。

为了保证可读性，如不特殊说明，以下特殊说明， **以下部分均使用YAML格式代替JSON格式** 。

## 基本模板

一份syntax文件的模板是这样子的：

```yaml
---
name: Syntax Name
scopeName: source.syntax_name
patterns: []
repository: {}
---
```

> **name**
>
> \[选填\]
>
> 语法名称。在某些ide里会当做名称放在列表菜单里。

> **scopeName**
>
> \[必填\]
>
> 语法的顶级scope名称，可以在syntax文件中随处引用。命名方式有 `source.<lang_name>` 或者 `text.<lang_name>` 。前者一般用于 **程序语言** ，后者一般用于 **标记语言** 。
> 
> 比如，JavaScript的是 `scopeName` 为 `source.js` ，HTML的 `scopeName` 为 `text.html.basic` 。

> [**patterns**](#patterns)
>
> \[必填\]
>
> 语法正文。

> [**repository**](#repository)
>
> \[选填\]
>
> 用于存储 `sub scope` ，类似于变量的存在，挂在顶级 `scope` 名下。
> 可以用 `include` 引入。

## 语法正文

syntax使用正则表达式匹配语法，这里的正则使用的是[这个版本](https://raw.githubusercontent.com/kkos/oniguruma/master/doc/RE)。相对于前端同学熟悉的阉割版的JS正则表达式，这个版本的正则表达式更完整，有更多的语法。

### Patterns

#### 完整匹配

```yaml
---
patterns:
  - name: keyword.other.ssraw
    match: \$([A-Za-z][A-Za-z0-9_]+)
    captures:
      '1': {name: constant.numeric.ssraw}
---
```

> **name**
>
> \[可选\]
> 
> 规则名称。会作为语法堆栈解析上的名称。

> **match**
> 
> \[必填\]
> 
> 匹配规则。

> **captures**
>
> \[必填\]
> 
> 正则表达式分组对应的名称。会作为语法堆栈解析上的名称，分组从`1`开始

#### begin-end匹配

```yaml
patterns:
  - name: string.other.ssraw
    begin: (\$)(\{})([0-9]+):
    beginCaptures:
      '1': {name: keyword.other.ssraw}
      '3': {name: constant.numeric.ssraw}
    end: \}
    patterns:
      - include: 'source.js#string'
      - include: '#command-set'
      - name: support.other.ssraw
        match: .
```

begin-end规则匹配顺序是 `begin` -> `patterns[].include` -> `end`

> **begin**
>
> 开始边界。

> **beginCaptures**
>
> 对应begin正则表达式分组对应的名称，同captures的规则。

> **end**
>
> 结束边界。

> **endCaptures**
>
> 对应end正则表达式分组对应的名称，同captures的规则。

> **patterns**
>
> 同开头的patterns。

> **include**
>
> 引入其他scope。当引入repository当中的scope时，需要加上#，顶级scope不需要这个符号。
> 
> 比如，`source.js#string`，意思是引入`source.js`下repository中的`string`。

### repository

Repository与Patterns类似，区别在于需要以scope名称开头。

```yaml
---
repository:
  # set
  command-set:
    name: meta.namespace.set
    begin: (set)(:)
    beginCaptures:
      '1': {name: keyword.commands.set}
      '2': {name: punctuation.separator.key-value.csr}
    end: (?=\s*\?>)
  # name
  command-name:
    name: meta.namespace.name
    match: (name)(:)([a-zA-Z0-9._]+)
    captures:
      '1': {name: storage.type.name}
      '2': {name: punctuation.separator.key-value.csr}
      '3': {name: variable.other.readwrite}
---
```

repository各属性与patterns的规则一致。

### scope命名

这里有个疑问，**name** 与 **captures**当中的这些命名是根据什么来的呢？

回想一下，我们在使用vscode时，不同语法模块显示的颜色不一样，具有不同的样式。这里的这些命名可以理解为某些特定样式的class下的属性。比如`storage.type`有一个预设样式为`foreground: "#569cd6"`，那么`storage.type.function.js`就会命中这个样式，显示`#569cd6`这个颜色。vscode皮肤插件就是针对这些class设计不同的样式，至于怎么使用这些牙膏好死，就是我们的问题了。

同时，根据这些命名不难看出，针对大部分语法场景已经有一套成熟的使用场景。

在captures当中，比如单引号用`string.quoted.single`，比较运算符用`keyword.operator.relational`。

在name当中，一般以meta开头命名代表语法块，比如**if语法块**叫做`meta.tag.if`，方法里面的参数叫做`meta.function.parameters`。

想要参考已有的语法设计结构，可以在vscode编辑器区域当中用组合键：`cmd(win) + shift + p`，选择“Developer: Inspect TM Scopes”，会弹出一个语法结构的浮层，显示当前光标悬浮位置的语法信息。

如果需要比较完整的命名规则，可以[参考这里](https://www.sublimetext.com/docs/scope_naming.html)
