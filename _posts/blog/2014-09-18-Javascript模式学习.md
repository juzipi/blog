#Javascript模式学习
---
##变量

### 1、全局变量
都知道不能随便取全局变量，在浏览器中存在一个默认的一个全局对象this，还有一个全局的属性window，
	
	var n = 2；
	this.n; //2
	this.window.n; //2

如果想要改变全局对象名称也可以随便在任何函数里面都可以
	 
	 var global=(function(){
	 	return this;
	 }())
	 //重新命名全局对象的名称未global;

### 2、函数内变量会前置

	var n="blue";
	function varFront(){
		consloe.log(n); //undefined
		var n = "red"; 
		console.log(n); //red
	}

这里我认为的第一个会是`blue`，但是却是`undefined`，因为`javascript`的引擎机制的原因，会先创建变量、函数、参数，这个阶段是一个预编译的阶段，会扫描整个上下文，再来代码运行阶段，运行里面的函数表达式。在这一段代码中因为后面创建的变量和前面的全局变量是同名的，根据刚才的原则先创建变量再运算。

实际就变成：

	var n="blue";
	function varFront(){
		var n; //提前到前面
		console.log(n);
		var n = "red";
		console.log(n); //rend
		
	}


## 循环

### 1、for

循环也是常用的一种语法方式，在开发中避免对于DOM操作时候多次寻找耗费资源

比如有一个ul下面的li集合
	
	for（var i = 0; i<li.length;i++ ）{
		//li[i];
	}
这里每一次的读取i都回去重新找一遍li，而操作DOM是非常耗资源的一个事情，应该改成：
	
	var length=li.length;
	for (var i=0; i<length; i++){
		//li[i]
	}
保存为一个变量存起来，就读取一次减少资源消耗。
再简化一下：
	
	var i,li=[];
	for(i=li.length;i--){
		//li[i]
	}
这里减少一个变量，并且是减量循环，减量循环比增量循环速度更快，解释是：

`因为和零比较要比和非零数字或数组长度比较要高效的多`?这个没有很明白；
再有就是while循环

	var li=[],i=li.length;
	while(i--){
		//li[li];
	}
### 2、for-in
这个用来循环对象里面的属性，这里需要了解对象属性有两种情况，一种是原生一种prototype原型立案链集成。通过方法hasOwnProperty()去过滤。
实例：

	var man={hands:2,head:1,legs:2},i;
	if(Object.prototype.clone=="undefined"){
		Object.prototype.clone=function(){};
	}
	for(i in man){
		if(man.hasOwnProperty(i))}  
		//通过hasOwnProperyt判断clone是不是原有的属性，还是通过prototype加进来的。这里如果运行到clone，通过的值是false；
			console.log(i, ":", man[i]);
		}
	}

### 不扩充内置圆形
可以扩充构造函数的prototype的原型，可以给object()，添加新的方法，但是这种方式尽量不要使用因为：
<ul>
<li>不好控制，和你一起开发的人员不知道你加了这么一个原型在里面，会不会重新覆盖，</li>
<li>一个有可能新的EC版本增加新的方法和现在的方法冲突，</li>
<li>就像上面那个object遍历如果不加hasOwnProperty()判断会遍历出来，这个和预想的结果不一样，而加了这个需要和你合作的每个开发人员说明。</li>
</ul>

也有情况是可以加上的，就是未来EC版本会加上这个方法，你可以预先把这个实现了，并且和团队其他人员沟通过后，给出文档。

### switch