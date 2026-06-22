# 手机连接 Codex 或 Claude (CLI) 完整指南

随着 Claude Code 和 Codex 等命令行 AI 编程工具的发布，开发者们拥有了更强大的本地开发助手。然而，由于这些工具本质上是运行在电脑端的 CLI 程序，如何在手机或平板等移动设备上便捷地连接并使用它们，成为了许多人的痛点。

本指南将为您介绍目前主流的三种连接方案，您可以根据自己的使用习惯和技术背景进行选择。

---

## 方案一：官方 Remote Control (最推荐)

Anthropic 官方为 Claude Code 推出了 **Remote Control** 功能，允许用户从手机端直接同步并控制桌面端的 CLI 会话。

### 1. 使用前提
- 拥有 Claude Pro、Max、Team 或 Enterprise 订阅。
- 电脑端已安装最新版本的 `claude-code`。
- 如果你的网络需要代理，请先确认本机代理端口；下面的代理命令需要改成你自己的代理端口。

### 2. 操作步骤
1.  **启动远程模式**：在电脑项目目录下运行命令：
    ```bash
    claude remote-control
    ```
    或者在已有的 Claude 会话中输入 `/remote-control` (或简写 `/rc`)。
    如果需要通过代理启动，可以参考下面的写法，并把 `PROXY_PORT` 换成你自己的代理端口：
    ```bash
    HTTP_PROXY=http://127.0.0.1:PROXY_PORT HTTPS_PROXY=http://127.0.0.1:PROXY_PORT claude remote-control
    ```
2.  **获取链接**：终端会显示一个专用的 URL 和一个二维码。
3.  **移动端登录**：
    - 使用手机扫描二维码。
    - 或者在手机浏览器访问 `claude.ai/code` 并登录相同账号。
4.  **开始操作**：此时，您的手机端将与电脑端实时同步，您可以在手机上发送语音或文本指令，控制电脑进行代码修改、文件阅读和命令执行。

---

## 方案二：第三方专用应用 (移动端体验优化)

如果您觉得在移动端浏览器操作不够灵敏，或者需要更原生的交互体验，可以使用社区开发的专用 App。

### 1. Happy Coder (跨平台)
**Happy Coder** 是一款专为 Claude Code 和 Codex 打造的移动端中继应用。
- **优点**：提供加密中继服务，自带针对代码输入的虚拟键盘优化，支持 iOS 和 Android。
- **使用**：在电脑端运行其提供的配套服务端脚本，然后在手机 App 中输入配对码即可。

第一步：去应用市场搜索“happy”，一定要要记住这个H的图标。
第二步：要保证你的电脑有nodejs，支持nodejs 18+环境，安装下面的命令：




当然，官方说是所有的通信都是匿名加密，不会获取到信息，
安装

安装 HappyCoding CLI


 ```js
 npm install -g happy-coder
```

2.  首次登录/绑定

```js
 happy auth login

 happy auth login --force
```

3.  启动模式

```js
happy          # Claude 模式
happy codex    # Codex 模式
```

2. 后台进程管理

启停与状态


```js
happy daemon start
happy daemon stop
happy daemon status
happy daemon list
```

2.  健康检查与诊断

 ```js
 happy doctor
```

3.  清理异常残留进程


 ```js
 happy doctor clean
```

3. 卸载
推荐按“先停服务，再卸载，再清数据”：

停止后台与清理进程

```js
happy daemon stop
happy doctor clean
```
2.  退出登录并清认证

```js
happy auth logout
```

3.  卸载全局包

```js 
 npm uninstall -g happy-coder
 ```
