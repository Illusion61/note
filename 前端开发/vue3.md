# Vue3

#### 创建vue项目

```
npm init vue@latest
```

- 项目参数配置
  - 输入项目名称:**不要存在大写字母!**
  - 

```
cd 项目名称
npm i
npm run dev
```

#### 项目目录结构

```
项目名称:
--->public文件夹:存储资源文件
--->src文件夹:存储前端页面源码
--->vite.config.js:Vue配置文件
```







## Vue3的UMD模式使用naive

```html
<html>
	<head>
		<meta charset="utf-8" />
		<script src="https://unpkg.com/vue"></script>
		<!-- 会使用最新版本，你最好指定一个版本 -->
		<script src="https://unpkg.com/naive-ui"></script>
	</head>
	<body>
		<div id="app">
			<n-button>{{message}}</n-button>
		</div>
		<script>
			const { createApp, h } = Vue;
			const { NButton } = naive;
			const app = Vue.createApp({
				data() {
					return {
						message: 'Hello Vue 3!'
					};
				}
			});
			app.use(naive);
			app.mount('#app');
		</script>
	</body>
</html>
```

