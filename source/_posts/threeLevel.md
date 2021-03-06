---
title: 三级联动
date: 2020-01-09 23:33:33
tags:
- js
- ajax
- node.js
- html
categories:
- web前端
---

# 封装原生Ajax请求数据完成三级联动

使用node.js创建服务器,引用了express框架（express设置路由是真的方便），其中有用到art-template模板引擎（其实不用感觉代码量也不多，感觉像这种小玩意不需要使用到，当然我觉得现在练练，倒是肯定有能用到的时候）

<!-- more -->

##### 上代码

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
  <select name="province" id="province"></select>
  <!-- 城市跟县区都要初始化不然省份未被选择前option为空 -->
  <select name="city" id="city">
    <option value="">请选择城市</option>
  </select>
  <select name="area" id="area">
    <option value="">请选择县区</option>
  </select>


  <!-- 使用art-template模板引擎 -->
  <script src='../js/template-web.js'></script>
  <!-- 使用自己封装的ajax -->
  <script src='../js/ajax.js'></script>

  <!-- 省份的渲染模块 -->
  <script type='text/html' id='provinceTpl'>
    <option value="">请选择省份</option>
    {{each data}}
    <option value={{$value.id}}>{{$value.name}}</option>
    {{/each}}
  </script>
  <!-- 城市的渲染模块 -->
  <script type='text/html' id='cityTpl'>
    <option value="">请选择城市</option>
    {{each data}}
    <!-- 注意此处不是id 二十 citiesID 用于索引县区 -->
    <option value={{$value.citiesId}}>{{$value.name}}</option>
    {{/each}}
  </script>
  <!-- 县区的渲染模块 -->
  <script type='text/html' id='areaTpl'>
    <option value="">请选择县区</option>
    {{each data}}
    <option value={{$value.id}}>{{$value.name}}</option>
    {{/each}}
  </script>

  <script>
    // 获取下拉框的元素
    let province = document.getElementById('province');
    let city = document.getElementById('city');
    let area = document.getElementById('area');

    // 获取省份信息
    ajax({
      type:'get',
      url: 'http://localhost:3000/province',
      success(data) {
        // 响应数据后,渲染页面
        let html = template('provinceTpl', {data});
        province.innerHTML = html;
      }
    })

    // 当省份发生改变时,获取下一级的数据并渲染
    province.onchange = function() {
      // 当省份发生改变时,县区\城市都要被初始化
      let html = template('areaTpl', {})
      area.innerHTML = html;
      // 获取点击的元素的value
      let pid = this.value;
      ajax({
        url: 'http://localhost:3000/cities',
        data: {id: pid},
        success(data) {
          let html = template('cityTpl', {data});
          city.innerHTML = html;
        }
      })
    }

    city.onchange = function() {
      let cid = this.value;
      ajax({
        url: 'http://localhost:3000/area',
        data: {id: cid},
        success(data) {
          let html = template('areaTpl', {data})
          area.innerHTML = html;
        }
      })
    }

  </script>
</body>
</html>
```

##### 服务器

```js
// 1.引入express 模块 与 path 模块
const express = require('express');
const path = require('path');

// 2.创建服务器
const app = express();

// 3.设置静态资源路径
app.use(express.static(path.join(__dirname, 'public')));

// 4.监听端口
app.listen(3000, () => {
  console.log('localhost:3000')
})

// 配置路由
// 省级数据
app.get('/province', (req, res) => {
  provinceData = [
    {id: 1, name: '广东'},
    {id: 2, name: '北京'},
    {id: 3, name: '中山'}
  ]

  res.send(provinceData)
})

app.get('/cities', (req,res) => {
  let item = [];
  citiesData = [
    {id: 1, name: '广州1', citiesId: 1},
    {id: 1, name: '广州2', citiesId: 2},
    {id: 2, name: '广州3', citiesId: 3},
    {id: 2, name: '广州4', citiesId: 4},
    {id: 3, name: '广州5', citiesId: 5},
    {id: 3, name: '广州6', citiesId: 6},
    {id: 3, name: '广州7', citiesId: 7},
    {id: 3, name: '广州8', citiesId: 8},
    {id: 2, name: '广州9', citiesId: 9}
  ]
  // 判断传过来的值是否等于id的值,如果相等,就把数据加入数据,最后发送数据
  citiesData.forEach(element => {
    if(parseInt(req.query.id) === element.id) {
      item.push(element)
    }
  })
  res.send(item)
})

app.get('/area', (req,res) => {
  let item = [];
  areaData = [
    {id: 1, name: '天河区1'},
    {id: 1, name: '天河区2'},
    {id: 2, name: '天河区2'},
    {id: 2, name: '天河区3'},
    {id: 3, name: '天河区3'},
    {id: 3, name: '天河区5'},
    {id: 4, name: '天河区4'},
    {id: 4, name: '天河区6'},
    {id: 5, name: '天河区5'},
    {id: 5, name: '天河区6'},
    {id: 6, name: '天河区6'},
    {id: 6, name: '天河区2'},
    {id: 7, name: '天河区7'},
    {id: 7, name: '天河区1'},
    {id: 8, name: '天河区8'},
    {id: 8, name: '天河区75'},
    {id: 9, name: '天河区9'},
    {id: 9, name: '天河区10'},
    {id: 9, name: '天河区11'},
    {id: 9, name: '天河区12'},
    {id: 8, name: '天河区13'}
  ]
  areaData.forEach(element => {
    if(parseInt(req.query.id) === element.id) {
      item.push(element);
    }
  })
  res.send(item);
})
```



##### 最后是自己封装的ajax（因为我是直接在markdown里写的，有一些地方会有错误，到时碰到在修改）

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



多看！！多练！！！多做！！！！