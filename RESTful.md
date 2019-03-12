# RESTful Api
> REpresentational State Transfer
## 0.为什么写这篇内容
使用百度搜索restful能所搜到很多，几乎包含了大部分RESTful的介绍，有几篇写的也很好，但是在这么多内容中很难找到一篇最全面的，希望这篇文章能让您“**一站式的了解和学会RESTful**”。
*内容会掺杂部分个人见解，可能会存在一些问题，欢迎各位来怼*

## 1.定义，什么是RESTful（What）
REpresentational State Transfer（具象表述传输），我的理解是，**就算没有文档，通过URL我也知道这个接口是干什么的**
RESTful 是一种规范，是一种设计风格。类似于“**咱们规定以后代码中，所有的变量名都使用首字母小写的驼峰，类名都使用首字母大写的驼峰，所有数据库字段都用小写下划线**”。RESTful不是架构，不是任何别的东西。
RESTful != HTTP/HTTPS，RESTful风格设计通常应用在**包含但不限于**HTTP协议上，比如在物联网协议中的CoAP协议也可以遵循RESTful，MQTT协议中的topic也可以遵循RESTful。但是目前应用最广的还是HTTP，因此以下内容均已HTTP协议展开。

## 2.为什么是它（Why）
古代Web开发，前后端揉在一起，使用“模板”进行服务端数据绑定。
在移动互联网的大环境中，服务端开发需要适应多端接入，因此服务端提供的API需要与客户端完全解耦。
那么基于资源的RESTful风格，无疑便成为了较好的解决方案。

## 3.如何设计（How）
### 3.0 简介

### 3.1 HTTP Method 选择
先用两个链接描述一下：
[HTTP 1.1标准请求方法](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)
[RESTful推荐使用方法](https://restfulapi.net/http-methods/)
以下仅介绍RESTful推荐的几种方法

| 方法 | 适用 | 安全 | 幂等 |
| - | - | - | - |
| GET | 查询资源，不允许对资源状态修改 | 安全 | 幂等 |
| POST | 创建资源 | 非安全 | 非幂等 |
| PUT | 更新资源 | 非安全 | 幂等 |
| PATCH | 更新资源 | 非安全 | 幂等 |
| DELETE | 更新资源 | 非安全 | 幂等 |

PUT和PATCH是概念最容易混淆的两个方法。
我在项目中也很少用到PATCH方法。
查阅很多资料后，发现大多数认为PUT用于整体更新，PATCH用于单字段更新。
比如：

	用户资源有mobile和email两个字段
	PUT /users/1
	{"mobile":123,"email":"a@a.com"}
	
	PATCH /users/1
	{"mobile":123}
	或
	PATCH /users/1/mobile
	123

而RESTful的推荐的PATCH实现如下：

	PATCH /users/1
	[{"op":"replace","path":"email","value":"new.email@example.org"},{"op":"remove","path":"mobile"}]
	
本人目前对该推荐实现持保留意见，在业务上这种接口形式的需求很少见（有不用意见，可以在issue中讨论）。

### 3.2 URI设计
先展示几个非RESTful的URI:

	GET /v1/getUserList
	GET /users/get-user-list
	POST /createUser
	POST /updateUser

设计原则：
用最直观的语言概述设计原则：**操作一个类型的集合或集合中的一个实例**
比如：

	我需要所有停车场的资料
	GET /parks
	
	给我那个停车场的所有资料
	GET /parks/1
	
	去那个停车场，去找一辆车牌号是“沪A12345”的车
	GET /parks/1/vehicles?plate_num=沪A12345
	
	我需要一颗炸弹
	POST /bombs
	{"name":"小男孩","type":1,"equivalent":20}
	
	把这颗炸弹绑到那台车上去
	POST /vehicles/1/bombs
	{"id":2}
	
	设置刚才那颗炸弹的起爆时间
	PUT /bombs/2 或者 /vehicles/1/bombs/2
	{"exploded_at":"2019-03-08T18:59:59"}
	
	给赎金了，拆了炸弹吧
	DELETE /vehicles/1/bombs/2
	这里删除的是绑定关系
	
	把炸弹扔了
	DELETE /bombs/2
	
以上例子相对较有趣，希望能加深理解。
URI中的单词，需要使用名词，集合操作需要用复数。
比如：

	某组织的所有管理员
	GET /organizations/1/administrators
	
	某辆车的违法记录
	GET /vehicles/1/violations
	
	某台车的发动机参数（假设一台车只有一台发动机）
	GET /vehicles/1/engine

### 3.3 请求参数设计
在GET请求中经常会用到请求参数，比如分页，筛选，排序等。

#### 3.3.1 分页：
就分页而言设计参数可以是：page和page_size，offset和limit
这两中没有孰优孰劣，整套系统统一就好。我倾向于后者，因为一个单词搞定并且在业务代码中不用再计算offset。

	GET /organizations/1/vehicles?offset=20&limit=10
	GET /organizations/1/vehicles?page=3&page_size=10
	
#### 3.3.2 排序：
通常情况下：

	GET /organizations/1/vehicles?sort_by=price&order_by=desc
这种方式如果需要对多字段排序存在一定的问题或者说不那么优雅：

	sort_by[]=price&order_by[]=desc&sort_by[]=length&order_by[]=asc
	或
	sort_by=price&order_by=desc&sort_by=length&order_by=asc
而以下两种相对优雅（好看）：

	GET /organizations/1/vehicles?sort=price,desc;length,asc
	GET /organizations/1/vehicles?sort=-price,+length
	
#### 3.3.3 筛选：
筛选通常比较简单，比如：

	plate_num=沪A12345
但如果出现比较的时候，大致会变成这样：

	price_max=150000&price_min=100000&length_min=4000
	10到15万区间内，四米车长的车辆
再看看我认为相对较好的：

	price=between:100000,150000&length=gte:4000
我们分别使用
**eq、neq、gt、lt、gte、lte、between、in、nin**
表示
**相等、不等、大于、小于、大于等于、小于等于、区间、在集合中、不在集合中**

#### 3.3.4 返回字段：
	
	fields=plate_num,vin,created_at
	
#### 3.3.5 关系：
在很多情况下，当我们请求一个示例或者集合时，要求服务端返回某些特定的关系，如最常见的树形结构。
比如：
id为1的组织为根组织，我们需要查询它的子组织，可以使用以下请求
	
	GET /organizations/1/children
但是，当我们只知道根组织，但是需要子组织的子组织（孙子节点）时怎么办？这是需要告知接口，需要返回关系。

	GET /organizations/1/children?with=children
我们也可以要求返回多个关系

	GET /vehicles/1?with=driver,license
	GET /vehicles?with=driver,license
	
### 3.4 使用Header（HTTP Header）
header是HTTP协议中很重要的一部分，好的header设计会让你的api看上去不那么的冗余，我们看一下下面这个api

	/v2/organizations?response=xml&language=en_us&charset=gbk&token=123456789
首先我们肯定的是，这个api没有任何问题。在代码实现上response,language,charset等字段完全可以通过middleware等手段做全局处理，但这并不是最佳实践。
在协议上其实已经为你准备好了你要的东西，何必自己重复造轮子。

	response和version 可以使用 Accept
	language 可以使用 Accept-Language
	charset 可以使用 Accept-Charset
	token 可以使用 Authorization
很多人会把版本号放在url上，但我跟喜欢把版本号放在header中，还是那句话，**保持URI整洁**。（可参考 [RESTful推荐版本控制](https://restfulapi.net/versioning/)）
其实header还可以处理很多，比如缓存控制：**If-Match和ETag**，**Last-Modified和If-Modified-Since**
比如在以下场景中：
> 微信个人相册页面，每次打开需要请求所有用户图片。为了节省流量，客户端可以缓存图片在本地。那么用户如果更新图片，如何让客户端重新请求呢？

实现概要：图片名以hash方式存储时，可以在列表页请求时就返回不同的文件名，客户端可通过查询本地缓存中是否存在同名图片实现。优势是省去了单个图片的网络请求，但同时存在一个极小概率问题，如果用户更新的图片恰好hash冲突时，无法得到更新。
这时可以在返回中加上**Last-Modified**内容可以是上传时间，在下次请求是加上**If-Modified-Since**，如果服务器判断没有更新，直接返回304，并不返回内容即可。

其实，缓存控制还有很多需要考虑的地方。比如：**Expires: Tue, 12 Mar 2019 07:28:00 GMT**，告知客户端只缓存到2019-03-12 07:28:00。具体应用还需要根据业务场景判断及使用。

### 3.5 使用Http状态码（HTTP Status Code）
状态码在整个体系中，应该是最重要的。在工作中我曾和一位工程师沟通返回状态码的过程中，听到了一下的一句话：
背景是这样的：在一个应该返回400的地方，他返回了200
> “200 表示你已经请求到我的controller了，由于你的参数传错了，我在返回body的code中返回给你了400”

其实他说的没有什么问题，早期在做web service的时候，所有请求一律返回200，错误内容放在body中，由业务代码负责处理错误。这种方式没什么不好的，只要大家约定好：
> 我们玩CS、魔兽的时候不要使用鼠标，只能用触摸板
> 在讨论问题的时候只能写字，不要用嘴说

很多时候我们都需要做一些约定，而合理使用返回状态码，能让我们在业务层之前就接获异常并能根据不同的情况而处理。试想一下在微服务的架构中，所有的请求异常如果都放在业务代码中是什么样的情况。

状态码：

| 类型 | 描述 |
| - | - |
| 1xx 信息 | 传输协议层信息 |
| 2xx 成功 | 请求已被接受并处理 |
| 3xx 重定向 | 客户端需要做一些额外的处理来完成请求 |
| 4xx 客户端异常| 由于客户端的原因，请求失败 |
| 5xx 服务端异常| 由于服务端的原因，请求失败 |

具体列表可以参考：[维基百科](https://zh.wikipedia.org/wiki/HTTP状态码)