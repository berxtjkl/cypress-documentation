---
title: 调试工具
---

{% note info %}
# {% fa fa-graduation-cap %} 通过这篇文档你将会学习到

- Cypress如何在同一个事件循环中运行你的代码, 并且保持调试简单易懂
- Cypress如何接受标准的开发工具
- 如何以及何时使用`调试工具`已经{% url `.debug()` debug %}速记命令
- 如何解决Cypress本身的问题
{% endnote %}

# 使用`调试器`

你的Cypress测试代码运行在与应用程序相同的运行循环中.这意味着你可以访问页面上运行的代码, 以及浏览器为你提供的东西, 比如`document`, `window`等等, 当然也包括`调试器`.

## 像往常一样调试

基于这些陈述, 你可能想在测试中添加一个`调试器`, 就像这样:

```js
it('let me debug like a fiend', function() {
  cy.visit('/my/page/path')

  cy.get('.selector-in-question')

  debugger // 不工作
})
```

这可能并不完全像你所期望的那样工作. 你们可能还记得{% url "Cypress介绍" introduction-to-cypress %}, `cy`命令对稍后要执行的操作排队. 从这个角度来看,你能看出上面的测试将做什么吗?

{% url `cy.visit()` visit %}和{% url `cy.get()` get %}两者都将马上返回, 把他们的工作排在以后要做的队列里, 并且`debugger`会在任何命令实际运行之前执行.

让我们在执行期间使用{% url `.then()` then %}来利用Cypress命令,并且在合适的时机添加一个`debugger`:

```js
it('let me debug when the after the command executes', function () {
  cy.visit('/my/page/path')

  cy.get('.selector-in-question')
    .then(($selectedElement) => {
      // 在cy.visit之后debuger被叫出
      // 并且cy.get命令也完成了
      debugger
    })
})
```

现在我们开始搞正事了! 第一次运行的时候, {% url `cy.visit()` visit %}和{% url `cy.get()` get %}的连锁操作(伴随着附加{% url `.then()` then %}命令)排队等待Cypress执行. `it`块退出, 并且Cypress开始它的工作:

1. 页面被访问, 并且Cypress等待它加载.
2. 元素被查询, 并且如果这个元素不能马上被发现的话,Cypress会自动等待并且重试一会儿.
3. 传递给{% url `.then()` then %}的函数被执行, 执行的元素是上一个命令产生和出来的元素.
4. 在{% url `.then()` then %}函数的上下文中, `debugger`被呼叫, 阻止浏览器并且呼出并聚焦到开发者工具.
5. 现在你进入了开发者工具! 如果你将`debugger`放入应用程序代码中,请检查应用程序的状态.

## 使用 {% url `.debug()` debug %}

Cypress还公开了调试命令的快捷方式, {% url `.debug()` debug %}. 让我们使用这个帮助方法重写上面的测试:

```js
it('let me debug like a fiend', function() {
  cy.visit('/my/page/path')

  cy.get('.selector-in-question')
    .debug()
})
```

当前的主题是由{% url `cy.get()` get %}产生的, 在Developer Tools中作为变量`subject`公开,以便你可以在控制台中与它进行交互.

{% imgTag /img/guides/debugging-subject.png "Debugging Subject" %}

使用{% url `.debug()` debug %}在测试期间快速检查你的应用程序的任何（或许多!）部分. 你可以将它连接到任何Cypress的命令链,以查看当时系统的状态.

# 使用开发人员工具

虽然Cypress已经建成了{% url "优秀的测试运行器" test-runner %}为了帮助你了解应用程序和测试中发生的情况,浏览器团队在其内置开发工具上所做的所有令人惊叹的工作都无法取代. 再一次,我们看到Cypress与现代生态系统流的相配,选择尽可能好的方式利用这些工具吧.

{% note info %}
## {% fa fa-video-camera %} 在行动观察它!

你可以{% url "在我们的React教程系列的这一部分中" https://vimeo.com/242961930#t=264s %}看到从Cypress调试一些应用程序代码的演练.
{% endnote %}

## 获取命令的控制台日志

所有Cypress的命令,当点击{% url "命令日志" test-runner#Command-Log %}的时候, 打印有关命令,其主题及其产生的结果的额外信息. 尝试打开开发人员工具,点击命令日志! 你可以在这里找到一些有用的信息.

### 当点击`.type()`命令时, Developer Tools控制台输出以下内容:

{% imgTag /img/api/type/console-log-of-typing-with-entire-key-events-table-for-each-character.png "Console Log type" %}

# Cypress排除故障

有时你会遇到Cypress本身的错误或意外行为. 在这种情况下, 我们建议你**首先**检查这些支持资源.

## 支持渠道

- 与我们的{% url "Gitter" https://gitter.im/cypress-io/cypress %}社区联系
- 搜索现有的{% url "GitHub问题" https://github.com/cypress-io/cypress/issues %}
- 搜索此文档(搜索框位于右上方) 😉
- 搜索{% url "Stack Overflow" https://stackoverflow.com/questions/tagged/cypress %}的相关答案
- 如果你的组织注册了我们的一个{% url "有偿计划" https://www.cypress.io/pricing/ %}, 你可以获得专门的电子邮件支持,从而为你提供我们团队的一对一帮助.
- 如果你还没有找到解决方案,{% url "打开一个问题" https://github.com/cypress-io/cypress/issues/new %}并且*包含一个可重复的例子*.

## 隔离问题

调试失败的测试时,请遵循以下一般原则来隔离问题:

- 看一下{% url "视频录制和截图" screenshots-and-videos %}.
- 将大型spec文件拆分为较小的文件.
- 将长测试分成较小的测试.
- 使用运行相同的测试{% url '`--browser chrome`' command-line#cypress-run-browser-lt-browser-name-or-path-gt %}. 问题可能是与Electron浏览器有关.
- 如果隔离到Electron浏览器. 在Electron和Chrome中运行相同的测试,然后比较屏幕截图/视频. 查找并隔离命令日志中的任何差异.

{% partial chromium_download %}

## 清除Cypress缓存

如果你在安装Cypress期间遇到问题. 尝试清除Cypress缓存的内容.

这将清除可能在你的计算机上缓存的所有已安装的Cypress版本.

```shell
cypress cache clear
```

运行此命令后,在再次运行Cypress之前, 你将需要运行`cypress install`.

```shell
npm install cypress --save-dev
```

## 启动浏览器

Cypress试图{% url '自动为你找到已安装的Chrome版本' launching-browsers %}. 但是,在不同环境中探测浏览器可能容易出错. 如果Cypress找不到浏览器,但你知道你已安装了它, 有办法确保Cypress可以"看到"它.

{% note info Using the `--browser` command line argument %}
你也可以提供`--browser`命令行参数,用于从已知文件系统路径启动浏览器以绕过浏览器自动检测. {% url "有关详细信息,请参阅'启动浏览器'" launching-browsers#Launching-by-a-path % } %}
{% endnote %}

要从浏览器启动器查看调试日志,请运行Cypress`DEBUG`环境变量, 并设置为`cypress:launcher`.

### Mac

在Mac上, Cypress试图通过捆绑标识符查找已安装的浏览器. 如果这不成功, 它将回归Linux浏览器检测方法.

浏览器名称| 预期的捆绑标识符| 预期的可执行文件
--- | --- | ---
`chrome` | `com.google.Chrome` | `Contents/MacOS/Google Chrome`
`chromium` | `org.chromium.Chromium` | `Contents/MacOS/Chromium`
`canary` | `com.google.Chrome.canary` | `Contents/MacOS/Google Chrome Canary`

### Linux

在Linux上, Cypress会对于许多不同的二进制名称扫描你的`路径`. 如果你尝试使用的浏览器不存在于其中一个预期的二进制名称下,则Cypress将无法找到它.

浏览器名称| 预期的二进制名称
--- | ---
`chrome` | `google-chrome`, `chrome`, 或 `google-chrome-stable`
`chromium` | `chromium-browser` 或 `chromium`
`canary` | `google-chrome-canary`

这些二进制名称应适用于大多数Linux发行版. 如果你的发行版使用不同的二进制名称打包浏览器,则可以使用预期的二进制名称添加符号链接,以便Cypress可以检测到它.

举例来说, 如果你的发行版将Google Chrome打包为`chrome`, 你可以像这样添加一个符号链接`google-chrome`:

```shell
sudo ln `which chrome` /usr/local/bin/google-chrome
```

### Windows

在Windows上, Cypress扫描以下位置以尝试查找每个浏览器:

浏览器名称| 预期的路径
--- | ---
`chrome` | `C:/Program Files (x86)/Google/Chrome/Application/chrome.exe`
`chromium` | `C:/Program Files (x86)/Google/chrome-win32/chrome.exe`
`canary` | `%APPDATA%/../Local/Google/Chrome SxS/Application/chrome.exe`

要自动检测在不同路径上安装的浏览器, 使用`mklink`在Cypress希望找到你的浏览器的位置创建符号链接.

{% url '阅读有关在Windows上创建符号链接的更多信息' https://www.howtogeek.com/howto/16226/complete-guide-to-symbolic-links-symlinks-on-windows-or-linux/ %}

## Chrome扩展程序白名单

Cypress在测试运行器中使用Chrome扩展程序以便正常运行. 如果你或你的公司将特定的Chrome扩展程序列入白名单, 这可能会导致运行Cypress的问题. 你需要让管理员将下面的Cypress扩展ID列入白名单:

```sh
caljajdfkjjjdehjdoimjkkakekklcck
```

## 清除应用数据

Cypress维护一些本地应用程序数据,以便保存用户首选项并更快地启动. 有时这些数据可能会被破坏. 你可以通过清除此应用数据来解决你遇到的问题.

### 清除应用数据

1. 打开Cypress`cypress open`
2. 去`File` -> `View App Data`
3. 这将带你进入文件系统中存储App Data的目录. 如果你打不开Cypress, 在文件系统中搜索名为`cy`的目录, 其内容应该是这样的:

  ```
  📂 production
    📄 all.log
    📁 browsers
    📁 bundles
    📄 cache
    📁 projects
    📁 proxy
    📄 state.json
  ```
4. 删除`cy`文件中的所有内容
5. 关闭Cypress并再次打开它

## 逐步完成测试命令

你可以使用命令运行test命令{% url `.pause()` pause %}命令.

```javascript
it('adds items', function () {
  cy.pause()
  cy.get('.new-todo')
  // 更多命令
})
```

这允许你检查Web应用程序, DOM和网络, 每个命令后的任何存储,以确保一切按预期发生.

## 打印DEBUG日志

Cypress是用{% url 'debug' https://github.com/visionmedia/debug %}模块. 这意味着你可以通过运行Cypress来启用有用的调试输出. **备注:** 运行`DEBUG=...`设置时你会看到很多消息.

**在Mac或Linux上:**

```shell
DEBUG=cypress:* cypress open
```

**在Windows上:**

```shell
set DEBUG=cypress:*
```

```shell
cypress open
```

阅读更多{% url '关于这里的CLI选项' command-line#Debugging-commands %}和{% url "良好记录" https://glebbahmutov.com/blog/good-logging/ %}的博客文章.

### 详细日志

有几个层次的`DEBUG`信息

```shell
# 打印很少的顶级消息
DEBUG=cypress:server ...
# 从服务器包打印所有消息
DEBUG=cypress:server* ...
# 仅从配置解析中打印消息
DEBUG=cypress:server:config ...
```

这使你可以更好地隔离问题

### Debug登录浏览器

如果在`cypress open`这期间看到问题, 你也可以在浏览器中打印调试日志. 打开浏览器的开发者工具和设置一个`localStorage`属性:

```javascript
localStorage.debug = 'cypress*'

// 禁用调试消息
delete localStorage.debug
```

重新加载浏览器并在Developer Tools控制台中查看调试消息. 你只会看到"cypress:driver"在浏览器中运行的包日志, 你可以在下面看到.

{% imgTag /img/api/debug/debug-driver.jpg "Debug logs in browser" %}

## 记录Cypress事件

除了`DEBUG`消息, Cypress还会发出你可以收听的多个事件,如下所示. {% url '在此处阅读有关在浏览器中记录事件的更多信' catalog-of-events#Logging-All-Events %}.

{% imgTag /img/api/catalog-of-events/console-log-events-debug.png "console log events for debugging" %}

## 在测试之外运行Cypress命令

如果需要直接从Developer Tools控制台运行Cypress命令, 你可以使用内部命令`cy.now('命令名称', ...参数)`. 例如, 运行例如`cy.task('数据库', 123)`这种正常执行命令链之外的命令:

```javascript
cy.now('task', 123)
  .then(console.log)
// 运行cy.task(123)并打印已解析的值
```

{% note warning %}
`cy.now()`命令是一个内部命令,将来可能会更改.
{% endnote %}

## 附加信息

### 将命令日志写入终端

你可以包含该插件[cypress-failed-log](https://github.com/bahmutov/cypress-failed-log)在你的测试中. 如果测试失败,此插件会将Cypress命令列表写入终端以及JSON文件.

{% imgTag /img/api/debug/failed-log.png "cypress-failed-log terminal output" %}

# Cypress之hack操作

如果你想深入Cypress并自己编辑代码, 你可以做到这一点. Cypress的代码是开源的, 并在一个版本{% url "MIT执照" https://github.com/cypress-io/cypress/blob/develop/LICENSE %}下获得许可. 我们在下面概述了一些入门提示. 

## 贡献

如果你想直接参与Cypress代码, 我们很乐意得到你的帮助！ 请查看我们的{% url "贡献引导" https://github.com/cypress-io/cypress/blob/develop/CONTRIBUTING.md %}来了解你可以贡献的多种方式. 

## 单独运行Cypress应用程序

Cypress附带了一个解析参数的npm CLI模块, 启动Xvfb服务器(如有必要), 然后打开构建于之上的{% url "Electron" https://electronjs.org/ %}测试运行器. 关于你为什么要这样做的一些常见情况是:

- 调试Cypress没有启动或挂起
- 调试与npm CLI模块解析CLI参数的方式相关的问题

以下是在没有npm CLI模块的情况下直接启动Cypress应用程序的方法. 首先, 使用{% url "`cypress cache path`" command-line#cypress-cache-path %}命令找到二进制文件的安装位置.

例如, 在Linux机器上:

```shell
npx cypress cache path
/root/.cache/Cypress
```

然后, 尝试进行冒烟测试, 验证应用程序是否具有主机上存在的所有必需依赖项:

```shell
/root/.cache/Cypress/3.3.1/Cypress/Cypress --smoke-test --ping=101
101
```

如果缺少依赖项, 应用程序应该打印一条错误消息. 你可以通过设置{% url "环境变量 ELECTRON_ENABLE_LOGGING" https://electronjs.org/docs/api/environment-variables %}来查看Electron详细日志消息:

```shell
ELECTRON_ENABLE_LOGGING=true DISPLAY=10.130.4.201:0 /root/.cache/Cypress/3.3.1/Cypress/Cypress --smoke-test --ping=101
[809:0617/151243.281369:ERROR:bus.cc(395)] Failed to connect to the bus: Failed to connect to socket /var/run/dbus/system_bus_socket: No such file or directory
101
```

如果冒烟测试无法执行, 检查是否缺少共享库(Linux机器上的常见问题, 没有所有Cypress依赖项).

```shell
ldd /home/person/.cache/Cypress/3.3.1/Cypress/Cypress
	linux-vdso.so.1 (0x00007ffe9eda0000)
	libnode.so => /home/person/.cache/Cypress/3.3.1/Cypress/libnode.so (0x00007fecb43c8000)
	libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fecb41ab000)
	libgtk-3.so.0 => not found
	libgdk-3.so.0 => not found
  ...
```

**小建议:** 使用{% url "Cypress Docker image" docker %}或者通过从我们的官方Docker镜像中复制它们来安装依赖项.

**注意:** 详细电子记录可能会显示仍然允许Cypress正常工作的警告. 例如, 尽管下面有可怕的输出, 但Cypress测试跑步者正常打开:

```shell
ELECTRON_ENABLE_LOGGING=true DISPLAY=10.130.4.201:0 /root/.cache/Cypress/3.3.1/Cypress/Cypress
[475:0617/150421.326986:ERROR:bus.cc(395)] Failed to connect to the bus: Failed to connect to socket /var/run/dbus/system_bus_socket: No such file or directory
[475:0617/150425.061526:ERROR:bus.cc(395)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
[475:0617/150425.079819:ERROR:bus.cc(395)] Failed to connect to the bus: Could not parse server address: Unknown address type (examples of valid types are "tcp" and on UNIX "unix")
[475:0617/150425.371013:INFO:CONSOLE(73292)] "%cDownload the React DevTools for a better development experience: https://fb.me/react-devtools
You might need to use a local HTTP server (instead of file://): https://fb.me/react-devtools-faq", source: file:///root/.cache/Cypress/3.3.1/Cypress/resources/app/packages/desktop-gui/dist/app.js (73292)
```

运行测试运行器, 你还可以看到详细的Cypress日志

```shell
DEBUG=cypress* DISPLAY=10.130.4.201:0 /root/.cache/Cypress/3.3.1/Cypress/Cypress --smoke-test --ping=101
cypress:ts Running without ts-node hook in environment "production" +0ms
cypress:server:cypress starting cypress with argv [ '/root/.cache/Cypress/3.3.1/Cypress/Cypress', '--smoke-test', '--ping=101' ] +0ms
cypress:server:args argv array: [ '/root/.cache/Cypress/3.3.1/Cypress/Cypress', '--smoke-test', '--ping=101' ] +0ms
cypress:server:args argv parsed: { _: [ '/root/.cache/Cypress/3.3.1/Cypress/Cypress' ], smokeTest: true, ping: 101, cwd: '/root/.cache/Cypress/3.3.1/Cypress/resources/app/packages/server' } +7ms
cypress:server:args options { _: [ '/root/.cache/Cypress/3.3.1/Cypress/Cypress' ], smokeTest: true, ping: 101, cwd: '/root/.cache/Cypress/3.3.1/Cypress/resources/app/packages/server', config: {} } +2ms
cypress:server:args argv options: { _: [ '/root/.cache/Cypress/3.3.1/Cypress/Cypress' ], smokeTest: true, ping: 101, cwd: '/root/.cache/Cypress/3.3.1/Cypress/resources/app/packages/server', config: {}, pong: 101 } +1ms
cypress:server:appdata path: /root/.config/Cypress/cy/production +0ms
cypress:server:cypress starting in mode smokeTest +356ms
101
cypress:server:cypress about to exit with code 0 +4ms
```

## 编辑已安装的Cypress代码

已安装的测试运行器配有完全转换后的版本, 你可以破解未经过混淆的JavaScript源代码. 你可能希望直接修改已安装的测试运行器代码:

- 调查难以重现的机器上发生的错误
- 在打开拉取请求之前更改Cypress的运行时行为
- 玩得开心🎉

首先, 使用{% url "`cypress cache path`" command-line#cypress-cache-path %}命令打印安装二进制文件的位置.

例如, 在Mac上:

```shell
npx cypress cache path
/Users/jane/Library/Caches/Cypress
```

然后, 在任何代码编辑器中打开以下路径的源代码. 一定要对要编辑的测试运行器的所需版本进行替换`3.3.1`字样的操作.

```text
/Users/jane/Library/Caches/Cypress/3.3.1/Cypress.app/Contents/Resources/app/packages/
```

你可以在JavaScript代码中更改任何内容:

{% imgTag /img/guides/source-code.png "Source code of the 测试运行器 in a text editor" %}

当结束之时, 如有必要, 删除已编辑的测试运行器版本并重新安装Cypress官方版本以返回官方发布的代码.

```shell
rm -rf /Users/jane/Library/Caches/Cypress/3.3.1
npm install cypress@3.3.1
```

