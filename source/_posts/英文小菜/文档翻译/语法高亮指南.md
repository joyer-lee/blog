---
#DO NOT TOUCH — Managed by doc writer
title: 语法高亮指南
categories: --vscode docs
ContentId: 2bb06188-d394-4b98-872c-0bf26c8a674d
date: 2022-12-06 15:43
#Summarize the whole topic in less than 300 characters for SEO purpose
MetaDescription: 语法高亮指南
---

# 语法高亮指南

语法高亮决定了在Visual Studio 代码编辑器中显示的源代码的颜色和风格。 它负责为 `if` 或 `for` 着色以区别于字符串、注释和变量名等其他JavaScript关键字。

语法高亮由两部分组成：

- [令牌化](#tokenization): 将文本分解为令牌列表
- [主题化](#theming): 使用主题或用户设置将令牌映射到特定颜色和样式

在深入了解细节之前， 一个良好的开端是使用 [范围检查器](#scope-inspector) 工具来探索源文件中存在什么样的令牌以及它们匹配的主题规则。 要看到语义和语法令牌，请在TypeScript 文件上使用一个内置的主题 (例如，Dark+)。

## 令牌化

文本的令牌化就是将文本分解为段，并用标记类型对每个段进行分类。

VS Code的令牌华引擎由[TextMate 语法][tm-grammars]驱动。 TextMate 语法是一个结构化的正则表达式集，作为插件(XML) 或JSON文件写成的。 VS Code扩展可通过 `语法` 贡献点贡献语法。

TextMate令牌化引擎与渲染器运行在相同的进程中，令牌随着用户类型的变化而更新。 令牌用于语法高亮，但也用于将源代码划分为注释、字符串、正则表达式区域。

从版本1.43开始，VS 代码也允许扩展通过 [语义令牌提供商](/api/references/vscode-api#DocumentSemanticTokensProvider) 提供令牌化。 语义提供者通常由语言服务器实现，这些语言服务器对源文件有更深的了解，并且可以在项目中解析符号。 例如，一个常量变量名可以在整个项目中使用常量高亮来呈现，而不仅仅是在其声明的地方。

基于语义令牌的高亮被视为基于 TextMate 的语法高亮的补充。 语义高亮显示在语法高亮上。 由于语言服务器可能需要一段时间来加载和分析一个项目，语义标记高亮显示可能会在短时间延迟后出现。

这篇文章着重于基于 TextMate 的令牌化。 语义令牌化和主题化在 [语义高亮指南](semantic-highlight-guide) 中进行了解释。

### TextMate 语法

VS Code使用 [TextMate 语法][tm-grammars] 作为语法令牌化引擎。 TextMate 编辑器创造以来， 由于大量的语言包被被开源社区创建和维护，许多其他的编辑器和IDE都采用了这些准则。

TextMate 语法依靠 [Oniguruma 正则表达式](https://macromates.com/manual/en/regular_expressions) 并且通常都是作为插件或 JSON 编写的。 您可以在[这里](https://www.apeth.com/nonblog/stories/textmatebundle.html)找到 TextMate 语法 很好的简介， 并且你可以看看现有的 TextMate 语法来更多地了解它们是如何工作的。

### TextMate 令牌和范围

令牌是一个或多个字符，是同一个程序元素的一部分。 示例令牌包括操作符，如 `+` 和`*`, 变量名如 `myVar`, 或字符串如 `"my string"`。

每个令牌都与定义令牌上下文的范围相关联。 作用域是一个点分隔的标识符列表，用于指定当前标记的上下文。 例如，JavaScript中的 `+` 操作符作用域为 `keyword.operator.arithmetic.js`。

主题将作用域映射到颜色和样式以提供语法高亮。 TextMate为许多主题目标提供了[常用作用域列表][tm-grammars] 。 为了使您的语法得到尽可能广泛的支持，请尝试在现有范围基础上进一步发展，而不是确定新的范围。

作用域嵌套，以便每个令牌也与父作用域列表相关联。 下面的示例使用[范围检查器](#scope-inspector)在一个简单的JavaScript函数中显示 `+`操作符的范围层次结构。 最具体的范围列在顶部，较一般的父范围列于下面：

![syntax highlighting scopes](images/syntax-highlighting/scopes.png)

父范围信息也用于主题化。 当主题以一个作用域为目标时，所有带有该父作用域的令牌都将被着色，除非主题还为它们各自的作用域提供更具体的着色。

### 贡献基本语法

VS Code支持 json TextMate 语法。 这些都是通过 `语法` [贡献点](/api/references/contribution-points) 贡献的。

每个语法贡献具体指：语法所适用语言的标识符。 语法标记的顶级范围名称和语法文件的相对路径。 下面的例子展示了一个虚构的`abc`语言的语法贡献

```json
{
  "contributes": {
    "languages": [
      {
        "id": "abc",
        "extensions": [".abc"]
      }
    ],
    "grammars": [
      {
        "language": "abc",
        "scopeName": "source.abc",
        "path": "./syntaxes/abc.tmGrammar.json"
      }
    ]
  }
}
```

语法文件本身由顶层规则组成。 这通常分为列出程序顶层元素的 `patterns` 部分和定义每个元素的 `repository`。 语法中的其他规则可以通过 `repository` 使用 `{ "include": "#id" }`来引用元素。

示例 `abc` 语法标记字母 `a`, `b`, 和 `c` 作为关键词和括号内嵌套表达式。

```json
{
  "scopeName": "source.abc",
  "patterns": [{ "include": "#expression" }],
  "repository": {
    "expression": {
      "patterns": [{ "include": "#letter" }, { "include": "#paren-expression" }]
    },
    "letter": {
      "match": "a|b|c",
      "name": "keyword.letter"
    },
    "paren-expression": {
      "begin": "\\(",
      "end": "\\)",
      "beginCaptures": {
        "0": { "name": "punctuation.paren.open" }
      },
      "endCaptures": {
        "0": { "name": "punctuation.paren.close" }
      },
      "name": "expression.group",
      "patterns": [{ "include": "#expression" }]
    }
  }
}
```

语法引擎将尝试对文档中的所有文本依次应用 `表达式`规则。 对于一个简单的程序，例如：

```
a
(
    b
)
x
(
    (
        c
        xyz
    )
)
(
a
```

示例语法生成以下作用域(从左到右从最特定的作用域到最不特定的作用域列出)

```
a               keyword.letter, source.abc
(               punctuation.paren.open, expression.group, source.abc
    b           keyword.letter, expression.group, source.abc
)               punctuation.paren.close, expression.group, source.abc
x               source.abc
(               punctuation.paren.open, expression.group, source.abc
    (           punctuation.paren.open, expression.group, expression.group, source.abc
        c       keyword.letter, expression.group, expression.group, source.abc
        xyz     expression.group, expression.group, source.abc
    )           punctuation.paren.close, expression.group, expression.group, source.abc
)               punctuation.paren.close, expression.group, source.abc
(               punctuation.paren.open, expression.group, source.abc
a               keyword.letter, expression.group, source.abc
```

注意，当前作用域中包含了不被任何规则匹配的文本，例如字符串`xyz`。 文件末尾的最后一个括号是`expression.group`的一部分，即使结束规则不匹配，因为在结束规则之前找到了文档结束。

### 嵌入式语言（Embedded languages）

如果您的语法在父语言中包含嵌入式语言，例如HTML中的CSS样式块，你可以使用`embeddedLanguages`贡献点告诉VS Code将嵌入式语言与父语言区别对待。 这确保括号匹配、注释和其他基本语言特性在嵌入式语言中按预期工作。

`embeddedLanguages`贡献点将嵌入式语言中的作用域映射到顶级语言作用域。 在下面的例子中，`meta.embedded.block.javascript`作用域中的任何令牌都将被视为JavaScript内容：

```json
{
  "contributes": {
    "grammars": [
      {
        "path": "./syntaxes/abc.tmLanguage.json",
        "scopeName": "source.abc",
        "embeddedLanguages": {
          "meta.embedded.block.javascript": "javascript"
        }
      }
    ]
  }
}
```

现在，如果你试图评论代码或触发代码片段在标记为 `meta.embeded.block.javascript`的令牌集合中, 他们将得到正确的 `//` JavaScript 风格的评论和正确的 JavaScript 代码片段。

### 开发新的语法扩展

快速创建一个新的语法扩展， 使用 [VS Code's Yeoman templates](/api/get-started/your-first-extension) 来运行 `yo code` 并选择 `New Language` 选项：

![Selecting the 'new language' template in 'yo code'](images/syntax-highlighting/yo-new-language.png)

Yeoman将通过一些基本问题带你去搭建新的扩展。 创建新语法的重要问题是：

- `Language id` - 您的语言的唯一标识符。
- `Language name` - 为您的语言提供一个人类可读的名称。
- `Scope names` - 语法的根TextMate作用域名称.

![Filling in the 'new language' questions](images/syntax-highlighting/yo-new-language-questions.png)

生成器假设您想要为该语言定义一种新语言和一种新语法。 如果您正在为现有语言创建语法，只需用目标语言的信息填充这些语法，并确保删除生成的`package.json`中的`languages`贡献点。

在回答所有问题后，Yeoman将创建一个结构性的新扩展：

![A new language extension](images/syntax-highlighting/generated-new-language-extension.png)

记住，如果你要为VS Code已经知道的语言贡献语法，一定要删除生成的`package.json`中的`languages`贡献点。

#### 转换现有的 TextMate 语法

`yo code` 也可以帮助将现有的 TextMate 语法转换为 VS 代码扩展。 同样，首先运行`yo code`并选择`语言扩展`。 当被问到一个现有的语法文件时，给它一个`.tmLanguage`或`.json TextMate`语法文件的完整路径：

![Converting an existing TextMate grammar](images/syntax-highlighting/yo-convert.png)

#### 使用 YAML 书写语法

随着语法变得越来越复杂，将其作为json来理解和维护可能会变得困难。 如果您发现自己正在编写复杂的正则表达式，或者需要添加注释来解释语法的各个方面，那么可以考虑使用yaml来定义语法。

Yaml语法具有与基于json的语法完全相同的结构，但允许您使用Yaml更简洁的语法，以及多行字符串和注释等特性。

![A yaml grammar using multiline strings and comments](images/syntax-highlighting/yaml-grammar.png)

VS Code只能加载json语法，所以基于yaml的语法必须转换成json。 `js-yaml`包和命令行工具使这变得很容易。

```bash
# 将js-yaml安装为扩展中的开发依赖项
$ npm install js-yaml --save-dev

# 使用命令行工具将yaml语法转换为json
$ npx js-yaml syntaxes/abc.tmLanguage.yaml > syntaxes/abc.tmLanguage.json
```

### 注入语法（Injection grammars）

注入语法允许您扩展现有的语法。 注入语法是一种常规的TextMate语法，它被注入到现有语法中的特定范围中。 注入语法的实例应用：

- 在评论中突出TODO等关键字。
- 在现有语法中添加更具体的范围信息。
- 为Markdown围栏代码块添加新语言的高亮显示。

#### 创建一个基本的注入语法

就像常规语法一样，注入语法通过 `package.json` 所提供。 然而，注入语法不指定语言，而是使用`injectTo`来指定要将语法注入到其中的目标语言作用域列表。

对于本例，我们将创建一个简单的注入语法，将`TODO`突出显示为JavaScript注释中的关键字。 为了在JavaScript文件中应用我们的注入语法，我们在`injectTo`中使用`source.js`目标语言作用域：

```json
{
  "contributes": {
    "grammars": [
      {
        "path": "./syntaxes/injection.json",
        "scopeName": "todo-comment.injection",
        "injectTo": ["source.js"]
      }
    ]
  }
}
```

除了最顶层的`injectionSelector`条目外，语法本身是标准的TextMate语法。 `injectionSelector`是一个范围选择器，它指定应该在哪个范围中应用注入的语法。 在我们的例子中，我们想要在所有的`//`注释凸显单词`TODO`。 使用`scope inspector`，我们发现JavaScript的双斜杠注释具有`comment.line.double-slash`范围，所以我们的注入选择器是`L:comment.line.double-slash`：

```json
{
  "scopeName": "todo-comment.injection",
  "injectionSelector": "L:comment.line.double-slash",
  "patterns": [
    {
      "include": "#todo-keyword"
    }
  ],
  "repository": {
    "todo-keyword": {
      "match": "TODO",
      "name": "keyword.todo"
    }
  }
}
```

注入选择器中的 `L：` 意味着注入被添加到现有语法规则的左边。 这基本上意味着我们注入的语法规则将在任何现有语法规则之前应用。

#### 嵌入式语言（Embedded languages）

注入语法也可以为父级语法提供嵌入语言。 就像普通语法一样，注入语法可以使用`embeddedLanguages`将范围从嵌入式语言映射到顶级语言范围。

例如，在JavaScript字符串中突出显示SQL查询的扩展可以使用`embeddedLanguages`来确保标记为`meta.embedded.inline.sql`的字符串中的所有令牌都被视为SQL，以实现基本的语言功能，如括号匹配和代码片段选择。

```json
{
  "contributes": {
    "grammars": [
      {
        "path": "./syntaxes/injection.json",
        "scopeName": "sql-string.injection",
        "injectTo": ["source.js"],
        "embeddedLanguages": {
          "meta.embedded.inline.sql": "sql"
        }
      }
    ]
  }
}
```

#### 令牌类型和嵌入语言

对于注入语言嵌入式语言还有一个额外的复杂性: 默认情况下，VS Code将字符串中的所有标记视为字符串内容，将带有注释的所有标记视为标记内容。 由于括号匹配和自动关闭对等功能在字符串和注释中被禁用，如果嵌入式语言出现在字符串或注释中，这些嵌入式语言中的功能也将被禁用。

要覆盖此行为，可以使用`meta.embedded.*`范围重置VS Code标记为字符串或注释内容。 将嵌入式语言封装在`meta.embedded.*`范围中是个好主意，以确保VS Code正确的处理嵌入式语言。

如果您不能添加 `meta.embeded.` 范围到您的语法中， 您也可以在语法的贡献点中使用 `tokenTypes` 将特定范围映射到内容模式。 下面的 `tokenTypes` 部分确保了 `my.sql.template.string` 范围中的任何内容被当作源代码处理：

```json
{
  "contributes": {
    "grammars": [
      {
        "path": "./syntaxes/injection.json",
        "scopeName": "sql-string.injection",
        "injectTo": ["source.js"],
        "embeddedLanguages": {
          "my.sql.template.string": "sql"
        },
        "tokenTypes": {
          "my.sql.template.string": "other"
        }
      }
    ]
  }
}
```

## 主题

主题化是为令牌分配颜色和样式 主题规则在颜色主题中指定，但用户可以在用户设置中自定义主题规则。

TextMate 主题规则在 `tokenColors` 中定义，并且与普通的 TextMate 主题具有相同的语法。 每个规则都定义一个 TextMate 范围选择器和由此产生的颜色和风格。

在评估令牌的颜色和样式时，将根据规则的选择器匹配当前令牌的作用域，为每个样式属性(前景色、粗体、斜体、下划线) 找到最具体的规则。

[颜色主题指南](/api/extension-guides/color-theme#syntax-colors) 描述了如何创建一个颜色主题。 语义令牌的主题在 [语义高亮指南](semantic-highlight-guide#theming) 中作了解释。

## 范围检查器（Scope inspector）

VS Code的内置范围检查工具有助于调试语法和语义标记。 它在文件中当前位置显示令牌和语义语义令牌的范围，以及关于应用于该令牌的主题规则的元数据。

在命令面板使用`Developer: Inspect Editor Tokens and Scopes`或`create a keybinding`命令可触发范围检查器：

```json
{
  "key": "cmd+alt+shift+i",
  "command": "editor.action.inspectTMScopes"
}
```

![scope inspector](images/syntax-highlighting/scope-inspector.png)

范围检查器显示以下信息：

1. 当前令牌。
1. 关于令牌的元数据和关于其计算外观的信息。 如果您使用的是嵌入式语言，那么这里的重要条目是`language`和`token type`。
1. 当语义令牌提供程序可用于当前语言且当前主题支持语义高亮显示时，将显示语义令牌部分。 它显示了当前的语义标记令牌和修饰符，以及与语义令牌类型和修饰符匹配的主题规则。
1. TextMate部分显示了当前TextMate令牌的作用域列表，最具体的作用域位于顶部。 它还显示与范围相匹配的最具体主题规则。 这只显示负责令牌当前样式的主题规则，不显示被覆盖的规则。 如果存在语义令牌，则仅当主题规则与匹配语义令牌的规则不同时才显示主题规则。

[tm-grammars]: https://macromates.com/manual/en/language_grammars

[tm-grammars]: https://macromates.com/manual/en/language_grammars
