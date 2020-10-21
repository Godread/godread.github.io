---
title: 实现简单的axios异步增删改查功能
categories:
  - javascript
tags:
  - 异步请求
  - ajax
  - axios
toc: true
abbrlink: 6581d66
---

> 手动实现一个简单的 axios 增删改查的功能封装，实现非 rest API 接口的数据请求，数据的模拟用 json-server 来实现

<!-- more -->

## 预备数据

首先，全局安装 json-server

```
npm install -g json-server
```

在本地建立 db.json 文，并填充数据

```
{
  "posts": [
    {"id": 1, "title": "json-server", "author": "typicode"},
    {"id": 2, "title": "json-server2", "author": "typicode2"},
    {"title": "json-server+++", "author": "typicode+++", "id": 3}
  ],
  "comments": [
  	{"id": 1, "body": "some comment", "postId": 1}
  ],
  "profile": {
    "name": "typicode"
  }
}
```



## 实现原理

#### html代码

```html
<div>
    <button onclick="testGet()">xhr发送GET请求</button>
    <button onclick="testPost()">xhr发送请POST求</button>
    <button onclick="testPut()">xhr发送PUT请求</button>
    <button onclick="testDelete()">xhr发送DELETE请求</button>
</div>
```



#### js代码

```javascript
// 引入 xios
<script src="https://cdn.bootcdn.net/ajax/libs/axios/0.20.0/axios.min.js"></script>
<script>
    // 调用axios
    function testGet() {
        axios({
            url: 'http://localhost:3000/posts',
            method: "get",
            params: {
                'id': 2,
                "title": 'json-server2'
            }
        }).then(
            response => {
            	console.log(response)
            },
            error => {
            	alert(error.message)
        	}
        )
    }

    function testPost() {
        axios({
            url: 'http://localhost:3000/posts',
            method: "post",
            data: {
                "title": "json-server---",
                "author": "typicode---"
            }
        }).then(
            response => {
            	console.log(response)
            },
            error => {
            	alert(error.message)
            }
        )
    }

    function testPut() {
        axios({
            url: 'http://localhost:3000/posts/3',
            method: "put",
            data: {
                "title": "json-server+++",
                "author": "typicode+++"
            }
        }).then(
            response => {
            	console.log(response)
            },
            error => {
            	alert(error.message)
            }
        )
    }

    function testDelete() {
        axios({
            url: 'http://localhost:3000/posts/4',
            method: "delete",
        }).then(
            response => {
           		console.log(response)
            },
            error => {
            	alert(error.message)
            }
        )
    }
    
    /**
    * 封装axios ---> 原理
    */
    function axios({
        url,
        method = 'GET',
        params = {},
        data = {}
    }) {
    	// 返回一个promise对象
        return new Promise((resolve, reject) => {
        	// 处理method，变为大写
        	method = method.toUpperCase()

            // 处理query参数(拼接到url上) id=1&xxx=abc
            /*
            {
                id: 1,
                xxx: abc
            }
            */
        	let queryString = ''
            Object.keys(params).forEach((key) => {
            	queryString += `${key}=${params[key]}&`
            })

            if (queryString) {
                // 去除最后面的&
                queryString = queryString.substring(0, queryString.length - 1)
                // 拼接到url上
                url += '?' + queryString
            }

            // 1.执行异步ajax请求
            // 创建xhr对象
            const request = new XMLHttpRequest()
            
            // 打开连接(初始化请求，没有请求)
            request.open(method, url, true) // 开启异步
            
            // 发送请求
            // 这里有method必须要大写，因为上面己经处理method为大写，保持一致
            if (method === 'GET' || method === 'DELETE') {
            	request.send()
            } else if (method === 'POST' || method === 'PUT') {
                // 发送json数据时，要设置请求头，告诉服务器，请求体的格式是json
                request.setRequestHeader('Content-Type', 'application/json;charset=utf-8')
                request.send(JSON.stringify(data)) // 发送json格式请求体参数
            }

            // 2.绑定状态改变的监听
            request.onreadystatechange = function() {
                // 如果请求没有完成，直接结束
                if (request.readyState !== 4) {
                	return
                }

                // 如果状态响应码在200-300之间，代表成功，否则失败
                const {
                    status,
                    statusText
                } = request
                
                // 2.1.如果请求成功了，调用resolve()
                if (status >= 200 & status <= 299) {
                    // 准备结果数据对象response
                    const response = {
                        data: JSON.parse(request.response),
                        status,
                        statusText
                    }
                	resolve(response)
                } else {
                	// 2.2. 如果请求失败了，调用reject()
                	reject(new Error('request error status is ' + status))
              	}
            }
        })
    }
</script>
```

