<template>
	<view style="height:calc(100vh);">
		<view class="back-btn iconfont iconarrow_btn_left" @click="navBack">
			<text class="nav-title">恭喜您获得</text>
		</view>
		<fireworks />
		<view style="width: 100vw;height:calc(100vh);position: relative;">
			<image class="bg-img" style="width: 100vw;height:calc(100vh);" src="/static/images/box/bg.png" mode=""></image>
			<view style="position:absolute;top: 182rpx;left: 0;width: 100%;">
				<image style="width: 100vw;position:absolute;top: 342rpx;height: 685rpx;" src="/static/images/box/bg-show.png" mode=""></image>
				<view class="image-item flex justify-center padding-top-lg" style="z-index: 999;">
					<image class="image" :src="blind_box.goods_thumb" :class="[animation=='shake'?'animation-shake':'',toggleDelay?'animation-slide-bottom':'']"
					  :style="[{animationDelay: '0.3s'}]" mode="widthFix"></image>
				</view>
				<view class="text-white padding" style="position:absolute;top: 668rpx;z-index: 999;">
					<view class="text-xxl text-orange">{{blind_box.type_name}}</view>
					<view class="text-df padding-top-sm" style="width: 348rpx;margin: 0 auto;">
						{{blind_box.goods_name}}
					</view>
				</view>
				<view class="ui-tabbar-view-box sm-border padding-bottom">
					<view class="flex justify-center mb-sm margin-sm">
						<button class="cu-btn bg-gradual-orange round lg" @tap="prizeTap" style="width: 430rpx;">我的盒柜</button>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import fireworks from '@/components/fireworks/fireworks.vue';
	export default {
		data() {
			return {
				animation:'',
				blind_box:{},
				toggleDelay: false,
				prize: {},
				isReady: false,
				share:{}
			}
		},
		components: {
			fireworks
		},
		onReady() {
			uni.hideLoading();
			this.$nextTick(() => {
				setTimeout(() => {
					this.isReady = true;
				}, 50)
			});
		},
		onLoad(e) {
			uni.showLoading({
				title: '加载中...',
				mask: true
			});
			this.base_init(e);
			this.page_init(e);
		},
		onShareAppMessage(res) {
			return {
				title: this.share.title || '运气爆棚！我抽中了' + this.blind_box.goods_name,
				path: '/pages/activity/blind_box?id=' + this.blind_box.id,
				imageUrl:  this.share.thumb || this.blind_box.goods_thumb,
				success(res) {
					uni.showToast({
						title: '分享成功'
					})
				},
				fail(res) {
					uni.showToast({
						title: '分享失败',
						icon: 'none'
					})
				}
			}
		},
		onShow(e) {
			setInterval(() => {
			     this.animation= 'shake';
				 setTimeout(()=>{
				 	this.animation= '';
				 }, 1000)
			},4300);
		},
		methods: {
			page_init(e) {
				var params = {
					module: 'blind_box_burst'
				}
				this.$Http.get('/page/init', params).then(res => {
					if (res.statusCode == 200 && res.data) {
						var data = {
							title: res.data.name,
							thumb: res.data.thumb,
						}
						this.share = data;
					}
				});
			},
			base_init(e) {
				var params = {
				  order_id: e.order_id
				}
				this.$Http.get('/blind_box/prize', params).then(res => {
				  if (res.statusCode !== 200) {
					uni.showToast({
					  title: res.message,
					  icon: 'none'
					});
					return false;
				  }
				  
				  this.blind_box = res.data;
				})
			},
			// 返回上一页
			navBack() {
				uni.navigateBack();
			},
			prizeTap(){
				var options = {
					url: '/pages/activity/cabinet'
				}
				this.navigateTo(options, 1);
			}
		}
	}
</script>

<style>
	.back-btn {
		width: 100%;
		position: absolute;
		z-index: 999;
		padding-top: var(--status-bar-height);
		padding-left: 40upx;
		padding-right: 40upx;
		top: 40upx;
		font-size: 40upx;
		color: #1E1E1E;
		display: flex;
		align-items: center;
		/* .nav-title {
			width: 590upx;
			text-align: center;
		} */
	}
	
	.image{
		width: 480rpx;
		height: 480rpx;
	}
	
	.ui-tabbar-view-box {
		position: fixed;
		bottom: 120rpx;
		left: 0;
		width: 100%;
	}
</style>
