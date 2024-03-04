# Vue2语法

## Vue环境搭建

#### vscode插件和浏览器插件

- vue devtools: 浏览器插件
- vetur: vue2代码补全插件
- volar: vue3代码补全插件

#### 创建一个vue实例

- 准备容器:Vue管理的范围(`<div id="app"></div>`)
- 引入包:官网开发版本(完整的注释和警告)/生产版本(没有警告)
- 创建Vue对象:new Vue({})

## Vue2框架

- 指令配置项:渲染数据
  - el:指定挂载点
  - data:提供数据
  - methods:提供方法
  - computed:计算属性(提供缓存)
  - watch侦听器:检测元素变化,进行处理(eg:翻译,购物车本地存储)
  - components:局部注册组件

```vue
<div id="app">
    {{ msg }}
</div>
<script src="https://v2.cn.vuejs.org/js/vue.js"></script>
<script>
	const app = new Vue({
        el:'#app',
        data:{ msg:'Hello World'},
        methods:{ fn(){this.msg='Bonjour'} },
        computed:{
           totalNum(){
                get(){ return xxx;},//获取计算属性的值(eg:总和)
                set(value){}//设置其他依赖属性应该怎么变化(eg:全选),之后根据其他属性的值再计算出该属性的值
                //set:1.改变其他属性 2.再重新get
           }
        },
        watch:{
            fruitList:{
                deep: true,//深度监视复杂数据类型变化(eg:对象的属性变化而地址不变)
				immediate: true,//是否立刻执行一次handler
                handler (newValue,oldValue) {
                	localStorage.setItem('list',JSON.stringify(newValue))
                }
            }
        },
        components:{//局部注册组件
            HmHeader:HmHeader,
            HmMain:HmMain,
            HmFoot:HmFoot,
        },
        钩子函数(){}
    })
</script>
```

## Vue2指令

#### 插值表达式{{  }}

- 用于**存储内容信息**,作为标签的body部分(不会解析html标签)
- 不能出现复杂的多行语句,可以使用三目运算符(?:)
- 用于**构建响应式动态网页**

#### Vue2指令基础

位于标签的位置,但是不是普通的标签而是vue标签(以v-开头)

```vue
v-html="表达式":设置元素的innerHTML
v-bind:属性名="表达式": 动态设置属性值
	  :属性名="表达式": 动态设置属性值

v-show="表达式":表达式为true显示,表达式为false隐藏(display:none)
v-if="表达式":表达式为true元素存在,表达式为false元素被删除
v-else-if="表达式"
v-else
频繁切换显示隐藏的场景适合使用v-show(节省成本开销)(eg:购物车弹窗)
要么显示要么隐藏,不会频繁切换的场景(eg:登陆提示)
<div id="app">
    <p v-if="gender === 1">男 man</p>
    <p v-else>女 women</p>
</div>

v-on:事件名(当前元素添加监听)="内联语句"
@事件名="内联语句"
<button v-on:click="count+=2">+</button>
<button @click="count+=2">+</button>
@事件名="函数(参数)"
<h3>小黑自动售货机</h3>
<button @click="buy(+5)">可乐5元</button>
<button @click="buy(+10)">咖啡10元</button>
<p>银行卡余额{{ money }}元</p>
buy(cost){
app.money -= cost
}

v-for="数组/对象/数字": 基于数据循环,多次渲染整个元素
v-for的key: 对数据进行唯一标识,防止原地修改元素(样式没有跟着被删除).key只能是字符串或者数字
<ul><!-- 循环生成列表元素 -->
    <li v-for="(item,index) in list" :key="item.id">{{item}} - {{index}}</li>
</ul>
------循环生成图片src(标签)
<div class="img-container" v-for="index in files" :key="index">
    <img :src="index" class="img_display">
</div>
------循环生成innerHTML
<div v-for="item in items" :key="item.id" v-html="item.content"></div>

v-model: 给表单元素使用双向数据绑定(获取或者设置元素内容)
<input type="text" name="uname" v-model="username">
data:{username:""}
```

- 注意事项:
  - 使用v-for的时候最好搭配:key使用
  - @事件最好搭配使用.prevent,否则默认行为不受控制

#### Vue2指令进阶

- 使用v-bind设置类名
  - 对象:适用于一个类名来回切换
  - 数组:批量添加或者删除类

```vue
:class="{类名1:布尔值1,类名2:布尔值2}"
:class="[类名1,类名2,类名3]"
```

- 使用v-bind设置样式

```
:style="{ CSS属性名1: CSS属性值, CSS属性名2: CSS属性值 }"
```

- 指令修饰符:

```vue
按键修饰符:
	@keydown.enter
v-model修饰符:
	v-model.trim:去除前后空格
	v-model.number:如果是数字就转换为数字
```

## 生命周期

beforeCreate: new Vue()执行之前,data中的变量还没有初始化

**created**: data中的变量已经初始化,这个阶段用来**发送网络初始化请求获取动态数据**

beforeMount: 渲染前,dom元素还没有初始化

**mounted**: dom元素已经被初始化,可以操作dom元素(eg:添加eventListener之类)

beforeUpdate: 视图更新前(data中变量已经改变,dom还没有变化)(会执行多次无限循环)

updated: 元素更新后(dom变化完成)(会执行多次无限循环)

**beforeDestroy**: 销毁vue前执行,**用来释放资源**(定时器,延时器...)

```vue
<script>
const app = new Vue({
	data:{fruitList=null},
    methods:{},
    async created(){
        const res = await axios(...)
        this.fruitList = res//发送网络请求初始化变量
    },
    mounted(){
    	this.myChart = echarts.init(document.getElementById('main'));
        this.myChart.setOption({...});//在dom渲染完之后选择并修改dom元素
	}
})
</script>
```





# Vue2工程化开发

## 工程环境搭建

#### 搭建vue2脚手架

- 搭建nodejs环境(npm)
- 安装yarn: npm i -g yarn

```bash
yarn global add @vue/cli #全局安装vue2
vue --version #查看vue版本,确保安装成功
vue create project-name #创建项目架子
yarn serve / npm run serve #启动项目(在package.json中定义的serve)
```

- 脚手架工作原理:
  - index.html作为整个代码的支架
  - main.js初始化vue定义渲染方法,读取组件将代码注入到index.html中
  - 子组件(全局组件)作为一个整体export,被父组件import之后代码注入到父组件中去

#### 组件管理

- 好处: 1.方便管理(定位错误,修改) 2.方便复用

- 组件代码结构:

  - template: html模板
  - script: vue代码

  - style: 样式

```vue
<template>
	<div class="app">
		<HmHeader></HmHeader>
		<HmMain></HmMain>
		<HmFoot></HmFoot>
	</div>
</template>
<script>
import HmHeader from './components/HmHeader.vue'
import HmMain from './components/HmMain.vue'
import HmFoot from './components/HmFoot.vue'//导入组件
export default {//导出当前vue组件
	components:{//局部注册组件
		HmHeader:HmHeader,
		HmMain:HmMain,
		HmFoot:HmFoot,
	}
}
</script>
<style>
.app{
	width: 1280px;
	height: 720px;
	background-color: rgb(192, 212, 255);
}
</style>
```

## 组件化编程

- 使用<vue快速生成组件模板

#### 导出组件

- 使用 `export default{配置项}` 默认导出组件
- 导出组件的时候会根据三个部分(script,template,style)整体生成js进行导出
- import的时候会找到导入文件中的export部分,将对象进行导入

#### 导入+注册组件

- 注册:
  - 局部注册:父组件的components中进行注册,适用于只在当前父组件中使用的子组件
  - 全局注册:main.js中进行注册,适用于整个页面中都有可能会用到的组件(eg:按钮)

```vue
<script>
//导入组件
import HmHeader from './components/HmHeader.vue'
import HmMain from './components/HmMain.vue'
import HmFoot from './components/HmFoot.vue'
//局部注册
export default {
	components:{
		HmHeader:HmHeader,
		HmMain:HmMain,
		HmFoot:HmFoot,
	}
}
</script>
```

```javascript
//main.js
//导入HmButton组件
import HmButton from './components/HmButton.vue'
//引入全局注册
Vue.component('HmButton',HmButton)
```

# Vue2路由

## 安装配置

- vue2使用vue-router3版本,vue3使用vue-router4版本
- 安装代码:

```shell
yarn add vue-router@3.6.5
import VueRouter from 'vue-router'
```















