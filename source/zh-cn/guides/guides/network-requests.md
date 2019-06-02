---
title: 网络请求
---

{% note info %}
# {% fa fa-graduation-cap %} 您将学习

- Cypress如何使用{% url `cy.route()` route %}来中断后端
- 当存根网络请求时，我们做出了哪些权衡
- Cypress如何在命令日志中可视化网络管理
- 如何使用固件去复用XHR响应
- 如何使用别名去引用XHR请求并等待它们
- 如何编写抗片状的声明性测试
{% endnote %}

# 测试策略

Cypress使在应用程序内测试Ajax/XHR请求的整个生命周期变得更加容易。Cypress提供了对XHR对象的直接访问，允许您使用其属性进行断言。此外，您甚至可以存根和模拟请求的响应。

{% partial network_stubbing_warning %}

***常见的测试场景：***

- 对请求的主体进行断言
- 对请求的url进行断言
- 对请求的头部进行断言
- 存根响应的主体
- 存根响应的状态码
- 存根相应的头部
- 延迟响应
- 等待响应的发生

在Cypress中，您可以选择是存根响应，还是允许它们实际访问您的服务器。您还可以在同一个测试中通过选择存根某些请求，同时允许其他请求访问您的服务器来进行混合搭配。

让我们来研究一下这两种策略：为什么您会使用其中一种而不是另一种，以及为什么您要经常使用这两种策略。

## 不要存根响应

没有存根的请求实际上会到达服务器。通过*不*存根您的响应，您编写的才是真正的“端到端”测试。这意味着您以与真实用户相同的方式驱动应用程序。

> 当请求没有存根时，这将确保客户端和服务器之间的*契约*正常工作。

换句话说，您可以确信您的服务器正在以正确的结构发送正确的数据供您的客户端使用。围绕应用程序的*关键路径*进行*端到端*测试是一个好主意。这些通常包含登录、注册，或其他关键路径，如计费。

***您应该意识到不存根响应的不足之处：***

- 因为没有存根响应，这意味着 **您的服务器必须实际发送真实的响应**。这可能会有问题，因为您可能必须在每一个测试前*播种一个数据库*才能生成状态。例如，如果您要测试*分页*，您必须要为用于复制应用程序中此功能的每一个对象播种数据库。
- 因为真实的响应要经过服务器的每一层（控制器、模型、视图等）所以测试通常比存根响应*慢得多*。

如果您正在写一个传统的服务器端应用程序，多数的响应是`HTML`，可能很少有存根响应。可是，大多数提供`JSON`服务的现代应用程序都可以利用存根。

{% note success 优点 %}
- 保证在生产中工作
- 围绕服务器端点的测试覆盖率
- 适用于传统服务器端的HTML渲染
{% endnote %}

{% note danger 缺点 %}
- 需要播种数据
- 慢得多
- 难以测试边缘用例
{% endnote %}

{% note info 使用建议 %}
- 谨慎使用
- 适用于应用程序的*关键路径*
- 对某个功能的*适当路径*进行测试是有帮助的
{% endnote %}

## 存根响应

存根响应使您能够控制响应的各个方面，包括响应`body`、`status`、`headers`，甚至网络`delay`。存根速度非常快，大多数响应将在小于20毫秒内返回。

> 存根响应是一个控制返回客户端数据的好方法。

您不需要在服务器上做任何操作。您的应用程序将不知道它的请求被存根，因此*不需要更改代码*。

{% note success 优点 %}
- 易于控制响应主体、状态和头部
- 可以强制响应花费更长的时间来模拟网络延迟
- 不对服务器或客户端代码进行任何代码更改
- 响应速度快，小于20毫秒
{% endnote %}

{% note danger 缺点 %}
- 不能保证您的存根响应与服务器发送的实际数据相同
- 某些服务器端点没有测试覆盖
- 如果您正在使用传统的服务器端HTML渲染，则不太适用
{% endnote %}

{% note info 使用建议 %}
- 用于绝大多数测试
- 混合搭配，通常有一个真正的端到端测试，然后存根其余的测试
- 完美适用JSON接口
{% endnote %}

# 存根

Cypress使存根响应和控制响应的`body`、`status`、`headers`，甚至网络`delay`变得很容易。

***要开始存根响应，您需要做两件事。***

1. 启动一个 {% url `cy.server()` server %}
2. 提供一个 {% url `cy.route()` route %}

这两个命令一起工作来控制命令选项中的响应行为。{% url `cy.server()` server %}能够存根, 而{% url `cy.route()` route %}提供一个路由表以便于Cypress了解哪一个响应应该处理哪一个请求。

{% note info %}
关于如何存根响应的使用说明，请查看{% url '`cy.server()` options' server#Options %}和{% url '`cy.route()` options' route#Options %}。
{% endnote %}

# 请求

当应用程序中发生XHR请求时，Cypress会自动显示。这些会记录在命令日志中(不管它是否存根)。Cypress显示请求何时启动和何时完成。此外，Cypress在发出请求时获取DOM快照，在响应返回时获取另一个快照。

![snapshot_request](https://user-images.githubusercontent.com/1271364/26947393-930508b0-4c60-11e7-90a0-4d42ee3f24c0.gif)

默认情况下，Cypress被配置为*忽略*获取静态内容的请求，像`.js`或`.html`文件。这样可以减少命令日志的干扰。可以通过覆盖{% url '`cy.server()` options' server#Options %}的默认白名单来更改此选项。

Cypress自动收集请求的`headers`和请求的`body`并将这些提供给您。

# 路由选择

```javascript
cy.server()           // 使响应存根
cy.route({
  method: 'GET',      // 路由所有GET请求
  url: '/users/*',    // url匹配'/users/*'
  response: []        // 强制响应为：[]
})
```

当您启动一个{% url `cy.server()` server %}并定义{% url `cy.route()` route %}命令时，Cypress会在命令日志中的“Routes”下显示该命令。

{% imgTag /img/guides/server-routing-table.png "Routing Table" %}

只要使用{% url `cy.server()` server %}启动了服务器，测试剩余部分的所有请求都是可控的。当运行一个新的测试时，Cypress会恢复默认行为并且移除所有路由和存根。想要了解关于API和选项的完整介绍，参考各个命令的文档。

- {% url `cy.server()` server %}
- {% url `cy.route()` route %}

# 固件

固件是用于测试的存储在文件中的一组固定数据。测试固件的目的是确保有一个公认且固定的环境来运行测试，以便结果是可重复的。通过调用{% url `cy.fixture()` fixture %}命令，可以在测试中访问固件。

Cypress使存根网络请求变得很容易，并让它立即响应固件数据。

当存根处理响应时，通常需要管理潜在的大型复杂JSON对象。Cypress允许您将固件语法直接集成到响应中。

```javascript
cy.server()

// 我们将响应设置为activite.json固件
cy.route('GET', 'activities/*', 'fixture:activities.json')
```

您还可以在响应中引用{% url 'aliases' variables-and-aliases %}。这些别名不必指向固件，但这是一个常见的用例。分离固件使您能够在将该对象传递给响应之前对其进行操作和修改。

```javascript
cy.server()

cy.fixture('activities.json').as('activitiesJSON')
cy.route('GET', 'activities/*', '@activitiesJSON')
```

## 组织

Cypress会自动搭建一个推荐的文件夹结构，用于在每个新项目中组织固件。默认情况下，当您将项目添加到Cypress时，会创建一个`example.json`文件。

```text
/cypress/fixtures/example.json
```

固件可以在其他文件夹中进一步组织。例如，您可以创建另一个名为`images`的文件夹并添加图像：

```text
/cypress/fixtures/images/cats.png
/cypress/fixtures/images/dogs.png
/cypress/fixtures/images/birds.png
```

要访问`images`文件夹中嵌套的固件，只需在{% url `cy.fixture()` fixture %}命令中引入该文件夹。

```javascript
cy.fixture('images/dogs.png') //以base64编码方式返回dog.png
```

# 等待

无论您是否选择存根响应，Cypress都允许您声明性地使用{% url `cy.wait()` wait %}来处理请求及其响应。

{% note info %}
下面的部分使用了一个{% url 'Aliasing' variables-and-aliases %}的概念。如果您是Cypress的新手，您最好先看看这个。
{% endnote %}

以下是别名路由再等待路由的示例：

```javascript
cy.server()
cy.route('activities/*', 'fixture:activities').as('getActivities')
cy.route('messages/*', 'fixture:messages').as('getMessages')

// 访问dashboard页面将发起与以上两条路由匹配的请求
cy.visit('http://localhost:8888/dashboard')

// 传递一个路由别名数组，迫使Cypress等待，直到它看到与每个别名匹配的请求的响应
cy.wait(['@getActivities', '@getMessages'])

// 在上面的wait命令解析之前，这些命令不会运行
cy.get('h1').should('contain', 'Dashboard')
```

如果希望检查别名路由的每个响应数据，可以使用多个`cy.wait()`调用。


```javascript
cy.server()
cy.route({
  method: 'POST',
  url: '/myApi',
}).as('apiCheck')
cy.visit('/')
cy.wait('@apiCheck').then((xhr) => {
  assert.isNotNull(xhr.response.body.data, '1st API call has data')
})
cy.wait('@apiCheck').then((xhr) => {
  assert.isNotNull(xhr.response.body.data, '2nd API call has data')
})
cy.wait('@apiCheck').then((xhr) => {
  assert.isNotNull(xhr.response.body.data, '3rd API call has data')
})
```

等待别名路由有很大的优势：

1. 测试更加健壮，而薄片更少。
2. 失败消息要精确得多。
3. 可以断言底层XHR对象。

让我们分析一下每一个优势。

## 薄片

声明式等待响应的一个优点是它减少了测试薄片。您可以将{% url `cy.wait()` wait %}看作一个警卫，当您希望发出一个匹配特定路由别名的请求时，它会通知Cypress。这将阻止下一个命令在响应返回之前运行，并防止出现请求最初被延迟的情况。

***自动完成的例子：***

下面这个例子之所以如此强大，是因为Cypress将自动等待匹配`getSearch`别名的请求。您可以测试结果的实际原因，而不是强制Cypress测试成功请求的副作用(显示Book结果)。

```javascript
cy.server()
cy.route('/search*', [{ item: 'Book 1' }, { item: 'Book 2' }]).as('getSearch')

// 我们的自动完成字段是节流的，这意味着从最后一次按键500毫秒后只发送一个请求
cy.get('#autocomplete').type('Book')

// 等待请求+响应，从而将我们与节流请求隔离开来
cy.wait('@getSearch')

cy.get('#results')
  .should('contain', 'Book 1')
  .and('contain', 'Book 2')
```

## 失败

在上面的示例中，我们添加了一个断言来显示搜索结果。

***搜索结果与我们的应用程序中的一些内容相关联：***

1. 我们的应用程序向正确的URL发出请求。
2. 我们的应用程序正确地处理了响应。
3. 我们的应用程序将结果插入DOM。

在这个例子中，有许多可能的失败原因。在大多数测试工具中，如果我们的请求未能通过，我们通常只会在尝试在DOM中找到结果却没有匹配元素时才会收到错误。这是有问题的，因为不知道*为什么*结果没有显示。我们的渲染代码有问题吗？我们是否修改或更改了元素的属性，如`id`或`class`？也许我们的服务器给我们发送了不同的Book条目。

使用Cypress，通过添加一个{% url `cy.wait()` wait %}，您可以更容易地确定您的特定问题。如果响应没有返回，您将收到如下错误：

{% imgTag /img/guides/clear-source-of-failure.png "Wait Failure" %}

现在我们明确知道为什么我们的测试失败了。它与DOM无关。相反，我们可以看到要么我们的请求从未发出，要么请求发出到错误的URL。

## 断言

对请求使用{% url `cy.wait()` wait %}的另一个好处是，它允许您访问实际的`XHR`对象。当您想对这个对象进行断言时，这是非常有用的。

在上面的示例中，我们可以对请求对象进行断言，以验证它是否以URL中查询字符串的形式发送了数据。尽管我们在模拟响应，但仍然可以验证应用程序是否发送了正确的请求。

```javascript
cy.server()
cy.route('search/*', [{ item: 'Book 1' }, { item: 'Book 2' }]).as('getSearch')

cy.get('#autocomplete').type('Book')

// 这将生成XHR对象，其中包含请求、响应、url、方法等字段
cy.wait('@getSearch')
  .its('url').should('include', '/search?query=Book')

cy.get('#results')
  .should('contain', 'Book 1')
  .and('contain', 'Book 2')
```

***{% url `cy.wait()` wait %}生成的XHR对象具有进行断言所需的一切内容，包括：***

- URL
- 方法
- 状态码
- 请求体
- 请求头
- 响应体
- 响应头

# 另请参阅

- {% url "Network requests in Kitchen Sink example" https://github.com/cypress-io/cypress-example-kitchensink/blob/master/cypress/integration/examples/network_requests.spec.js %}
