---
title: vscode碎片化解读
tags: []
id: '109'
categories:
  - - vscode源码解读
---

\[toc\]

## 关于渲染

1.vs\\base\\browser\\ui\\splitview 切分上中下 2.vs\\base\\browser\\ui\\grid 中间的拆分 3.菜单栏 \\src\\vs\\workbench\\browser\\workbench.ts 窗口工作面板的功能相关

```javascript
src\vs\workbench\browser\layout.ts
initLayout{
    registerLayoutListeners(){
        //Listeners
        this.registerLayoutListeners();
    }
}
```

注册方法

```typescript
src\vs\workbench\services\title\electron-sandbox\titleService.ts
import { registerSingleton } from 'vs/platform/instantiation/common/extensions';
import { TitlebarPart } from 'vs/workbench/electron-sandbox/parts/titlebar/titlebarPart';
import { ITitleService } from 'vs/workbench/services/title/common/titleService';

registerSingleton(ITitleService, TitlebarPart);


src\vs\workbench\browser\parts\titlebar\titlebarPart.ts
createContentArea{
   this.installMenubar();
}
```

## 关于服务（service）

这里通过 createService 创建一些基础的 Service 运行环境服务 EnvironmentService src/vs/platform/environment/node/environmentService.ts 通过这个服务获取当前启动目录，日志目录，操作系统信息，配置文件目录，用户目录等。 日志服务 MultiplexLogService src/vs/platform/log/common/log.ts 默认使用控制台日志 ConsoleLogMainService 其中包含性能追踪和释放信息，日志输出级别 配置服务 ConfigurationService src/vs/platform/configuration/node/configurationService.ts 从运行环境服务获取内容 生命周期服务 LifecycleService src/vs/platform/lifecycle/common/lifecycleService.ts 监听事件，electron app 模块 比如：ready， window-all-closed，before-quit 可以参考官方electron app 文档 状态服务 StateService src/vs/platform/state/node/stateService.ts 通过 FileStorage 读写 storage.json 存储，里记录一些与程序运行状态有关的键值对 请求服务 RequestService src/vs/platform/request/browser/requestService.ts 这里使用的是原生 ajax 请求，实现了 request 方法 主题服务 ThemeMainService src/vs/platform/theme/electron-main/themeMainService.ts 这里只设置背景颜色，通过 getBackgroundColor 方法 IStateService 存储 签名服务 SignService src/vs/platform/sign/node/signService.ts      

## 顶部菜单的触发

设置一级的枚举\\src\\vs\\platform\\actions\\common\\actions.ts 添加二级的函数\\src\\vs\\workbench\\browser\\actions\\windowActions.ts 注：菜单实际触发方式是通过命令出发 命令:\\src\\vs\\workbench\\contrib\\files\\browser\\fileCommands.ts