---
title: vscode项目国际化
tags: []
categories:
  - - 工作日志
date: 2023-4-14 17：30：00
---

## 为什么要进行国际化

国际化（i18n）是指将应用程序设计为可以轻松地适应不同语言、地区和文化的能力，更好的在海外推广国内产品。

## 国际化功能要点
1.多语言支持：包括界面和文档翻译、本地化应用等
2.多语言环境切换：切换后，读取相应的语言包内容

## 如何实现国际化（步骤）

1.将所有需要翻译的字符串放在单独的消息文件中。如 json 格式文件，每个文件对应一个语言。 
2.使用国际化框架或库如`vue-i18n`，这些框架可帮助管理不同语言的消息文件。 
3.根据用户的语言环境动态加载适当的消息文件。

## 结合自身项目特点实现国际化

我们的产品为多项目架构，主项目基于 vscode 源码实现，包含 vscode 扩展项目、webview 界面，包含三种情况：

### vscode 原生项目国际化
安装时可选环境
- 设置默认语言环境
  初始安装时：默认语言环境为本机语言环境、若没有相应的语言包，默认为 en 
  
- 扩展中文包消息字段
  语言包目录：`extensions\ms-ceintl.vscode-language-pack-zh-hans-1.55.1\translations\main.i18n.json`
  ```json
  	"vs/workbench/browser/parts/headerbar/modelLibraryView": {
  		"loadModel": "载入模型"
  	}
  ```
- 本地化应用

```typescript
import * as nls from 'vs/nls';

const test1: IAction = {
  id: '1',
  label: nls.localize('loadModel', 'load model'), // 本地化
  run: async () => {
    console.log('载入模型 run');
    this.loadModelToScene(item);
    rename;
  },
  dispose() {},
};
```
- 语言环境切换方法
可以通过使用Configure Display Language命令显式设置 VS Code 显示语言来覆盖默认 UI 语言，按Ctrl+Shift+P调出命令面板 > 键入“display” > 显示Configure Display Language命令 

底层实现：
```typescript
// src\vs\workbench\contrib\localizations\browser\localizationsActions.ts
	if (selectedLanguage) {
				await this.jsonEditingService.write(this.environmentService.argvResource, [{ path: ['locale'], value: selectedLanguage.label }], true); // 写入运行时配置文件中 路径：C:\Users\userName\.modouide-oss
				const restart = await this.dialogService.confirm({
					type: 'info',
					message: localize('relaunchDisplayLanguageMessage', "A restart is required for the change in display language to take effect."),
					detail: localize('relaunchDisplayLanguageDetail', "Press the restart button to restart {0} and change the display language.", this.productService.nameLong),
					primaryButton: localize('restart', "&&Restart")
				});

				if (restart.confirmed) {
					this.hostService.restart();
				}
			}
```
### 基于vscode的子项目国际化
> 语言切换需要重启项目才可生效
  browerview项目可通过界面查询字符获取locale值
  扩展项目可通过通信访问配置文件中的locale值


## vue 项目国际化

1.使用 vue-i18n 插件实现 
2.切换语言环境:可以使用 $i18n.locale 属性来切换语言


## 问题
### 后端返回的数据仅中文，如何兼容？