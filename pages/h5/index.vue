<template>
	<view>
		<web-view :webview-styles="webviewStyles" :src="web_url"></web-view>
	</view>
</template>


<script>
	export default {
		data() {
			return {
				webviewStyles: {
					progress: {
						color: '#FF3333'
					}
				},
				web_url:''
			}
		},
		onLoad(option) {
			if(option.module === 'login'){
				if(option.type == 'user'){
					this.web_url = 'https://api.easyshop.org.cn/api/rich/resolve?type=user';
				} else {
					this.web_url = 'https://api.easyshop.org.cn/api/rich/resolve?type=privacy';
				}
			}
			
			if(option.module === 'blind_box'){
				this.web_url = 'https://api.easyshop.org.cn/api/rich/resolve?module=blind_box&type=rule';
			}
		}
	}
</script>

<style>

</style>
