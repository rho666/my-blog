---
title: Ajax
date: 2020-01-11 00:24:43
tags:
- ajax
- node.js
categories:
- web前端
---

## 同步与异步

同步意味着  客户端提出了一个请求后，在回应之前只能等待

异步意味着 客户端提出了一个请求后，还可以继续提交其他请求

<!-- more -->


> 举个例子： 服务员把客人点的菜单交给厨师，厨师把菜做完后，服务员在端给客人，期间服务员一直在厨房等待厨师把菜做完——这就是同步；服务员把客人点的菜单交给厨师后，继续干自己的事情，比如将其他客人的菜单也提交给厨师，等厨师做好菜后，再端给下单的客人 —— 这就是异步

​		

## GET请求方式

```javascript
xhr.open('get', 'http:/baidu.com?name=zhangsan&age=12')
```

## POST 请求方式

```js
// post请求必须要设置
xhr.setRequestHeader('content-Type','application/x-www-form-urlencoded') 
xhr.send('name=zhangsan&age=20')
```

### 解决一个低版本ie浏览器的缓存问题

> 我们只需要时在请求地址上加入一个参数，是这个参数的值每次都不一样就行了
>
> ```
> xhr.open('get','http://localhost:3000/cache?t='+Math.random())
> ```
>
> Math.random() --->随机获取[0,1)之间的小数



## 封装一个Ajax

```js
function ajax(options) {
	// 如果没传入数据的话就使用默认数据
	const defaults = {
		type: 'get', // 默认为get
		url: '',
		data: {},
		header: {
			'Content-Type': 'application/x-www-form-urlencoded',
		},
		success(data) {},
		error(err,xhr) {}
	}
	
	// Object.assign方法，options有值则替换defaults
	Object.assign(defaults, options);
	
	// 创建XMLHttpRequest()对象
	const xhr = new XMLHttpRequest();
	let params = '';
	
	// 如果时get请求，遍历data， 把数据拼装成 name=zhang&age=14 这种形式
	if(defaults.type === 'get') {
		for(let attr in defaults.data) {
			params += attr + '=' + defaults.data[attr] + '&';
		}
		// 把最后的那个 & 剪切掉，重新赋值给params
		params = params.substring(0, params.length - 1);
		// 拼装get请求的url
		defaults.url = defaults.url + '?' + params;
	}
	
	// 提交方式
	xhr.open(defaults.type, defaults.url);
	
	// 如果时post请求的话还需要加 xhr.setRequestHeader()
	if(defaults.type == 'post') {
		let contentType = defaults.header['Content-Type'];
		xhr.setRequestHeader('Content-Type', contentType);
		// 如果时json数据的话，需要将数据转换
		if(contentType === 'application/json') {
			params = JSON.stringify(defaults.data)
		}
		// post与get 的请求体不同， get在url上，post在send()上
		xhr.send(params);
	}else {
		// 发送请求
		xhr.send(); 
	}
	
	// 返回数据在这里处理
	xhr.onload = function() {
		let responseText = xhr.responseText
		let contentType = xhr.getResponseHeader('Content-Type');
		// 如果时json数据，就将其转换
		if(contentType.includes('application/json')) {
			responseText = JSON.parse(responseText);
		}
		//判断是否请求成功
		if(xhr.status === 200) {
			// 请求回调success函数
			defaults.success(responseText)
		}else {
			// 错误回调error函数
			defaults.error(responseText,xhr)
		}
	}
}
```

## FormData()对象

**`FormData`** 接口提供了一种表示表单数据的键值对的构造方式，经过它的数据可以使用 [`XMLHttpRequest.send()`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/send) 方法送出，本接口和此方法都相当简单直接。如果送出时的编码类型被设为 `"multipart/form-data"`，它会使用和表单一样的格式。

### 方法

- [`FormData.append()`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/append)

  向 `FormData` 中添加新的属性值，`FormData` 对应的属性值存在也不会覆盖原值，而是新增一个值，如果属性不存在则新增一项属性值。

- `FormData.delete()`

  从 FormData 对象里面删除一个键值对。

- [`FormData.entries()`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/entries)

  返回一个包含所有键值对的[`iterator`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)对象。

- [`FormData.get('key')`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/get)

  `返回在 FormData` 对象中与给定键关联的第一个值。

- [`FormData.getAll()`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/getAll)

  返回一个包含 `FormData` 对象中与给定键关联的所有值的数组。

- [`FormData.has()`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/has)

  `返回一个布尔值表明 FormData` 对象是否包含某些键。

- [`FormData.keys()`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/keys)

  返回一个包含所有键的[`iterator`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)对象。

- [`FormData.set('key', 'value')`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/set)

  给 `FormData` 设置属性值，如果`FormData` 对应的属性值存在则覆盖原值，否则新增一项属性值。

- [`FormData.values()`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/values)

  返回一个包含所有值的[`iterator`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)对象。



如下代码

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <form action="" id='form'>
    <input type="text" name='username'><br>
    <input type="password" name='pwd'><br>
    <input type="button" value='提交' id='btn'>
  </form>
  <p></p>


  <script>
    let form = document.getElementById('form');
    const btn = document.getElementById('btn');
    const content = document.getElementsByTagName('p')[0];
    console.log(content)
	// 监听按钮的点击，上传表单
    btn.addEventListener('click', () => {
      //创建一个FormData对象，并且传入表单
      let formdata = new FormData(form);
      
      /*
      	get('key') 获取表单对象属性的值
      	set('key', 'value') 设置表单对象属性的值
      */
      
	  // 获取表单对象属性的值
      console.log(formdata.get('username'));
      
	  // 如果设置的表单属性存在，将会覆盖属性原有的值
      formdata.set('username', 'username');
      // 如果设置的表单属性不存在，将会创建这个表单属性
      formdata.set('age', 18);
    
      const xhr = new XMLHttpRequest();

      xhr.open('post', 'http://localhost:3000/formdata')

      xhr.send(formdata);

      xhr.onload = function() {
        if(xhr.status === 200) {
          let cnt = JSON.parse(xhr.responseText);
          console.log(cnt)
          if(cnt.username.length === 0) {
            content.innerText = '无用户名';
            return
          }
          if(cnt.pwd.length === 0) {
            content.innerText = '请输入密码';
            return 
          }
          content.innerText= cnt.username + '用户您好!, 是否确认密码'+cnt.pwd
        }
      }
    })

    

</script>
</body>
</html>
```



从写一遍服务器逻辑

```js
// 引入express框架，用于创建服务器等
const express = require('express');
// 引入path模块
const path - require('path');
// 引入第三方formidable模块，用于解析表单数据，很好用！！
const formidable = require('formidable')
// 创建服务器
const app = express();
// 设置请求静态资源路径,在根目录下手动创建public文件夹
app.use(express.static(path.join(__dirname, 'public')));
// 监听3000端口
app.listen(3000, () => {
	console.log('请访问127.0.0.1:3000')
})

// 配置路由
app.post('/formdata', (req, res) => {
	// 创建formidable对象
	let form = formidable.IncommingForm();
	// 解析表单,须传入req
	// err时错误数据， fields是普通文本数据，files是文件数据 如图片等
	form.parse(req, (err, fields, files) => {
		// 这里只需要文本数据,直接发送回去
		res.send(fields)
	})
})
```

> 直接在markdown手打，代码没测试，可能有错误，但调式一下就ok！！