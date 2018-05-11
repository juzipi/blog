#vue中的keep-alive

###一、介绍
能够缓存组件，避免重复加载。

页面生命周期钩子如上面的代码所示，这四个是最常用到的部分。这部分需要注意下，当引入keep-alive的时候，页面第一次进入，钩子的触发顺序created-> mounted-> activated，退出时触发deactivated。当再次进入（前进或者后退）时，只触发activated。

###二、使用
####实例1：返回充填
可以使用该方式，返回回去，前面一步填写内容还保存在内存中。当不需要保存的时候可以用：

	to="{name:XXX,from:'xxx'}；
	
通过activated钩子能够知道上一步from是什么，以此确定是否要清空还是保持数据。

####实例2：部分组件保持keep-alive

	<!-- 这里是需要keepalive的 -->
	<keep-alive>
   	 <router-view v-if="$route.meta.keepAlive"></router-view>
	</keep-alive>

	<!-- 这里不会被keepalive -->
	<router-view v-if="!$route.meta.keepAlive"></router-view>
	
设置路由信息:


	{
	  path: '',
	  name: '',
	  component: ,
	  meta: {keepAlive: true} // 这个是需要keepalive的
	},
	{
	  path: '',
	  name: '',
	  component: ,
	  meta: {keepAlive: false} // 这是不会被keepalive的
	}

###实例3：列表和详情
list.vue

	<ul>
    		<li v-for="(value, index) in getFilterData().data">
        	<router-link(:to="'/article/' + index")>
          <span v-text="value.title"></span>
        </router-link>
   	 </li>
	<ul>  

detail.vue

	<article v-html="details.content"></article>
	
问题是每次请求，因为用了keep-alive详情中内容不会变化。
解决方法是在beforeRouteEnter这个钩子中重新获取一边数据


