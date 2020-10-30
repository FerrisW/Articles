## 前言  
因项目需求，需要对现有Electron app项目的教师端和学生端在现有的两者直接异步实时通信的基础上，加一层教师端-->主进程-->学生端三者之间的异步通信，确保教师端能够实时控制学生端。由于教师端和学生端是以嵌入在主进程的webview的方式来运行，因此本次需要实现的是webview与渲染进程之间的通信。  
### 关键点  
**app、ipcMain、ipcRenderer、BrowserWindow、webview、如何处理学生端同时收到教师端及主进程的webview发送的信息**  
### 相关概念  
**BrowserWindow：用于创建和控制浏览器窗口**  
### 注意点  
1.默认情况在Electron版本大于等于5时，webview标签是无法使用的，若想使用，则需要在构造BrowserWindow时设置webPreferences选项来启用  
    
    /* main.js */
    const electron = require('electron');
    const { app, BrowserWindow } = electron;
    //app控制应用程序的事件生命周期
    app.on("ready", () => {
        //当Electron完成初始化时，将会执行一次
        //...
    });
    app.on("window-all-closed", () => {
        //当所有的窗口都被关闭时触发
        //...
        app.quit();  //在应用程序退出时发出（不包括window系统中系统关机、重启、注销当前用户的情况，此时该事件不 会被触发）
    })
    app.commandLine.appendSwitch("disable-http-cache");
    //app.commandLine: 操作Chromium读取应用程序的命令行参数
    //appendSwitch(switcj[, value]) 通过可选的参数value给Chromium中添加一个命令行开关
    const mainWindow = new BrowserWindow({
        webPreferences: {
            webviewTag: true,  //Boolean, 决定是否启用webview标签。默认值为false
            nodeIntegration: true,  //是否启用node集成，默认值为false
            nodeIntegrationInWorker: true,  //是否在Web工作器中启用node集成，默认值为false
            zoomFactor: 1.0,  //页面默认的缩放比例
            enableRemoteModule: true,  //是否启用远程模块，默认值为false
        }
    });
    mainWindow.openDevTools();   //控制是否在加载成功时打开开发者工具
    mainWindow.loadURL("file://" + __dirname + "/index.html");  //初始化请求加载的页面地址，可以是远程地址，也可以是file:// 协议的本地HTML文件的路径  

2.webview标签的使用  
     
    /* 若要在应用中嵌入网页，则可将webview标签添加至应用程序的被嵌入页面中*/
    /* index.html */
    <webview
        id="webview"
        src="https://www.github.com/"
        style="样式"
        nodeIntegration  //设置后webview的访客页将具有node集成，即访客页可调用webview所在页面的方法【访客页 即webview的src中指定的页面】
        nodeIntegrationInSubFrames  //设置后在web视图中的子frame中启用nodeJS支持
    ></webview>
    /* 若想控制嵌入内容，可在被嵌入页面中添加针对webview的监听事件 */
    <script>
        onload = () => {
            const webview = document.getElementBtId('webview');
            webview.addListener('需监听的事件', () => {
                //...
            })
            webview.openDevTools();  //打开webview内嵌页面的调试工具
        }
    </script>  

3.实现内嵌页面（渲染进程）和webview之间的通信  
     
    //被嵌入页面
    //在被嵌入页面中可通过监听'ipc-message'来实现对嵌入页面（渲染进程）发送事件的监听
    webview.addEventListener('ipc-message', (event) => {
        console.log(event.channel);  //Print 'qqq'
    })
    webview.send('webview-send-channel', 'ppp')

    //嵌入页面
    const { ipcRenderer } = require('electron');
    ipcRenderer.on('webview-send-channel', (event, args) => {
        ipcRenderer.sendToHost('qqq')
    })
    //同ipcRenderer.send不同的是，ipcRenderer.sendToHost会被发送到host页面上的webview元素，而非主进程

4.webview与web端（渲染进程）、客户端（主进程）的通信。实例链接：https://blog.csdn.net/weixin_42333548/article/details/91946334  

5.如何处理学生端同时收到教师端及主进程的webview发送的信息  
     
     //针对该问题，可在老师端在分别发送至学生端和主进程的消息对象中添加一个hash字段，借助随机生成的一个由数字和字母组成的字符串，然后在学生端收到来自教师端和主进程的消息时，在本页面利用一个数组来存放该hash值，若数组中不包含当前hash值，则说明没有任何一方的信息到达学生端，则处理该信息；若已存在，则说明已有一方的信息发送至学生端并已处理，此时收到的信息无需再次处理，直接return即可

     //随机生成由数字和字母组成的字符串（5或6位长度的字符串）：
     Math.random().toString(36).substr(7);  


参考链接：  
1.[Electron官方文档之ipcMain](http://www.electronjs.org/docs/api/ipc-main)  
2.[Electron官方文档之ipcRenderer](http://www.electronjs.org/docs/api/ipc-renderer)  
3.[Electron官方文档之webview](http://www.electronjs.org/docs/api/webview-tag)  
4.[Electron官方文档之BrowserWindow](http://www.electronjs.org/docs/api/browser-window)  
5.[Electron官方文档之app](http://www.electronjs.org/docs/api/app)  