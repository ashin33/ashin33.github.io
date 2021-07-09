---
layout: Restful HTTP API
title: Restful HTTP API 设计规范
date: 2021-07-09 15:57:56
tags: [2021]
categories: [代码]
top_img: /img/restful.jpeg
cover: /img/restful.jpeg
---
# What's RESTful
RESTful 是一种软件设计风格，由 [Roy Fielding](https://roy.gbiv.com/) 在他的 论文 中提出，全称为 Representational State Transfer，直译为表现层状态转移

# 使用RESTful的优势
* 安全可靠，高效，易扩展。
* 简单明了，可读性强，没有歧义。
* API 风格统一，调用规则，传入参数和返回数据有统一的标准

# 设计规范

## 域名
1. 域名应尽量使用https协议,可使用[certbot](https://certbot.eff.org/)制作免费https证书,后面研究一下
2. api应与主域名区分开,使用专用域名(.eg: `https://api.alon.wang`)或者放在主域名下(eg:`https://alon.wang/api`)

## 版本迭代
随业务的发展,api大概率会发生迭代,为了保证新老用户的使用,应控制好版本.
实现方法
* 版本号加入URL中

```
https://alon.wang/api/v1
https://alon.wang/api/v2
```
* 使用HTTP请求头的Accept来进行区分

```
  https://alon.wang/api/
      Accept: application/prs.alon.v1+json
      Accept: application/prs.alon.v2+json
```
后端具体实现可以参考[zedisdog的实现思路](https://learnku.com/laravel/t/10579/do-api-multi-version-compatibility-through-the-version-field-in-header)
>推荐使用第二种,前端可以不需要修改url


## url编写
RESTful的设计中,所有东西都看作资源,而每一个资源都应该是一个名词,且尽量应为复数(也可以为单数,比如github中项目的issue锁,一个问题只有一个锁,所以此时锁为单数),URL设计中,应避免动词出现

```github的栗子
GET /issues                                      列出所有的 issue
GET /orgs/:org/issues                            列出某个项目的 issue
GET /repos/:owner/:repo/issues/:number           获取某个项目的某个 issue
POST /repos/:owner/:repo/issues                  为某个项目创建 issue
PATCH /repos/:owner/:repo/issues/:number         修改某个 issue
PUT /repos/:owner/:repo/issues/:number/lock      锁住某个 issue
DELETE /repos/:owner/:repo/issues/:number/lock   解锁某个 issue
```


> GET:获取资源
> POST:新建资源
> PUT:更新整个资源的属性,应提供资源的所有属性过来更新
> PATCH:更新资源的部分属性,不需要提供资源的所有属性
> DELETE:删除某个属性

## 响应码
根据不同的响应,我们应该使用不同的状态码,使响应的数据更加规范与可读

| 响应码 | 注释                                                                                              |
|:------|:-------------------------------------------------------------------------------------------------|
| 200   | OK - 对成功的 GET、PUT、PATCH 或 DELETE 操作进行响应。也可以被用在不创建新资源的 POST 操作上               |
| 201   | Created - 对创建新资源的 POST 操作进行响应。应该带着指向新资源地址的 Location 头                          |
| 202   | Accepted - 服务器接受了请求，但是还未处理，响应中应该包含相应的指示信息，告诉客户端该去哪里查询关于本次请求的信息 |
| 204   | No Content - 对不会返回响应体的成功请求进行响应（比如 DELETE 请求）                                      |
| 304   | Not Modified - HTTP 缓存 header 生效的时候用                                                        |
| 400   | Bad Request - 请求异常，比如请求中的 body 无法解析                                                    |
| 401   | Unauthorized - 没有进行认证或者认证非法                                                              |
| 403   | Forbidden - 服务器已经理解请求，但是拒绝执行它                                                         |
| 404   | Not Found - 请求一个不存在的资源                                                                    |
| 405   | Method Not Allowed - 所请求的 HTTP 方法不允许当前认证用户访问                                          |
| 410   | Gone - 表示当前请求的资源不再可用。当调用老版本 API 的时候很有用                                          |
| 415   | Unsupported Media Type - 如果请求中的内容类型是错误的                                                 |
| 422   | Unprocessable Entity - 用来表示校验错误                                                             |
| 429   | Too Many Requests - 由于请求频次达到上限而被拒绝访问                                                  |
