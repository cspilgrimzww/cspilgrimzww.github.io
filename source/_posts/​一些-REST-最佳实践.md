title: '​一些 REST 最佳实践 '
date: 2015-6-30 00:49:18
tags: RESTFUL
---
[原文地址](http://toutiao.io/r/gwd8p)

如今，REST APIs 已经非常普遍，几乎所有WEB应用都用到了它们。提供简单，一致，实用的API是种义务，方便其它人很容易的使用。即使下面的这些规范，在你看来很正常，我经常看到人们不遵守它。这是我写这篇文章的原因。

当你设计 RESTful API 时，下面是一些应该牢记的最佳规范。

免责声明：下面的这些规范是根据过去的经验，我认为最好的。如果你有别的想法，咱们邮件讨论。

**1. 给API加上版本**
API版本应该是必备的。这样API不会随时间过时。一种方法是把版本放到URL里（/api/v1...)

另一个巧妙的花招是使用 Accept HTTP header，来传递需要的版本，正如github所做的。（https://developer.github.com/v3/media/#request-specific-version）

（备注：Github 的格式是，application/vnd.github[.version].param[+json]，version指定版本，param是想要的格式，txt，html等。从评论看似乎Github的方式更完美）

通过版本，你可以改变API的结构，而不用担心老版客户端的兼容问题。（备注：API提供者默默承受维护多套API的痛苦）

**2. 使用名词，而不是动词**
我经常看到有人使用动词而不是名词来表示资源名称，例如下面这些：
* /getProducts
* /listOrders
* /retreiveClientByOrder?orderId=1

从结构整洁和一致角度考虑，你应该总是使用名词。而且，巧妙使用 HTTP 方法（GET，POST）可以把想要的操作从资源名称上去除。如下面的例子：
* GET  /products      返回所有产品列表
* POST /products      添加产品到产品列表
* GET  /products/4    提取Id为4的产品
* PATCH/PUT /products/4 更新Id为4的产品

**3. 使用复数形式**
在我看来，同一资源命名，混合使用单数和复数形式不是好主意。很快就会混淆，带来不一致。

即使对 show/delete/update 操作，使用 /artists 而不是 /artist 也更好点。

**4. GET 和 HEAD 操作应该是安全的（无副作用）**
RFC2616 明确规定 HEAD 和 GET 必须是安全的（不能改变资源状态）

右边是一个不好的例子： GET /deleteProduct?id=1

如果搜索引擎检索了那个页面，画面太美我不敢看（备注：实践中对删除操作都有权限验证，就算操作引擎抓取也没啥破坏）

（备注：POST和PUT的区别，POST是不幂等的，PUT是幂等的。如果多次调用URL得到的结果都一样，那就是幂等的。例如，发评论，如果评论ID在提交评论前已经生成，那么无论点多少次提交，看到的都是一条评论。如果评论ID在提交评论后生成，点多少次提交就看到多少条评论。）

**5. 使用嵌套资源**
如果想获取全部的子集，使用嵌套路由来让风格简洁。例如想从所有唱片中选取特定的，使用 GET /artists/8/albums （备注：这里8就是所谓嵌套路由，指导选取哪个唱片）

**6. 分页**
通过 HTTP 返回超大结果集不是好主意。序列化大的JSON数据很慢，这会导致性能问题。

通常的做法是分页，Facebook，Twitter，Github都是这么做的。提取少里数据更快，就算需要多次调用，也比一次提取很大（但执行很慢）的数据更高效。

如果想分页，一个好的方法是通过LINK HTTP header，来提示前一页和后一页。正如 Github 做的那样。

（备注：LINK的用法 Link: <http://next_url>; rel="next", <http://last_url>; rel="last", <http://first_url>; rel="first", <http://prev_url>; rel="prev"）

**7. 使用合适的 HTTP 状态码**
请求返回时，无论请求成功与否，总是使用正确的返回码。下面是一些可能用到的状态码。

7.1 成功状态码（2XX系列）
* 201 Created 当成功创建资源时（INSERT）*
* 202 Accepted 当请求被接受，并放到后台执行时（异步任务）*
* 204 No Content 当请求成功，但是没有内容返回时（例如 DELETE 时）*

（备注：是否 200 就足够了，引入其它是否增加理解负担）

7.2 客户端错误（4xx系列）
* 400 Bad Request 当处理querystring或http body时出错（例如非法JSON）*
* 401 Unauthorized 认证失败*
* 403 Forbidden 认证成功时，操作或请求的资源不被允许*
* 406 Not Acceptable 请求格式不被接受（例如试图请求JSON数据，但服务器只提供XML）*
* 410 Gone 请求的资源被永久删除（备注：咋判断是不存在还是被永久删除了）*
* 422 Unprocessable entity 当创建对象时发生可用性错误*

**8. 总是返回一致的错误内容**
当发生错误时，总是返回一致的错误描述。错误结构总是相同，这样更容易解析错误信息。

如下描述，清晰，简单，自说明
HTTP/1.1 401 Unauthorized
```
{
    "status": "Unauthorized",
    "message": "No access token provided.",
    "request_id": "594600f4-7eec-47ca-8012-02e7b89859ce"
}
```