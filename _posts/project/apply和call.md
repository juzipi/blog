---
layout: post
title: apply和call应用
description: 两个方法，详细使用场景
category: project
---

## 一、基本认识
都是Function原型定义的两个方法，Function.prototype.apply和Function.prototype.call。也就是可以通过函数进行调用。
		
		fn.apply(this,[arg1,arg2]);
		fn.call(this,arg1,arg2);
		
两个方法作用相同，第一个参数传入的都是，该函数内 this 的指向。

从第二个参数开始，可以看到传入的参数不同。前面apply传入的是一个数组，call传入的参数依次传入函数。
> call是可以在确定传入什么参数的时候使用，而apply是可以在一些不确定的参数情况下面使用，比如可以针对这个数组做一些添加和删除的动作。
> 
> 也就是apply传入的参数是可以变化的。

如果第一个参数传入的是null，函数体内this会指向默认的宿主对象，如果在浏览器中就是 window。

	var fn = function(){
		console.log(this);
	}
	fn.apply(null); //true

如果是严格模式下面，this还是为null。

	var fn = function(){
		"use strict";
		console.log(this===window);
	}
	fn.apply(null); //false

## 二、使用场景

1、改变this指向。
改变的是需要执行函数的内部指向。

	var FnApply = function(par){
		this.name = "apply";//这里其实是覆盖了ObjectApply对象的name属所以下面的这一行打印是apply
		console.log(this.name);//apply  
		console.log(this.sex); // man 这里调用的就是ObjectApply对象的色系属性
		console.log(par) // me  这里调用的是传入进Arr数组的第一个参数
		this.name = "apply2";//如果在这里重新覆盖，是不会对console.log(this.name);这个起作用。
	}
	var ObjectApply = {
		name : "ObjectApply",
		sex:"man"
	}
	var Arr = ["me","you"]
	FnApply.apply(ObjectApply,Arr);	
	
首先 FnApply.apply(ObjectApply,Arr);是执行了FnApply该函数，而把函数体内的 this指向的是ObjectApply这个对象。

修正this指向，在一些函数嵌套多一些的时候this指向经常，出现问题。
	
	document.getElementById("show").onclick=function(){
		console.log(this.id); //show
	}
如果在其中加入一个函数

	document.getElementById("show").onclick=function(){
		var fn = function(){
			console.log(this.id); //undefined
		}
		fn();
	}
这时候可以用apply或者是call来修正this指向。
	
	document.getElementById("show").onclick=function(){
		var fn = function(){
			console.log(this.id); //show
		}
		fn.call(this);
	}
	
> 当然在这里也可以前面把this缓存起来，放在一个变量里面。

	document.getElementById("show").onclick=function(){
		var _self = this;
		var fn = function(){
			console.log(_self.id); //show
		}
		fn();
	}
	
2、Function.prototype.bind
一般现在的浏览器都能很好的支持这个方法，该方法就是为函数内部指定this对象。如果没有实现该方法，模拟方法：

	Function.prototype.bind=function(context){
		var _self = this;
		return function(){
			return _self.apply(context,arguments);
		}
	}
	//分步骤解释一下该方法
	//原始方式
	document.getElementById("show").onclick=function(){
		var fn = function(){
			console.log(this.id); //show
		};
		fn.bind(this)();
	// fn.bind(),这里的fn是一个函数对象，还没有执行，只是执行了对象上面的bind方法。
	// 而bind方法返回的一个函数，也就是需要再一次执行。fn.bind()(),这个时候才是真正执行了bind之后返回的方法。
		fn(); //undefined,这里执行的是就是最开始的那段函数。
		
	}
	//直接把bind方法绑定在函数后面，这个时候赋给变量的fn就是执行bind完成之后返回的函数。
	//也就是：function(){
			return _self.apply(context,arguments);
		}

	document.getElementById("show").onclick=function(){
		var fn = function(){
			console.log(this.id); //show
		}.bind(this);
		fn();
		//这里执行的就是bind方法执行后返回的函数。
	}
	
	//也可以是这样：
	document.getElementById("show").onclick=function(){
		var fn = function(){
			console.log(this.id); //show
		};
		var fnfn = fn.bind(this);
		//把返回函数赋予一个变量
		fnfn();
	}
	
了解bind方法，需要了解的一点是，函数绑定该方法后，一定返回的是一个函数，需要再一次执行，实际上是执行了_self.apply(content,arguments);

高级bind方法实现：
	
	Function.prototype.bind = function(){
		var _self = this;
		var context = [].shift.call(arguments);
	//取出第一个参数作为上下文替换
		var arg = [].slice.call(arguments);
	//把其他参数转换成数组
		return function(){
			return _self.apply(context,[].concat.apply(arg,[].slice.call(arguments)));
	//concat合并后面执行函数的参数和前面arg数组
		}
	}
	
	document.getElementById("show").onclick=function(){
		var fn = function(){
			console.log(this.id); //show
			console.log(arguments); //you,wo 
		}.bind(this,"you");
		fn("wo");
	}

## 3、借用其他方法
1、类数组借用
一些类数组，但是不是真正的数组，不能使哟push方法，但是可以通过借用的方法，使得有数组的方法。例如arguments，它不是数组，但是通过借用可以实现数组的一些方法。
	
	[].shift.call(arguments); //取出最前面一项
	[].slice.call(arguments); //转换成数组
	[].push.call(arguments); //在后面添加一项
	
2、类似继承

借用别的方法放到，构建对象中。使对象也能借用到别的方法。
	
	var Add = function(a,b){
		return a + b;
	}
	var Del = function(a,b){
		return a - b;
	}
	var B =function(){
		this.add = Add.apply(this,arguments);
		this.del = Del.apply(this,arguments);
	}
	B.prototype.getAddNum = function(){
		return Add.apply(this,arguments);
		// 借用了Add方法，在对象中也有Add方法了。
	}

	var C  = new B(8,5);
	console.log(C.getAddNum(9,2)); // 11
	console.log(C.del);// 3

3、应用实例

3.1 单例模式实现

单例，简单来说就是，只是存在一个，不会出现两个，这个在弹出框的时候特别有用，这里牵涉到匿名函数，闭包，apply应用。	
	
	var singleFn = function(fn){
		var rel;
		return function(){
			return rel || (rel = fn.apply(this,arguments));
			//如果有rel,则返回rel，如果没有则执行fn函数。
		}
	}
	 
	var creatPopup = function(content){
		var popup = document.createElement("div");
		popup.style.display="none";
		popup.innerHTML = content;
		document.body.appendChild(popup);
		retuen popup;
	}
	
	var newCreatPopup = singleFn(creatPopup);
	
	document.getElementById("show").onclick=function(){
		var popup = newCreatPopup("创建弹出框");
		//这里应用就是单例函数中return返回回来的函数
		popup.style.display = "block";
	}

3.2 函数执行顺序

	Function.prototype.after = function(fn){
		var _self = this;
		//保存原有函数
		return function(){
			fn.apply(this,arguments);
			return _self.apply(this,arguments);
		// 返回原来函数执行结果
		}
	}
	Function.prototype.before= function(fn){
		var _self = this;
		return function(){
			var _s = _self.apply(this,arguments);
			fn.apply(this,arguments);
			return _s;
		// 返回原来函数执行结果
		}
	}
	var fn1 = function(){
		console.log(1);
	}
	var fn3 = function(){
		console.log(3);
	}
	var fn2 = function(){
		console.log(2);
	}

	var fnA = fn1.after(fn3).before(fn2);

	fnA(); // 3,1,2
	//这里先执行fn3,再到before方法中，这里首先执行fn1方法，再执行fn2

## 四、总结

使用apply或call，改变原有函数的上下文执行情况，要看当前函数执行的环境，在高级的函数使用中会经常用到apply概念。