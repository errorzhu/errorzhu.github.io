# web api 设计

## 原则
面向资源设计

## 命名约定
简单
直观
一致

因为大多数开发者都不是以英语作为母语的，这些命名规范的一个目的是确保大多数开发者可以很容易地理解一个API。规范通过鼓励在命名方法和资源时使用简单一致的小词汇量来达到这一目的。
APIs里面的名称应该是正确的美式英语。例如，license（而不是licence），color（而不是colour）。
为了简洁起见，可以使用常用的短语或长词的缩写。 例如，API就比Application Programming Interface更好一点。
如果有可能，使用直观而且人们熟悉的术语。举个例子，当要描述移除（并且删除）一个资源的时候，delete比erase更恰当。
为相同的概念使用相同的名称或术语，包括一些跨API共享的概念。
避免名字重载。 对不同的概念使用不同的名称。
避免在API上下文以及更大的Google APIs生态系统中使用过于笼统的含糊不清的名称。他们会导致API的一些概念被误解。相反，我们要选那些可以准确描述API概念的名称。这在定义最重要的API元素（如资源）的时候尤为重要。我们没有明确指出哪些是不能用来定义的名称，因为每一个名称都需要在上下文中被评估后确定。举个例子，instance，info和service这些名称在过去就被发现很有问题，因为选中的名字应该能够清晰地描述API的概念（例如：什么东西的instance）以及能够将它与一些相关的概念区分开来（例如“alert”指的是规则，还是型号，还是通知？）。
在使用那些可能与普通编程语言中的关键字冲突的名称时要小心。这些名字是可以被使用的，不过可能会在API review时接受额外的审查，所以应该谨慎并尽可能少地使用它们。

## 根据 HTTP 方法定义 API 操作
GET 检索位于指定 URI 处的资源的表示形式。 响应消息的正文包含所请求资源的详细信息。
POST 在指定的 URI 处创建新资源。 请求消息的正文将提供新资源的详细信息。 请注意，POST 还用于触发不实际创建资源的操作。
PUT 在指定的 URI 处创建或替换资源。 请求消息的正文指定要创建或更新的资源。
PATCH 对资源执行部分更新。 请求正文包含要应用到资源的一组更改。
DELETE 删除位于指定 URI 处的资源

## 自定义方法
对于自定义方法，它们应该使用如下通用HTTP映射
```
https://service.name/v1/some/resource/name:customVerb
```

## 数据筛选和分页
API 可以允许在 URI 的查询字符串中传递筛选器

```
/orders?limit=25&offset=50
```

## 对 RESTful Web API 进行版本控制


## 参考资料

https://learn.microsoft.com/zh-cn/azure/architecture/best-practices/api-design
https://google-cloud.gitbook.io/api-design-guide/documentation