---
layout: post  
title: "博客Restful API设计"  
subtitle: "服务计算"  
author: "ChenJF"  
header-img: "img/post-bg-imgs/2-systems-analysis-design.jpg"  
header-mask: 0.4  
catalog: true
tags:  服务计算
---

* TOC
{:toc}


﻿* 用户注册

  1. Request

     url: /v1/users/register(POST)
     
     

    ```
     {
     	"username": ""
     	"password": "",
     	"captcha_id": "",
     	"captcha_code": "",
     	"createtime": YYYY-MM-DDTHH:MM:SSZ
     }
    ```

  2. Response
```
{
  "id": 1
  "user_name": ""
}
status code: 201 
```

* 用户登陆
1. Request
 url: /v1/users/login(POST)



```
{
	"username": ""
	"password": "",
	"captcha_id": "",
	"captcha_code": "",
	"created_at": YYYY-MM-DDTHH:MM:SSZ
}
```

1. Response
  成功：

  
```
{
	"login_token": "",
	"expires_time": 0
}
status code: 201 OK
```
错误：



```
{
  "message": "Invalid password",
  "documentation_url": ""
}
status code: 401 Unauthorized
```
过多尝试禁止：



```
{
  "message": "attempts too much!",
  "documentation_url": ""
}
status code： 403 Forbidden
```

* 获取博客分类列表
1. Request
url: /v1/blog/categories(GET)

2. Response

   
```
{
 "categories"[
{
	"cate_id": "0", 
    "cate_name": "", 
    "art_count": 0
}],
...
}
status code: 200 OK
```
* 创建分类列表（需合法token）
1. Request
  /v1/blog/categories(POST)

  
```
{
    "name": ""
}
```
2. Response

   
```
{
    "id": 0,
    "cate_name": ""
}
status code: 201 
```

* 更新分类列表（需合法token）
1. Request
url: /v1/blog/categories/{category_id}(PUT)



```
 {
     "name": ""
 }
```

1. Response

   
```
{
    "id": 0,
    "name": ""
    "short_cut": "",
    "content": ""
}
status code: 200 OK
```

* 删除分类（需合法token）
1. Request
url: /v1/blog/categories/{category_id}(DELETE)
```
{
    "name": ""
}
```
2. Response

```
{
  "status": "",
}
status code: 204 No Content
```
* 获取某个分类内的文章
1. Request
url: /v1/blog/categories/{category_id}(GET)

2. Response
```
{
    "category": "",
    "articles": [
        {
            "id": 0,
            "title": "",
            "author": "",
            "created_at": YYYY-MM-DDTHH:MM:SSZ,
            "short_content": ""
        },
    ]
    ...
}
status code: 200 OK
```
* 获取某个文章
1. Request
url: /v1/blog/articles/{article_id}(GET)

2. Response
```
{
    "id": 0,
    "category_id": 0,
    "category_name": "",
    "author": "",
    "title": "",
    "content": ""
}
status code: 200 OK
```
* 创建新文章（需合法token）
1. Request
url: /v1/blog/articles(PUT)
```
{
    "category_id": 0,
    "article_id": 2,
    "title": "",
    "content": ""
}
```
2. Response
```
{
    "categoryId": 0,
    "article_id": 0,
    "title": "",
    "content": ""
    "created_at": YYYY-MM-DDTHH:MM:SSZ
}
status code: 201 
```

* 更新文章（需合法token）
1. Request
url: /v1/blog/articles/{article_id}(PUT)
```
{
    "category_id": 0,
    "article_id": 0,
    "title": "",
    "content": ""
}
```
2. Response
```
{
    "categoryId": 0,
    "article_id": 0,
    "created_at":YYYY-MM-DDTHH:MM:SSZ
    "updated_at":YYYY-MM-DDTHH:MM:SSZ
    "url": ""
}
status code: 200 OK
```

* 删除文章（需合法token）
1. Request
url: /v1/blog/articles/{article_id}(DELETE)

2. Response
```
{   
    "message": ""
}
status code: 204 No Content
```
* 获取文章的评论
1. Request
url: /v1/blog/articles/{article_id}/comments(GET)

2. Response
Status: 200 OK
```
    {
        "id": "0",
        "user": {
            "id": 0,
            "username": "",
        },
        "content": "",
        "created_at": YYYY-MM-DDTHH:MM:SSZ
    },
    Status: 200 OK
```

* 添加评论（需合法token）
1. Request
url: /v1/blog/articles/{article_id}/comments(POST)
```
{
    "content": ""
}
```

2. Response
```
{
    "id": 0,
    "article_title": ""
    "content": "",
    "createdAt": YYYY-MM-DDTHH:MM:SSZ
}
status code: 200 OK
```
注：若有请求**不满足格式要求**会返回以下响应：
```
{
    message: ""
}
status code: 422
```
未登录：
```
{
  "message": "",
}
status code: 401 Unauthorized
```