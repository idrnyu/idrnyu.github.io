---
title: vue-monaco-editor网页嵌入编辑器
date: 2018-04-22 23:01:09
tags: vue
---

<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_10.png" width="40%"/>
## monaco-editor web编辑器
>monaco-editor 是微软出的一条开源web在线编辑器
>支持多种语言，代码高亮，代码提示等功能，与Visual Studio Code 功能几乎相同。

<!-- more -->

>在项目中可能会用带代码编辑功能，或者展示代码。由于代码高亮色比较多，自己调试样式很头痛。所以在网页中嵌入monaco-editor来实现此功能

### 在Vue-cli中使用monaco-editor
##### 运行vue-cli脚手架搭建
``` bash
$ vue init webpack myproject
```
<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_1.png" width="40%"/>
##### 在npm上找到 `vue-monaco-editor` 按提示安装
> --save  保存在发布环境

https://www.npmjs.com/package/vue-monaco-editor
``` bash
$ npm install vue-monaco-editor --save
```
``` bash
$ npm install copy-webpack-plugin --save-dev
```
>默认情况下，monaco-editor使用异步方式从cdn加载require。要使用monaco-editorwebpack 的本地副本，我们需要在构建目录中公开依赖项：

<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_2.png" width="40%"/>
##### 安装完成后配置 vue项目的 `build/webpack.prod.conf.js` 文件
	在头部引入
```javascript
const CopyWebpackPlugin = require('copy-webpack-plugin');
```
<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_4.png" width="40%"/>
	在 `plugins`选项中引入
```javascript
// 复制自定义静态资源
    new CopyWebpackPlugin([
      {
//      from: path.resolve(__dirname, '../static'),
//      to: config.build.assetsSubDirectory,
//      ignore: ['.*']
        from: 'node_modules/monaco-editor/min/vs',
        to: 'vs',
      }
    ]),
```
<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_5.png" width="40%"/>
##### 在vue项目的组建中编写代码
```javascript
<template>
  <div class="code_editor" style="height: 100%;">
  	<div id="language_options">
  		<label>语言: </label>
	  	<select v-model="language">
			  <option v-for="item in languageOptions" v-bind:value="item">
			    {{ item }}
			  </option>
			</select>
  	</div>
  	
    <MonacoEditor
	    :language="language"
	    :code="codes"
	    :options="options"
	    :highlighted="highlightLines"
	    :changeThrottle="500"
	    theme="vs-dark"
	    @mounted="onMounted"
	    @codeChange="onCodeChange"
	    ref="vscode"
	    >
    </MonacoEditor>
  </div>
</template>

<script>
	import MonacoEditor from 'vue-monaco-editor'
	export default {
    components: {
      MonacoEditor
    },
    props: ['codes'],
    data () {
      return {
        code: '//<!-- Type away! -->\n ',		// 代码内容
        language: 'typescript',	 //语言	
        languageOptions: [
        	'typescript','javascript','html','css','bat','c',
        	'coffeescript','cpp','csharp','csp','dockerfile',
        	'fsharp','go','handlebars','ini','java','json','less',
        	'lua','markdown','msdax','mysql','objective-c','pgsql',
        	'php','plaintext','postiats','powershell','pug','python',
        	'r','razor','redis','redshift','ruby','rust','sb','scss',
        	'sol','sql','swift','vb','xml','yaml'
        ],
        highlightLines: [{ number: 0, class: 'red'}],  //高亮
//      theme: ['vs', 'hc-black', 'vs-dark']  // 编辑器样式
        options: {		//选项
          selectOnLineNumbers: false,
				  roundedSelection: false,
				  readOnly: false,		// 只读
				  cursorStyle: 'line',		//光标样式
				  automaticLayout: false,	//自动布局
				  glyphMargin: true,  //字形边缘
				  useTabStops: false
//				  fontSize: 20,		//字体大小
//				  quickSuggestionsDelay: 500,	//代码提示延时
        },
        newCode: '',
      }
    },
    watch: {
    	//  更换语言的时候需要延时后 再销毁并创建
    	language(newLanguage,oldLanguage){
    		this.reload();
    	},
    	// code 父组件传递的 code发生变化就需要重载一次
    	codes(a,b){
				this.reload();
    	}
    },
    methods: {
      //编辑器挂载时触发
      onMounted(editor) {
	      console.log('after mount!', editor, editor.getValue(), editor.getModel());
	      this.editor = editor;
	    },
	    //代码发生变化时触发
	    onCodeChange(editor) {
	      console.log('code changed!', 'code:' + this.editor.getValue());
	    },
	    clickHandler() {
	      console.log('here is the code:', this.editor.getValue());
	    },
	    // 获取代码
	    getcodevalue(){
	    	return this.editor.getValue();
	    },
	    getlanguage(){
	    	return this.language;
	    },
	    // 重载编辑框
	    reload(){
	    	clearTimeout(time);
    		let time = setTimeout(()=>{
		    	this.$refs.vscode.destroyMonaco();   // 销毁
	 				this.$refs.vscode.createMonaco();		// 创建
	 			}, 600);
	    }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
	.code_editor #language_options{
		padding: 16px;
	}
	.code_editor #language_options select{
		height: 33px;
		width: 20%;
	}
</style>

```
<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_6.png" width="40%"/>
##### 运行项目
``` bash
$ npm run dev
```
>此时会报错 提示没有es2015编译器

<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_7.png" width="40%"/>
##### 安装 es2015 保存在开发环境
``` bash
$ npm install  babel-preset-es2015 --save-dev
```
<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_8.png" width="40%"/>

##### 运行项目
``` bash
$ npm run dev
```
<img src="/Dom/imgs/2018_04_22/vue-monaco-editor_10.png" width="40%"/>
