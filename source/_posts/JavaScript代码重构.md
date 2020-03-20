---
title: JavaScript代码重构
tags: JavaScript
---
摘自【JavaScript设计模式与开发实践】一书，稍作精简修改，如需详细了解，请阅读原书。
文中出现作者一词，特指原书作者。
<!--more-->

设计模式的目的是为许多重构行为提供目标。实际开发中，除了用设计模式重构外，本文列出了一些容易忽略的细节。

本文提出的都是建议，没有哪些严格遵守的标准，具体是否重构，以及如何进行重构，需要根据系统的类型、项目工期、人力等外界因素具体考量。

# 提炼函数
一个好的函数有着良好的命名，函数体内包含逻辑清晰明了。如果一个函数过长，需要加上若干注释才显得易读些，就有必要进行重构了。
如果函数中有一段代码可以独立出来，最好把这段代码提取成一个独立的函数，有以下几点好处：
- 避免出现超大函数
- 独立出来的函数利于代码复用
- 独立出来的函数容易被覆写
- 独立出来的函数如果有良好命名，本身就起到注释作用

# 合并重复的条件片段
函数体内有些条件分支语句，内部散布了一些重复代码，有必要进行合并去重。
```js
var paging = function(currPage) {
	if (currPage <= 0) {
		currPage = 0;
		jump(currPage); // 跳转 
	} else if (currPage >= totalPage) {
		currPage = totalPage;
		jump(currPage); // 跳转
	} else {
		jump(currPage); // 跳转
	}
};
```
jump( currPage )在每个条件分支内都出现，进行去重重构
```js
var paging = function(currPage) {
	if (currPage <= 0) {
		currPage = 0;
	} else if (currPage >= totalPage) {
		currPage = totalPage;
	}
	jump(currPage); // 把 jump 函数独立出来
};
```

# 把条件分支语句提炼成函数
在程序设计中，复杂的条件分支语句是导致程序难以阅读和理解的重要原因，也容易导致一个庞大的函数。
```js
var getPrice = function(price) {
	var date = new Date();
	if (date.getMonth() >= 6 && date.getMonth() <= 9) { // 夏天
		return price * 0.8;
	}
	return price;
};
```
把判断是否处于夏天部分提炼成一个单独函数，能更准确表达代码意思，函数名也起到注释作用
```js
var isSummer = function() {
	var date = new Date();
	return date.getMonth() >= 6 && date.getMonth() <= 9;
};
var getPrice = function(price) {
	if (isSummer()) { // 夏天
		return price * 0.8;
	}
	return price;
};
```
# 合理使用循环
合理利用循环完成一些重复性工作，在代码量更少情况下，完成相同功能。

# 提前让函数退出代替嵌套条件分支
嵌套的 if、else语句相比平铺的 if、else，在阅读和理解上更加困难，难于维护
```js
var del = function(obj) {
	var ret;
	if (!obj.isReadOnly) { // 不为只读的才能被删除
		if (obj.isFolder) { // 如果是文件夹
			ret = deleteFolder(obj);
		} else if (obj.isFile) { // 如果是文件
			ret = deleteFile(obj);
		}
	}
	return ret;
};
```
在面对一个嵌套的 if 分支时，我们可以把外层 if 表达式进行反转return，避免嵌套
```js
var del = function(obj) {
	if (obj.isReadOnly) { // 反转 if 表达式
		return;
	}
	if (obj.isFolder) {
		return deleteFolder(obj);
	}
	if (obj.isFile) {
		return deleteFile(obj);
	}
};
```

# 传递对象参数代替过长参数列表
一个函数接收参数数量越多，越难理解和维护，使用该函数时首先得搞明白参数的含义，以免少传或传错位置。在中间新增参数，会涉及许多代码修改
```js
var setUserInfo = function(id, name, address, sex, mobile, qq) {
	console.log('id= ' + id);
	console.log('name= ' + name);
	console.log('address= ' + address);
	console.log('sex= ' + sex);
	console.log('mobile= ' + mobile);
	console.log('qq= ' + qq);
};
setUserInfo(1314, 'sven', 'shenzhen', 'male', '137********', 377876679);
```
把参数放入对象内，不必再关心参数数量和顺序，，保证参数对应key不变即可
```js
var setUserInfo = function(obj) {
	console.log('id= ' + obj.id);
	console.log('name= ' + obj.name);
	console.log('address= ' + obj.address);
	console.log('sex= ' + obj.sex);
	console.log('mobile= ' + obj.mobile);
	console.log('qq= ' + obj.qq);
};
setUserInfo({
	id      : 1314,
	name    : 'sven',
	address : 'shenzhen',
	sex     : 'male',
	mobile  : '137********',
	qq      : 377876679
});
```

# 尽量减少参数数量
我们需要尽量减少函数接收的参数数量

# 少用三目运算符
条件分支逻辑逻辑复杂时，用if、else语句会好很多，易于阅读易于维护。

# 合理使用链式调用
如果链条结构相对稳定，后期不易发生修改，可以使用链式调用。如果链条容易发生变化，还是建议使用普通调用。
```js
var User = function() {
	this.id = null;
	this.name = null;
};
User.prototype.setId = function(id) {
	this.id = id;
	return this;
};
User.prototype.setName = function(name) {
	this.name = name;
	return this;
};
console.log(new User().setId(1314).setName('sven'));

// 或者
var User = {
	id      : null,
	name    : null,
	setId   : function(id) {
		this.id = id;
		return this;
	},
	setName : function(name) {
		this.name = name;
		return this;
	}
};
console.log(User.setId(1314).setName('sven'));
```

# 分解大型类
在作者编写HTML5版“街头霸王”最初版代码中，负责创建游戏人物的Spirit类非常庞大，包括了人物的攻击、防御等动作
```js
var Spirit = function(name) {
	this.name = name;
};
Spirit.prototype.attack = function(type) { // 攻击
	if (type === 'waveBoxing') {
		console.log(this.name + ': 使用波动拳');
	} else if (type === 'whirlKick') {
		console.log(this.name + ': 使用旋风腿');
	}
};
var spirit = new Spirit('RYU');
spirit.attack('waveBoxing'); // 输出：RYU: 使用波动拳
spirit.attack('whirlKick'); // 输出：RYU: 使用旋风腿
```
面向对象设计鼓励将行为分布在合理数量的更小对象之中
```js
var Attack = function(spirit) {
	this.spirit = spirit;
};
Attack.prototype.start = function(type) {
	return this.list[type].call(this);
};
Attack.prototype.list = {
	waveBoxing : function() {
		console.log(this.spirit.name + ': 使用波动拳');
	},
	whirlKick  : function() {
		console.log(this.spirit.name + ': 使用旋风腿');
	}
};
```
Spirit 类变得精简了很多，不再包括各种各样的攻击方法，而是把攻击动作委托给Attack 类的对象来执行，这段代码也是策略模式的运用之一：
```js
var Spirit = function(name) {
	this.name = name;
	this.attackObj = new Attack(this);
};
Spirit.prototype.attack = function(type) { // 攻击
	this.attackObj.start(type);
};
var spirit = new Spirit('RYU');
spirit.attack('waveBoxing'); // 输出：RYU: 使用波动拳
spirit.attack('whirlKick'); // 输出：RYU: 使用旋风腿
```

# 用return退出多重循环
假设函数体内有一个两重循环语句，要在内层循环中判断，当达到某个临界条件时退出外层循环。通常会引入一个控制标记变量
```js
var func = function() {
	var flag = false;
	for (var i = 0; i < 10; i++) {
		for (var j = 0; j < 10; j++) {
			if (i * j > 30) {
				flag = true;
				break;
			}
		}
		if (flag === true) {
			break;
		}
	}
};
```
或设置支循环标记
```js
var func = function() {
	outerloop:
		for (var i = 0; i < 10; i++) {
			innerloop:
				for (var j = 0; j < 10; j++) {
					if (i * j > 30) {
						break outerloop;
					}
				}
		}
};
```
两种做法都难以阅读，更简单方法是在需要终止循环时直接return退出整个方法：
```js
var func = function() {
	for (var i = 0; i < 10; i++) {
		for (var j = 0; j < 10; j++) {
			if (i * j > 30) {
				return;
			}
		}
	}
    console.log( i ); // 这句代码没有机会被执行
};
```
但有个缺点return后，无法执行后面代码，解决方法是把需要执行代码放到return后，如果代码较多，可以提炼成一个单独函数
```js
var print = function(i) {
	console.log(i);
};
var func = function() {
	for (var i = 0; i < 10; i++) {
		for (var j = 0; j < 10; j++) {
			if (i * j > 30) {
				return print(i);
			}
		}
	}
};
```