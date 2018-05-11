# RequireJS初步认识
>关键词：冲突、依赖、加载

## 1、JS模块加载

### 模块：实现特定功能的一组方法。

#### 1.1 原始方法

污染了全局变量，可能会有命名冲突，不能明显看出模块之间联系。

	function f1(){
		//...
	}
	
#### 1.2 对象写法
建立一个对象，可以包含一些方法，


	var _module = {
		_count : 0,
		f1 : function (){
			//...
		},
		f2 : function (){
			//...
		}
	}

调用的时候：
		
	_module._count；

1、这样写可以降低一些命名冲突的可能性，但是也就是降低，不能保证就不会冲突，为了杜绝这种情况发生，可能就会加长命名空间：
		
	_module._mod_._Utils._count；
	
这确实解决命名冲突，但是一个调用起来麻烦，二要记忆住那个方法的调用也是比较费事。

2、这种方式也会暴露所有模块成员，内部状态被外部所改变

	_module._count＝5；		


#### 1.3 立即执行函数
>优点：不暴露私有属性

>缺点：外部也获取不到，不能成为公用组件。


	(function(){
		var _count=0;
		
		function f1(){
			//...
		}
	})();
	
## 2、JS文件加载
#### 2.1 原始文件加载
	
	<!DOCTYPE html>
	<html>
    	<head>
        	<script type="text/javascript" src="a.js"></script>
    	</head>
    	<body>
      		<span>body</span>
    	</body>
    </html>
a.js    

	function fun1(){
		alert("it works");
	}
	
	fun1();
当alert执行的时候页面一片空白，里面的`<span>body</span>` 没有执行。

这个时候js实际是阻塞了页面的渲染，这还只是一个简单的js，当一个实际页面载入的时候，可能会有十个以上的js加载，这样首屏渲染的速度就会大大降低。

#### 2.2 RequireJS加载


        <script type="text/javascript" src="require.js"></script>
        <script type="text/javascript">
            require(["a"]);
        </script>
a.js

	define(function(){
    	function fun1(){
     		 alert("it works");
    	}

    	fun1();
    })
这一次加载和上面不一样是，弹出alert的时候，页面已经被渲染，当然还有别的方式也可以达到这个目的，就是把a.js放在页面底部执行。

## 3、RequireJS使用
#### 3.1基本API

* define 定义一个模块

* require 加载依赖模块，并执行加载完后的回调函数

		require(["js/a"]);
		
处理加载完成后的回调函数
	
	require(["js/a"],function(){
    	alert("loaded");
	})
	
#### 3.2 paths使用

	require.config({
    	paths : {
        	"a" : "js/a"  
    	}
    })
    
	require(["a"],function(){
    	alert("loaded");
    })
require.config是require的配置项设置，这里paths主要就是为了配置加载模块的路径，起一个更短的名字好记。
	
	paths : {
        	"a" : ["js/a","js/b"]
    	}
  这里配置了两个路径表示当地一个路径没有加载成功的时候，会用第二路径模块加载。
  
####   加载库
  	
  	require.config({
  		baseUrl: "js/lib",
  		paths: {
  				"jquery": "jquery.min",
  				"underscore": "underscore.min",
  				"backbone": "backbone.min"
  				 }
  		});

#### 3.3 AMD模块写法
requireJS加载的是模块，采用的AMD规范，那么模块也必须要按照AMD规范来写。
	
	//math.js
	define(function(){
	
		var addFn = function(x,y){
			return x+y;
		};
		
		//模块返回的对象
		return {
			add : addFn
		}
		
	});
	
加载方法：

	//main.js
	require(['math'],function(m){
		console.log(m.add(2+2));  // 4
	});
如果定义的模块还依赖其他模块，define()函数的第一个参数，必须是数组，指明依赖模块名称
	
	//定义的模块依赖lib.js
	define(['lib'],function(mylib){
		
		function delFn(){
			mylib.doSomething();
		}
		
		return {
			del : delFn
		};
		
	});
	
	
#### 3.4 加载非规范模块

	

#### 3.5 全局配置

一般不会把config的配置单独写在一个文件中，作为配置文件进行加载

	<script type="text/javascript" src="require.js" data-main="main"></script>

data－main会在require.js加载完成之后再去加载main.js，这个js里面把页面中的一些配置项目放进去。

## 4、文件压缩合并

因为jquireJS这种工作方式，导致每一个模块一个文件，这样会有N多个模块被分成文件，对于前端性能里面来说会发起更多的请求，导致渲染速度减慢。这就需要一个合并文件的功能。

现在有两种方式:

* 一种是用grunt里面的`grunt-contrib-requirejs`
* 一种是通过r.js，进行优化

这里主要说一下r.js的配置使用，它使用方式也有两种：

一种是直接写着命令行里面：

	node r.js -o baseUrl=. paths.jquery=some/other/jquery name=main out=main-built.js
	
一种是建立一个build.js（推荐）：
	
	node r.js -o build.js

这里的-o表示自动优化

	




	
>参考文章：

##### requirejs中文网站

[http://www.requirejs.cn/docs/api.html](http://www.requirejs.cn/docs/api.html)

##### Javascript模块化编程（三）：require.js的用法

[http://www.ruanyifeng.com/blog/2012/11/require_js.html
](http://www.ruanyifeng.com/blog/2012/11/require_js.html)