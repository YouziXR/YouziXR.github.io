### 前言

----------
本文记录在对人人开源项目（前端部分）——`renren-fast-vue`进行二次开发过程中遇到的一些问题。

人人项目GitHub地址[https://github.com/renrenio/renren-fast-vue](https://github.com/renrenio/renren-fast-vue "人人")


----------
### 目录解读

事实上这部分内容在项目的wiki中有；这里多写写我自己的东西。

- build

构建项目的相关内容，都是webpack打包的配置信息，这里注意一点，如果不想用ESlint工具进行语法检测，可以在`webpack.base.conf.js`的方法`createLintingRule`把这些语法检测语句都注释掉；

----------
另外在根目录的`.eslintignore.js`里也能添加不需要进行检测的文件和文件夹

- config

前端配置项，改前端端口host之类的可以在这里改。

- src

源码，这个目录分了很多子目录，一般常用的有静态资源assets，全局路由router，全局仓库store，公用方法utils，视图views（业务代码都在这里），main.js（挂载一些常用的全局方法如axios请求等），另外main.js也引入了外部的依赖插件，像是vue-cookie等

- static

第三方资源，不进行打包；config里放全局变量，包括请求后台的API地址，域名版本号等；plugins放的是静态第三方库，原生项目里包括了echarts等

### 二次开发 

#### 前期准备

0. 二次开发的设计的目录一般都在src里，没有特殊标注的话根目录都是这个。
1. 前端写新组件，最好先使用静态路由，这样方便调试，不用改数据库，可以在组件完善后改成动态路由，这样可以由管理员进行权限配置。写好组件后，在`/router/index.js`文件下，`mainRoutes.children`添加静态路由，和`home`的形式一样，写一个`routes`对象即可，必需包含属性`path, component, name, meta`，关于路由元信息meta属性，参照[https://router.vuejs.org/zh/guide/advanced/meta.html#路由元信息](https://router.vuejs.org/zh/guide/advanced/meta.html#路由元信息 "路由元信息")；项目建议路由使用name属性进行跳转；路由写完后，需要在文件`views/main-sidebar.vue`进行侧边栏的渲染，不然是看不到无法点击这个路由的。
2. 组件一般就写在`/views`目录下，可以新建立个目录专门来放某个业务的代码，如上传文件之类的；

#### 功能模块开发

##### 查询工资条模块的开发：

- 这里用到了element-ui中的table表格，标签属性中`data`表示了这个表格绑定的数据对象，因为和Vue兼容，所以可以直接写成`v-bind:data="dataSource"`，把要渲染的数据绑定在dataSource上；关于el-table的使用[http://element-cn.eleme.io/#/zh-CN/component/table#table-biao-ge](http://element-cn.eleme.io/#/zh-CN/component/table#table-biao-ge)
- 由于前期我在开发时，后端数据还未生成，我也没有用mock测试接口，所以导致我在渲染表格时把表格的列写成静态的了，后来后端给我数据字段的时候我都懵B了，40多个字段。。
- **坑1：渲染表格字段时，写成循环渲染，用v-for来实现**
- **坑2：el-table标签，确定了要绑定的数据后，最好单独写一个数据对象来保存需要渲染的列名，进行渲染时注意不要把数据源和列名搞混了**
- **坑3：el-table-column标签，只需要绑定上prop属性就能进行渲染了，label属性是出现在表头的列字段名**

##### 绑定数据并循环渲染列名

直接贴一段代码

    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      style="width: 100%"
      v-show="dataPageVisible">
      <el-table-column
        header-align="center"
        align="center"
        v-for="(value, key, index) in renderList"
        :key="index"
        :label="key"
        :prop="key"
        >
      </el-table-column>
    </el-table>

另外贴一下dataList和renderList的数据结构

	let dataList = [
		{"key1": "value", "key2": "value", ...},
		{...},
		{...}
	]

	let renderList = {
		"key1": "",
		"key2": "",
		...
	}

renderList里保存了需要渲染的列名，dataList保存了全部数据；

而在`el-table`的属性`data`中绑定了dataList，`el-table-column`的属性`prop`绑定的是需要渲染的字段名（可以是中文和英文），一般来说是`dataList`的`key`值，因为是对dataList进行渲染，只要绑定了这俩属性，element-ui会自动进行内部的循环渲染，这意味着即使数据有好多条记录（好多行），都能自动的渲染，不需要前端再做处理（这个问题让我纠结了好久，我原来一直认为循环column是对数据进行循环。）；

总结一下`<el-table>`标签的使用，

- 将要渲染的数据绑定到`data`属性上；
- 建议单独新建一个数据对象保存需要渲染的表头，数据结构可以是数组，也可以是对象，用`v-for`进行循环渲染`<el-table-column>`，`prop`属性绑定数据的`key`，`label`绑定数据的字段名（一般是中文字段名）；
- 最后要注意一下渲染数据的数据结构，因为现在还没有看element-ui底层的代码，所以也不知道是怎么进行循环渲染的，现在确定能渲染的都是官方文档给出的数据结构。

##### 上传附件

组件使用`<el-upload>`，官方文档[http://element-cn.eleme.io/#/zh-CN/component/upload#upload-shang-chuan](http://element-cn.eleme.io/#/zh-CN/component/upload#upload-shang-chuan)

在官方文档中，对这个标签的属性介绍很详尽了。

##### 下载附件

思路，新创建一个`<a>`标签，利用H5的新属性`download`，模拟一次点击`<a>`标签，然后提示用户下载。要注意的点就是需要对内容进行URI编码，以及`href`属性，一般这个属性指向需要跳转的链接，但是这里处理成了页面内容；对于`data:...`的详细内容可以在[https://www.cnblogs.com/augustine/p/3669644.html](https://www.cnblogs.com/augustine/p/3669644.html)这篇博客看到；

	download(filename, text) {
        var element = document.createElement('a');
        element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(text));
        element.setAttribute('download', filename);
        element.style.display = 'none';
        document.body.appendChild(element);
        element.click();
        document.body.removeChild(element);
      }

##### 关于Vue.cookie

目录`src/main.js`先引入`vue-cookie`，还需要在`new Vue()`之前`Vue.use(VueCookie)`，全局引入之后就可以在其他组件里使用了，具体用法：`this.$cookie.set('key', value)`，`this.$cookie.get('key')`；

##### 阻止浏览器自动填入表单

一些情况下，我们不想让浏览器自动填入保存的表单信息，可以在`el-input`添加属性`readonly`，在初始化时置为`true`，延迟一秒后再置为`false`，这样可以让浏览器无法自动填充这个输入框。代码如下：

	<template>
		<el-input :readonly="readonly" type='text'></el-input>
	</template>
	<script>
		data: (){
			return{
				readonly: true
			}
		},
		methods: {
			init() {
				setTimeout(() => {
          		this.readonly = false;
        		}, 1000)
			}
		}
	</script>