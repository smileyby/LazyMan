# 如何实现一个LazyMan

##### 出处：前端面试题
##### 题目：实现一个LazyMan，可以按照以下方式调用：

```js

	LazyMan("Hank"); 
	//输出 Hi!This is Hank!

	LazyMan("Hank").sleep(10).eat("dinner");
	//输出 Hi!This is Hank!
	//等待10秒.. Wake up after 10 Eat dinner~
	
	LazyMan("Hank").eat("dinner").eat("supper")
	//输出 Hi! This is Hank! Eat dinner~ Eat dinner~

	LazyMan("Hank").sleepFirst(5).eat("supper")
	// 输出
 	// 等待5s Wake up after 5 
	// Hi This is Hank!
	// Eat supper
	// 以此类推

```
> 这是典型的JavaScript流程控制，问题的关键是如何实现任务的顺序执行。在Experss有一个类似的东西叫中间件，这个中间件和我们这里的吃饭，睡觉等任务很类似，每一个中间件执行完成后调用next()函数，这个函数调用下一个中间件。
> 对于这个问题，我们可以利用类似的思路来解决，首先创建一个任务队列，然后利用next()函数来控制任务的顺序执行：

```js
	
	function _LazyMan(name) {
		this.tasks = [];
		var self = this;
		var fn = (function(n){
			var name = n;
 			return function(){
				console.log("Hi!This is "+name+"");
				self.next();
			}
		})(name);
		this.tasks.push(fn);
		setTimeout(function(){
			self.next(); // 在下一个循环启动任务
		},0);
	}

	// 事件调度函数
	_LazyMan.prototype.next = function(){
		var fn = this.tasks.shift();
		fn && fn();
	}
	_LazyMan.prototype.eat = function(name){
		var self = this;
		var fn = (function(name){
			console.log("Eat "+name+"~");
			self.next();
		})(name);
		this.tasks.push(fn);
		return this; // 实现链式调用
	}
	_LazyMan.prototype.sleep = function(time){
		var slef = this;
		var fn = (function(time){
			return function(){
				setTimeout(function(){
					console.log("Wake up after"+time+"s!");
				},time*1000);
			}
		})(time);
		this.tasks.push(fn);
		return this; 
	}
	_LazyMan.prototype.sleepFirst = function(time){
		var slef = this;
		var fn = (function(time){
			return function(){
				setTimeout(function(){
					console.log("Wake up after"+time+"s!");
				},time*1000);
			}
		})(time);
		this.tasks.unshift(fn);
		return this;
	}
	// 封装
	function LazyMan(name){
		return new _LazyMan(name);
	}

```