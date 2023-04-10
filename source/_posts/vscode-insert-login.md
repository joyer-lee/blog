---
title: vscode启动时嵌入登录页面
tags: []
id: '123'
categories:
  - - vscode源码解读
date: 2020-12-08 09:59:25
---



## 任务

基于vscode源码实现一个登陆页面。

## 思路

根据[启动流程](https://cloud.tencent.com/developer/article/1454979 "启动流程")，在启动之前判断是否需要登陆，若需要弹出一个登陆页面，登陆验证成功后进入应用，否则弹出错误提示，若验证超过三次退出应用。 1.找到合适的启动程序位置。在启动应用之前，判断是否需要登陆。 2.在执行启动程序位置编写登陆程序。

## 实现

### 确定登陆逻辑合适的编写位置

在确定登陆逻辑位置之前，先复习下vscode源码的启动流程：

*   入口是 src/main.js
*   主进程（Main Process） 的实际调用路径是: `main.js -> vs/code/electron-main/main.ts -> vs/code/electron-main/window.ts` 在 `window.ts` 启动了一个`BrowserWindow` 加载了 `vs/code/electron-browser/workbench/workbench.html`
*   渲染进程（Renderer Process）的实际路径是： `vs/code/electron-browser/workbench/workbench.html -> vs/code/electron-browser/workbench/workbench.js`

我们要做的是，在程序主进程渲染之前，判断并弹出一个登陆窗口。需要使用到electron的API，结合vscode源码的环境划分，我们需要把逻辑写在vs/code/electron-main/main.ts文件下。

### 登陆逻辑实现

观察ts文件代码： ![](../images/uploads/sites/2/2020/12/image-20201207195618170.png) 发现主要代码写在CodeMain类里。 首先贴上我的源码实现：

```typescript
class CodeMain {
    main(loginWindow?:BrowserWindow):void {
        if (!this.token) {
            this.createWindow();
        } else {
            // Launch
            this.startup(args).then(()=>{
                if(loginWindow){
                    loginWindow.close();
                }
            });
        }
    }

    private token = false;
    //创建登录窗口
    private createWindow(): void {
        //子窗口
        let loginWindow = new BrowserWindow({
            width: 600,
            height: 800,
            frame: false,
            webPreferences: {
                nodeIntegration: true
            }
        });
        //加载页面
        loginWindow.loadFile('src/vs/code/browser/workbench/login.html');
        ipcMain.on('exit', function (event) {
            loginWindow.close();
        });
        //设置一个变量记录登录次数
        let count = 0;
        //从页面接收用户输入的账号密码
        ipcMain.on('login', (event, username, password) => { //如果没有数据需要传，可以不写参数
            //判断账号密码
            if (username === '123' && password === '123') {
                //登陆成功
                //隐藏登录窗口(loginWindow.close();无法启动主窗口)
                dialog.showMessageBox({
                    type: 'info',
                    title: '登录成功',
                    message: '登录成功',
                    buttons:['进入应用']
                }).then(()=>{
                    //更改标签，调用主窗口
                    this.token = true;
                    // Main Startup
                    this.main(loginWindow);
                });
            } else {
                //登录次数加 1
                count++;
                //超过三次，窗口关闭
                if (count >= 3) {
                    //返回三次登录信息
                    let loginMessage = '3次登陆失败，窗口将在1秒后关闭';
                    loginWindow.webContents.send('loginMessage', loginMessage);
                    //延时三秒关闭窗口
                    setTimeout(() => {
                        loginWindow.close();
                    }, 1000);
                } else {
                    //登陆失败
                    //返回失败信息
                    let loginMessage = '登陆失败！登录次数：' + count + '次，共有3次！';
                    //向窗口发送登陆失败消息
                    loginWindow.webContents.send('loginMessage', loginMessage);
                    //刷新窗口，重新登陆
                    loginWindow.reload();
                }
            }
        });
    }
}
```

代码解析： 1.声明一个验证变量：token 2.创建一个创建登陆窗口的方法：createWindow，内部实现使用了electron主进程api，BrowserWindow、IPC等。 3.登陆成功时需要先等待vscode主进程启动成功才能把loginWindow窗口关闭，否则会导致程序退出。 4.登陆页面的位置也是依据源码环境划分原则，放置在src/vs/code/browser/workbench目录下。

### 登陆优化计划

在页面运行成功后，添加login菜单选项，触发调起登陆页面。