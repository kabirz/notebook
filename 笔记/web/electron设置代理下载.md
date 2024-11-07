#electron 

```shell
npx cross-env ELECTRON_GET_USE_PROXY=true GLOBAL_AGENT_HTTPS_PROXY=http://localhost:3128 npm install electron -D
```

把`GLOBAL_AGENT_HTTPS_PROXY`替换成自己的代理即可下载安装,
参考[链接](https://stackoverflow.com/questions/60054531/how-can-i-solve-the-connection-problem-during-npm-install-behind-a-proxy)

<style>
	.tabs {
		 overflow: hidden;
	}
	.tabs div {
		display: none
	 }
	 .tabs div::first-child {
		 display: block;
	 }
	 .tabs input[type="radio"] {
		  display: none; 
	 }
	 .tabs input[type="radio"]:checked + label + div {
		 display: block;
	 }
</style>

<div class="tabs">
	<div id="tab1">  第一个tab页的内容 </div>
	<div id="tab2">  第二个tab页的内容 </div>
	<div id="tab3">  第三个tab页的内容 </div>
</div>

nihoa



