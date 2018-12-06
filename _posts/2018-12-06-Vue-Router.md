## 前言

Vue Router 是 Vue.js 官方的路由管理器。它和 Vue.js 的核心深度集成，让构建单页面应用变得易如反掌。包含的功能有：

- 嵌套的路由/视图表
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 Vue.js 过渡系统的视图过渡效果
- 细粒度的导航控制
- 带有自动激活的 CSS class 的链接
- HTML5 历史模式或 hash 模式，在 IE9 中自动降级
- 自定义的滚动条行为

用 Vue.js + Vue Router 创建单页应用，是非常简单的。使用 Vue.js ，我们已经可以通过组合组件来组成应用程序，当你要把 Vue Router 添加进来，我们需要做的是，将组件 (components) 映射到路由 (routes)，然后告诉 Vue Router 在哪里渲染它们。

### 起步

前言划重点**将组件映射到路由，然后告诉VueRouter在哪里渲染**；

另一个重点：**router实例的routes属性是对路由的配置，这个数组存放了整个路由的配置**
	
导航标签：可以写在Vue单文件组件里；`router-link标签存放导航链接，router-view会将传入的路由组件进行渲染`；

	<!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
	<!-- 路由出口 -->
	<!-- 路由匹配到的组件将渲染在这里 -->
	<router-view></router-view>

导航逻辑：这些逻辑需要写在`<script></script>`里，组件文件里应该也可以，但是使用组件文件需要手动导入；例子里会看到；

	// 0. 如果使用模块化机制编程，导入Vue和VueRouter，要调用 Vue.use(VueRouter)
	// import Vue from 'vue'
	// import Router from 'vue-router'
	// 导入HelloWorld组件
	// import HelloWorld from '@/components/HelloWorld'
	
	// Vue.use(Router)

	// 1. 定义 (路由) 组件。
	// 可以从其他文件 import 进来
	// 组件可以是外部的组件，导入进来就行
	const Foo = { template: '<div>foo</div>' }
	const Bar = { template: '<div>bar</div>' }
	
	// 2. 定义路由
	// 每个路由应该映射一个组件。 其中"component" 可以是
	// 通过 Vue.extend() 创建的组件构造器，
	// 或者，只是一个组件配置对象。
	// 我们晚点再讨论嵌套路由。
	const routes = [
	  { path: '/foo', component: Foo },
	  { path: '/bar', component: Bar }
	]
	
	// 3. 创建 router 实例，然后传 `routes` 配置
	// 你还可以传别的配置参数, 不过先这么简单着吧。
	// 注意这里是router的实例，
	// 如果前面是import Router from 'vue-router'，这里要写成new Router
	const router = new VueRouter({
	  routes // (缩写) 相当于 routes: routes
	  /* 另一种写法，path和component两个属性是必须的；
	  routes: [{
		path: '/foo',
		component: Foo
		}]
	  */
	})
	
	// 4. 创建和挂载根实例。
	// 记得要通过 router 配置参数注入路由，
	// 从而让整个应用都有路由功能
	// 注意这里要挂载到根实例上，这里的app是template模板的根实例。
	const app = new Vue({
	  router
	}).$mount('#app')
	
	// 现在，应用已经启动了！

### 动态路由匹配

动态路由指的是将某种模式匹配到所有路由，并映射到同一个组件；常见场景有一个`User`组件，对于不同用户都要用这个组件来渲染；我们在`Vue-Router`路由路径中使用动态路径参数；

例子：实例对象中

	const User = {
	  template: '<div>User</div>'
	}
	
	const router = new VueRouter({
	  routes: [
	    // 动态路径参数 以冒号开头
		// /user/youzi, /user/ayou等类似的路径都会映射到相同的路由。
	    { path: '/user/:id', component: User }
	  ]
	})

一个路径参数用`:`标记，匹配到路由是，参数值会被放在`this.$route.params`，这个参数是通用的可以在每个组件中使用；可以试试在组件`User`中使用这个参数；

	const User = {
	  template: '<div>User {{ $route.params.id }}</div>'
	}

一个路由中可以设置多段路径参数；对应的值都会添加到`$route.params`对象中，

	参数：/user/:username/post/:post_id	
	路径：/user/evan/post/123	 
	$route.params == { username: 'evan', post_id: '123' }

使用路由参数时，如果从同级路由切换，如从`/user/foo`导航到`/user/bar`，为了效率，原来的组件会被复用，这意味着组件的生命周期钩子不会再被调用；两种方法可以监控路由参数的变化；

	const User = {
	  template: '...',
	  watch: {
	    '$route' (to, from) {
	      // 对路由变化作出响应...
	    }
	  }
	}

或者使用导航守卫；

	const User = {
	  template: '...',
	  beforeRouteUpdate (to, from, next) {
	    // react to route changes...
	    // don't forget to call next()
	  }
	}

### 嵌套路由

两点：



1.在组件里添加标签`<router-view />`，用以渲染内层路由；

2.在`router`实例对象中的`routes`数组中某个路由路径下添加`children`数组，内含内嵌路由对象；

	const User = {
	  template: `
	    <div class="user">
	      <h2>User {{ $route.params.id }}</h2>
	      <router-view></router-view>
	    </div>
	  `
	}

	const router = new VueRouter({
	  routes: [
	    { path: '/user/:id', component: User,
	      children: [
			// 当 /user/:id 匹配成功，
        	// UserHome 会被渲染在 User 的 <router-view> 中
        	{ path: '', component: UserHome },
	        {
	          // 当 /user/:id/profile 匹配成功，
	          // UserProfile 会被渲染在 User 的 <router-view> 中
	          path: 'profile',
	          component: UserProfile
	        },
	        {
	          // 当 /user/:id/posts 匹配成功
	          // UserPosts 会被渲染在 User 的 <router-view> 中
	          path: 'posts',
	          component: UserPosts
	        }
	      ]
	    }
	  ]
	})

### 编程式的导航

除了使用标签创建导航链接，还可以借助router实例的方法；

`router.push(location)`在Vue实例内部通过`$router`访问路由实例，因此可以调用`this.$router.push`，这个方法会想H5的history栈添加一个新纪录，所以当用户点击浏览器后退按钮时，会回到之前的URL；

当点击标签`<router-link>`时，这个方法会在内部调用，所以`<router-link :to="..."`等同于调用`router.push(...)`；

该方法的参数可以是字符串路径，或者是一个描述地址的对象：

    // 字符串
    router.push('home')
    // 对象
    router.push({ path: 'home' })
    // 命名的路由
    router.push({ name: 'user', params: { userId: 123 }})
    // 带查询参数，变成 /register?plan=private
    router.push({ path: 'register', query: { plan: 'private' }})

如果提供了`path`，`params`会被忽略，

	const userId = 123
	router.push({ name: 'user', params: { userId }}) // -> /user/123
	router.push({ path: `/user/${userId}` }) // -> /user/123
	// 这里的 params 不生效
	router.push({ path: '/user', params: { userId }}) // -> /user

`router.replace(location)`和`push`方法很像，不同的是`replace`方法不会向history添加新的记录，而是替换当前的history记录。这个方法和标签的写法`<router-link :to="..." replace>`是一样的；

`router.go(n)`方法参数是个整数，意思是在history记录向前或向后退加多少部，类似`window.history.go(n)`

	// 在浏览器记录中前进一步，等同于 history.forward()
	router.go(1)
	// 后退一步记录，等同于 history.back()
	router.go(-1)
	// 前进 3 步记录
	router.go(3)
	// 如果 history 记录不够用，那就默默地失败呗
	router.go(-100)
	router.go(100)

### 命名路由

说白了命名路由就是`router`实例的`routes`配置中包含了`name`属性；

	// 这是一个路由的配置信息，其中name是user，我们可以在其他地方用到这个属性
	const router = new VueRouter({
	  routes: [
	    {
	      path: '/user/:userId',
	      name: 'user',
	      component: User
	    }
	  ]
	})
	// 这里在标签里面把to属性绑定到一个对象上，注意前面有冒号的':'；
	<router-link :to="{ name: 'user', params: { userId: 123 }}">User</router-link>
	// 这种方法和在实例中调用router.push()是一样的；
	router.push({ name: 'user', params: { userId: 123 }})

两种方式都能把路由导航到`/user/123`这个路径下；

### 命名视图

用于同时（同级）展示多个视图，而不是嵌套展示，如一个布局里有`sidebar`和`maincontent`两个视图，可以在界面中有多个单独命名的视图，而不是只有一个单独的出口，如果`router-view`没有设置名字，那么是`default`，一个视图使用一个组件渲染，因此对于同个路由，多个视图就要多个组件，配置的属性是`components`，带上s；

	<router-view class="view one"></router-view>
	<router-view class="view two" name="a"></router-view>
	<router-view class="view three" name="b"></router-view>

	const router = new VueRouter({
	  routes: [
	    {
	      path: '/',
	      components: {
	        default: Foo,
	        a: Bar,
	        b: Baz
	      }
	    }
	  ]
	})

嵌套命名视图的使用

