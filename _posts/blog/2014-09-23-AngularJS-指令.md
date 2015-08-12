---
layout: post
title: AngularJS应用－指令
description: AngularJS使用
category: blog
---


## 一、基本概念


说明概念：附加在HTML元素上面自定义标记，可以是：标签、属性、class类，通过AngularJS的HTML编辑器（$compile），在指定元素上面附加指定的行为，可以操作DOM，改变DOM元素，甚至它子节点。

一般都是用ng开始，表示指令。
部分内置指令：ngBind,ngModle,ngView,ngClick等。
指令匹配形式：
	
	// 推荐使用方式
	<span ng-bing="name"></span>
	<span ng:bind="name"></span>
	<span ng_bind="name"></span>
	// 符合HTML验证，（data）
	<span data-ng-bind="name"></span>
	<span x-ng-bing="name"></span>
	
## 二、应用	

### 2.1 注册指令：
基本方法：
	
	module.directive("dir",function(){
		return {
			
		}
	})
一般返回一个匿名的对象

替换内容
	
	<div class="" ng-controller="de">
		<hello></hello>
	</div>
	
	//定义了一个hello指令
	var MyModule = angular.module("MyModule",[]);
	MyModule.directive("hello",function(){
		return {
			restrict : "E",
			template : "<div>替换了</div>",
			replace : true
		}
	})

restrict 可以是四个值：

- E：表示元素标签，<hello></hello>
- A：表示属性(默认属性), <div hello></hello>
- M：表示注释, <!-- directive: hello -->
- C：表示样式（class）,<div class="hello" ></div>

> 遇到一个问题，样式（C）时，使用在input输入框，不能用“－”并且在指令名称中不能用大写，如果标签中是大写，那么在注册的时候需要改成小写。
> 但是不能input框，可以注册用大写。没有找到规律


### 2.2 保留原有内容，并且增加内容：

	MyModule.directive("hello",function(){
		return {
			restrict : "E",
			transclude : true,
			template : "<div>添加内容<div ng-transclude></div></div>"
		}
	})

### 2.3 引用html模版
当有大量的html需要修改或者替换的时候可用，引用html模版文件，这样减少在js中写大量到html代码，并且还需要考虑拼接：

	MyModule.directive("hello",function(){
		return {
			restrict : "E",
			templateUrl : "hello.html",
			replace : true
		}
	})
	<-- hello.html -->
	<div>替换内容</div>
	
> 这里可以考虑一些常用的html模块，直接用这种方式，给一个指令直接就把需要的html结构给出来，但是这里还牵涉到一个css，和这个模块要用到的js。参考fis

缓存模版，当有一些模版会重复使用到到的时候可以使用缓存模版方式，主要是牵涉到几个方法，put、get

	//注射器加载完所有模块时，此方法执行一次，run。
	MyModule.run(function($templateCache){
	
		//put设置缓存模版，第一参数定义名称，第二个是内容。
		$templateCache.put("h","<div>缓存内容</div>");
		
	})
	
	//获取缓存的内容
	MyModule.directive("hello",function(){
		return {
			restrict : "E",
			replace : true,
			template : $templateCache.get("h")
		}
	})
	
### 2.4 LINk方法：
参数有四个分别是scope,element,attr,ctrl
	
	<!-- 指令 link使用 -->
	<!-- 定义controller -->
	<div class="" ng-controller="myLoad">
		<load class="btn-deep-blue mt-vbig">测试加载</load>
		<div id="show">显示</div>
	</div>
	
定义contraoller里面的方法：

	var myLoadModule = angular.module("MyModule",[]);
	myLoadModule.controller("myLoad",["$scope",function($scope){

		$scope.loadData = function(){
			angular.element(document.querySelector("#show")).addClass("btn-deep-blue");
			console.log("加载。。");
		}

	}]);
	
	//定义指令
		myLoadModule.directive("load",function(){
		return {
			restrict:"E",
			link : function(scope,element,attr){
					element.bind("click",function(){
	//这里的scope相当于和控制器中的$scope相连接了，能够获取到控制中的方法，通过$scope中转站衔接
						scope.loadData();
						// 作用相同:
						// scope.$apply("loadData()")
					})
					
			}
		}
	})

### 2.5 指令复用
通过link中的attr属性去复用指令

HTML:

	<div class="" ng-controller="myLoad1">
		<loading class="btn-deep-blue mt-vbig" loadfn="loadData()">测试加载</loading>
	</div>	
	<div class="" ng-controller="myLoad2">
		<loading class="btn-deep-blue mt-vbig" loadfn="loadData2()">测试加载2</loading>
	</div>

JS:

	// 控制器定义
	var MyModule = angular.module("MyModule",[]);
	MyModule.controller("myLoad1",["$scope",function($scope){
		$scope.loadData = function(){
			console.log("加载。。1");
		}
	}]);
	
	MyModule.controller("myLoad2",["$scope",function($scope){
		$scope.loadData2 = function(){
			console.log("加载。。2");
		}
	}]);
	
	// 指令定义
	myLoadModule.directive("load",function(){
		return {
			restrict:"E",
			link : function(scope,element,attr){
					element.bind("click",function(){
						scope.$apply(attr.loadfn)
			// 这里loadfn注意要小写
					})
					
			}
		}
	})

### 2.6 指令间的调用
这里说一下另外一个方法：controller

HTML:

	<ul>
		<superman strength>力量</superman>
		<superman strength speed>力量、速度</superman>
		<superman strength speed light>力量、速度、光</superman>
	</ul>
	
这里的controller是指令内部的，是暴露一些方法给外部调用:

	// 指令间调用
	MyModule.directive("superman",function(){
		return {
			//独立作用域
			scope:{},
			restrict: "AE",
			controller: function($scope){
				$scope.abilities=[];
				// 这里的this指的是？这里看是调用这个controller方法的对象，通过下面的调用知道，这是指令对象
				this.addStrength=function(){
					$scope.abilities.push("strength");
				}
				this.addSpeed=function(){
					$scope.abilities.push("speed");
				}
				this.addLight=function(){
					$scope.abilities.push("light");
				}
			},
			link : function(scope,element,attr){
				element.addClass("btn-deep-blue mt-tiny");
				element.bind("click",function(){
					console.log(scope.abilities);
				})
			}
		}
	});
	
	MyModule.directive("strength",function(){
		return {
			//require 引用了一个指令对象
			require: "^superman",
			// link最后一个参数是传入 require引用的对象
			link: function(scope,element,attr,s){
				s.addStrength();
			}
		}
	});
	MyModule.directive("speed",function(){
		return {
			require: "^superman",
			link: function(scope,element,attr,supermanCtrl){
				supermanCtrl.addSpeed();
			}
		}
	});
	MyModule.directive("light",function(){
		return {
			require: "^superman",
			link: function(scope,element,attr,supermanCtrl){
				supermanCtrl.addLight();
			}
		}
	});

> 什么时候用controller什么时候用link，link指指令内部，做一个绑定事件，或者一些事件的处理，而controller是给外部调用 	
	
	
	
			
			