---
title: vscode初次上手-添加导航栏选项
tags: []
id: '113'
categories:
  - - 未分类
---

1.搜索导航一级菜单`"File"`字串涉及的目录有 2.搜索导航二级菜单`"New File"`字串涉及目录有 - src\\vs\\workbench\\contrib\\files\\browser\\fileActions.contribution.ts

> 关于file的所有菜单选项都在这里，包括`go`导航里的`go to file`选项 - src\\vs\\workbench\\contrib\\files\\browser\\fileActions.ts 第54行 语句`export const NEW_FILE_LABEL = nls.localize('newFile', "New File");`对应资源管理器右键`"New File"`字串 第91行 语句`static readonly LABEL = nls.localize('createNewFile', "New File");`修改字符后没发现页面变化 该页面对应文件导航选项的触发事件 - src\\vs\\workbench\\contrib\\welcome\\page\\browser\\vs\_code\_welcome\_page.ts 该页面导出欢迎页面HTML内容

3.搜索"new &&Window"字串涉及目录有 1.src\\vs\\platform\\menubar\\electron-main\\menubar.ts

> 存在判断mac机时的，导航操作显示处理

2.src\\vs\\workbench\\browser\\actions\\windowActions.ts

> 注册部分导航菜单

src\\vs\\platform\\menubar\\electron-main\\menubar.ts 添加`Test`一级菜单 src\\vs\\workbench\\browser\\parts\\titlebar\\menubarControl.ts