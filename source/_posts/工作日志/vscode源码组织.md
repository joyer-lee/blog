---
title: vscode源码组织
tags: []
id: '117'
categories:
  - - vscode源码解读
date: 2020-12-04 14:47:07
---



## 介绍

vscode由一个分层的、模块化的`core`组成（即src/vs目录），它可以通过`extensions`机制进行扩展。`extensions`运行在称为`extension host`的单独进程中,通过使用`extensions API`去实现。

## 目录结构

### 根目录结构

```
├── build       # gulp编译构建脚本 
├── extensions  # 内置插件 
├── out         # 编译输出目录
├── resources     # 平台相关静态资源，图标等 
├── scripts       # 工具脚本，开发/测试 
├── src           # 源码目录 
└── test          # 测试套件

```

### src下文件目录结构

```
├── bootstrap-amd.js    # 子进程实际入口
├── bootstrap-fork.js   #
├── bootstrap-window.js #
├── bootstrap.js        # 子进程环境初始化
├── buildfile.js        # 构建config
├── cli.js              # CLI入口
├── main.js             # 主进程入口
├── paths.js            # AppDataPath与DefaultUserDataPath
├── typings
│          └── xxx.d.ts        # ts类型声明
└── vs   
    ├── base            # 定义基础的工具方法和基础的 DOM UI 控件   
        │   ├── browser     # 基础UI组件，DOM操作、交互事件、DnD等  
        │   ├── common      # diff描述，markdown解析器，worker协议，各种工具函数     
        │   ├── node        # Node工具函数   
        │   ├── parts       # IPC协议（Electron、Node），quickopen、tree组件   
        │   ├── test        # base单测用例    
        │   └── worker      # Worker factory 和 main Worker（运行IDE Core：Monaco）
    ├── code            # VSCode Electron 应用的入口，包括 Electron 的主进程脚本入口   
        │   ├── electron-browser # 需要 Electron 渲染器处理API的源代码（可以使用 common, browser, node）
        │   ├── electron-main    # 需要Electron主进程API的源代码（可以使用 common, node）   
        │   ├── node        # 需要Electron主进程API的源代码（可以使用 common, node）  
        │   ├── test  
        │   └── code.main.ts 

    ├── editor   # Monaco Editor 代码编辑器：其中包含单独打包发布的 Monaco Editor 和只能在 VSCode 的使用的部分    
        │   ├── browser     # 代码编辑器核心   
        │   ├── common      # 代码编辑器核心   
        │   ├── contrib     # vscode 与独立 IDE共享的代码 
        │   ├── standalone  # 独立 IDE 独有的代码 
        │   ├── test    │   ├── editor.all.ts  
        │   ├── editor.api.ts   
        │   ├── editor.main.ts 
        │   └── editor.worker.ts  
    ├── platform        # 依赖注入的实现和 VSCode 使用的基础服务 Services   
    ├── workbench       # VSCode 桌面应用程序工作台的实现 
    ├── buildunit.json   
    ├── css.build.js    # 用于插件构建的CSS loader   
    ├── css.js          # CSS loader   
    ├── loader.js       # AMD loader（用于异步加载AMD模块，类似于require.js） 
    ├── nls.build.js    # 用于插件构建的 NLS loader   
    └── nls.js          # NLS（National Language Support）多语言loader   

```

## 层级

`core`分为以下几层：

*   `base`: 提供常规实用程序和用户界面构建块
*   `platform`: 为VS Code定义服务注入支持和基本服务
*   `editor`: “ Monaco”编辑器可作为单独的可下载组件使用
*   `workbench`: 托管“Monaco”编辑器，并提供“视口”的框架，例如资源管理器，状态栏或菜单栏，并利用Electron来实现VS Code桌面应用程序。

## 目标环境

VS Code的核心完全采用TypeScript实现。在每一层内部，代码都是由目标运行时环境组成的。这样可以确保仅使用运行时特定的API。在代码中，我们对目标环境做以下区分：

*   `common`目录存放的是：仅需要基本的JavaScript API，并且可以运行在其他目标环境中的源代码
    
*   `browser`目录存放的是：需要访问browserAPI如进行DOM操作的源代码。 可以引入`common`目录下的源码
    
*   `node`目录存放的是：引入nodejs API的源码。 可以引入`common`目录下的源码
    
*   `electron-sandbox`目录存放的是：即需要访问browserAPI如进行DOM操作和要求浏览器API，也需要操作一小部分ElectronAPI用来和Electron主进程进行通信的源代码。 可以使用`common`、`browser`、`electron-sandbox`目录下的源码。
    
*   `electro-browser`目录存放的是：引入了electron 渲染进程API的源码。 可以使用`common`、`browser`、`node`目录文件。
    
*   `electron-main`目录存放的是：引入了electron主进程API的源码。 可以使用`common`、`node`目录源码。
    

## 依赖注入

源码是通过`services`组成的，`services`更多的被定义在`platform`层中。`Services`通过`construction injection`（构造函数注入）的方式得到对应的客户端（clients）。 `service`定义分为两部分： （1）（`service`）服务的接口（`interface`） （2）服务(`service`)标识符(`identifier`) 服务标识符是必需的，因为`TypeScript`不使用标称类型而是结构性类型。 服务标识符是一种修饰器（针对`ES7`提出），并且应与服务接口具有相同的名称。 声明服务依赖关系是通过在构造函数参数中添加相应的修饰来实现的。 在下面的代码段中，`@ IModelService`是服务标识符修饰符，而`IModelService`是此参数的（可选）类型注释。 如果依赖项是可选的，请使用`@optional`装饰，否则实例化服务将引发错误。

```
class Client {
  constructor(
    @IModelService modelService: IModelService, 
    @optional(IEditorService) editorService: IEditorService
  ) {
    // use services
  }
}
```

使用实例化服务为服务使用者创建实例，例如`InstantiationService.createInstance（Client）`。 通常，当您注册为贡献内容（例如`Viewle`或`Language`）时，会为您完成此操作。

## VS Code Editor的源组织

*   `vs/editor`文件夹不应该有任何`node`或`electron-browser`依赖关系。
*   `vs/editor/common`和`vs/editor/browser`\-代码编辑器核心（若没有这些则编辑器没有任何意义）。
*   `vs/editor/contrib`\-VS Code和独立编辑器中附带的代码编辑器贡献。依照`browser`惯例，编辑器在缺失相关`contrib`的情况下，会导致相应功能被删除。
*   `vs/editor/standalone`\-仅独立编辑器附带的代码。不存在别的依赖。
*   `vs/workbench/contrib/codeEditor` -VS Code中附带的代码编辑器贡献。

## 工作台贡献

VS Code工作台（`vs/workbench`）由许多内容组成，可提供丰富的开发经验。示例包括全文搜索，集成的git和debug。从本质上讲，工作台并不直接依赖于所有这些贡献。相反，我们使用内部（而不是真正的扩展API）机制将这些`contrib`贡献给工作台。 贡献给工作台的所有贡献都存放在该`vs/workbench/contrib`文件夹中。此文件夹中有一些规则：

*   `vs/workbench/contrib`文件夹内部代码与外界没有任何依赖关系
*   每个贡献都应从单个文件中公开其内部API（例如`vs/workbench/contrib/search/common/search.ts`）
*   一个贡献可以取决于另一个贡献的内部API（例如git贡献可能取决于 `vs/workbench/contrib/search/common/search.ts`）
*   一个贡献永远都不能进入另一个贡献的内部（内部是贡献文件中不存在于单个公共API文件中的其他内容）
*   在让一个贡献依赖于另一个贡献之前要三思：这是否确实需要，是否有意义？是否可以通过使用工作台的可扩展性来避免这种依赖性呢？