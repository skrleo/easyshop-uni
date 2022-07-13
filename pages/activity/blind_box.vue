<template>
	<view style="overflow-x: hidden;background-color: #FFFFFF;height:calc(100vh)">
		<barrage ref="lffBarrage"></barrage>
		<scroll-view scroll-y="true">
			<view class="tag-tip">
				<view class="bg-blue flex justify-center" style="border-radius: 10rpx;">
					<view class="cu-tag bg-blue" @tap="ruleTap">规则</view>
				</view>
				<view class="margin-top-xs bg-red flex justify-center">
					<view class="cu-tag bg-blue" @tap="cabinetTap">盒柜</view>
				</view>
			</view>
			<view class="info">
				<view class="exhibit">
					<swiper class="screen-swiper square-dot" :indicator-dots="true" :circular="true"
					 :autoplay="true" interval="5000" duration="500">
						<swiper-item v-for="(item,index) in blind_box.blind_box_rule" :key="index">
							<view class="cu-card case no-card">
								<view class="shadow">
									<!-- <view style="position:absolute;">
										<image class="go" style="width: 456rpx;margin-left: calc(50vw - 208rpx);transform:rotate(-180deg);transition: all 2s;" src="/static/images/box/ligth.png" mode="widthFix"></image>
									</view> -->
									<view class="image">
										<image :src="item.goods_thumb"
										 :class="[animation=='shake'?'animation-shake':'',toggleDelay?'animation-slide-bottom':'']"
										  :style="[{animationDelay: '0.3s'}]" mode="widthFix"></image>
									</view>
									<view class="cu-list margin-top-xl">
										<view class="flex padding justify-center">
											<view class="text-center text-lg" style="width: 480rpx;">
												<text>{{item.goods_name}}</text>
											</view>
										</view>
										<view class="flex justify-center">
											<view class="text-center text-lg" style="width: 480rpx;">
												<cn-money :money="item.goods_price" :size="43"></cn-money>
											</view>
										</view>
									</view>
								</view>
							</view>
						</swiper-item>
					</swiper>
					
					<view class="flex justify-center">
						<view class="text-center text-xs padding-left-xs" v-for="(item, index) in blind_box.type_group" :key="index">
							<text class="cuIcon-title" :class="'text-' + item.color"></text>
							<text>{{item.title}}: {{item.rate}} %</text>
						</view>
					</view>
				</view>
				<view class="flex justify-center margin-top-sm padding-tb">
					<view class="text-center text-df text-black" style="width: 480rpx;">
						<button class="cu-btn cuIcon sm bg-red">
							<text class="cuIcon-like"></text>
						</button>
						<text class="margin-left-sm text-bold text-black">以下商品随机得一件</text>
					</view>
				</view>
				<scroll-view scroll-x>
					<view class="recharge">
						<view class="recharge-item recharge-item-active margin-right-xs" @tap="goodsTap(item)" v-for="(item, index) in blind_box.blind_box_rule" :key="index" :style="{marginLeft: !index ? '30rpx': ''}" @click="rechargeChange(index)">
							<view class="recharge-tag">
								<text class="recharge-tag-text">{{item.type_name}}</text>
							</view>
							<view class="recharge-item-thumb" style="overflow: hidden;">
								<image style="width: 240rpx;height: 240rpx;" :src="item.goods_thumb" mode="widthFix"></image>
							</view>
							<view class="text-center text-cut" style="padding: 6rpx;">
								{{ item.title }}
							</view>
							<view class="text-price text-bold text-black" style="padding: 12rpx;">{{item.goods_price}}</view>
						</view>
					</view>
				</scroll-view>
			</view>
		</scroll-view>
		<!-- 底部操作栏 -->
		<view class="bg-white ui-tabbar-view-box sm-border padding-bottom">
			<view class="flex justify-center mb-sm margin-sm">
				<button class="cu-btn bg-gradual-orange round lg" @tap="BuyTap" style="width: 430rpx;">￥{{blind_box.price}} 开盲盒</button>
			</view>
		</view>
		<!--弹出框-->
		<modal-box :show="isShow" :prize="prize" @prizeTap="prizeTap"/>
	</view>
</template>

<script>
	import barrage from '@/components/barrage/barrage.vue';
	import modalBox from '@/components/basics/modal-box.vue';
	const statusBarHeight = uni.getSystemInfoSync().statusBarHeight
	export default {
		components: {
			modalBox,
			barrage
		},
		data() {
			return {
				animation:'',
				blind_box:{},
				toggleDelay: false,
				statusBarHeight,
				current: 0,
				isShow: false,
				prize: {},
				share:{}
			}
		},
		onLoad(e) {
			uni.showLoading({
				title: '加载中...',
				mask: true
			});
			this.base_init(e);
			this.page_init(e);
		},
		onShow(e) {
			setInterval(() => {
			     this.animation= 'shake';
				 setTimeout(()=>{
				 	this.animation= '';
				 }, 1000)
			},4300);
		},
		onReady() {
			uni.hideLoading()
		},
		onShareAppMessage(res) {
			return {
				title: this.share.title || this.blind_box.title,
				path: '/pages/activity/blind_box?id=' + this.blind_box.id,
				imageUrl: this.share.thumb || this.blind_box.thumb,
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
		methods: {
			page_init(e) {
				var params = {
					module: 'blind_box'
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
					id: e.id
				}
				this.$Http.get('/blind_box/detail', params).then(res => {
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
			colrdo(){ //插入一条弹幕
				this.$refs.lffBarrage.add({item:'你好呀小伙子'});
			},
			goodsTap(e){
				uni.navigateTo({
					url: '/pages/goods/detail?goods_id=' + e.goods_id
				});
			},
			BuyTap() {
				this.checkAuth(() => {
					var params = {
						box_id: this.blind_box.id
					};
					this.$Http.get('/blind_box/open', params).then(res => {
						if (res.statusCode !== 200) {
							uni.showToast({
								title: res.message,
								icon: 'none'
							});
							return false;
						}
						var response = res.data;
						uni.requestSubscribeMessage({
							tmplIds: ['hVAFmQd4CLi6MHyiJXfmtKG20GIRpc1Xr3Iz079XDKo'],
							success: res => {

							},
							complete: res => {
								uni.requestPayment({
									provider: 'wxpay',
									nonceStr: response.nonceStr,
									package: response.package,
									paySign: response.paySign,
									signType: response.signType,
									timeStamp: response.timeStamp,
									success: info => {
										this.prize = response.prize;
										setTimeout(()=>{
											uni.navigateTo({
												url: "/pages/activity/burst?order_id=" + response.order.order_id
											})
										}, 1000)
									},
									fail: err => {
										uni.showToast({
											title: '取消支付',
											icon: 'none'
										});
										return false;
									}
								});
							}
						})
					})
				});
			},
			ruleTap(){
				uni.navigateTo({
					url: "/pages/h5/index?type=rule&module=blind_box"
				})
			},
			cabinetTap(){
				this.checkAuth(() => {
					var options = {
						url: '/pages/activity/cabinet'
					}
					this.navigateTo(options, 1);
				});
			},
			ToggleDelay() {
				this.toggleDelay= true;
				setTimeout(()=>{
					this.toggleDelay= false
				}, 1000)
			},
			// 初始化towerSwiper
			TowerSwiper(name) {
				let list = this[name];
				for (let i = 0; i < list.length; i++) {
					list[i].zIndex = parseInt(list.length / 2) + 1 - Math.abs(i - parseInt(list.length / 2))
					list[i].mLeft = i - parseInt(list.length / 2)
				}
				this.swiperList = list
			},
			
			// towerSwiper触摸开始
			TowerStart(e) {
				this.towerStart = e.touches[0].pageX
			},
			
			// towerSwiper计算方向
			TowerMove(e) {
				this.direction = e.touches[0].pageX - this.towerStart > 0 ? 'right' : 'left'
			},
			
			// towerSwiper计算滚动
			TowerEnd(e) {
				let direction = this.direction;
				let list = this.swiperList;
				if (direction == 'right') {
					let mLeft = list[0].mLeft;
					let zIndex = list[0].zIndex;
					for (let i = 1; i < this.swiperList.length; i++) {
						this.swiperList[i - 1].mLeft = this.swiperList[i].mLeft
						this.swiperList[i - 1].zIndex = this.swiperList[i].zIndex
					}
					this.swiperList[list.length - 1].mLeft = mLeft;
					this.swiperList[list.length - 1].zIndex = zIndex;
				} else {
					let mLeft = list[list.length - 1].mLeft;
					let zIndex = list[list.length - 1].zIndex;
					for (let i = this.swiperList.length - 1; i > 0; i--) {
						this.swiperList[i].mLeft = this.swiperList[i - 1].mLeft
						this.swiperList[i].zIndex = this.swiperList[i - 1].zIndex
					}
					this.swiperList[0].mLeft = mLeft;
					this.swiperList[0].zIndex = zIndex;
				}
				this.direction = ""
				this.swiperList = this.swiperList
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

<style lang="scss">
	
	@keyframes rotate{
	    0%{ transform: (0deg) }
	    100%{transform:(360deg)}
	}
	
	.go{
		animate: rotate 1s;
	}
	
	.exhibit{
		width: 100%;
		height: 100%;
		justify-content: center;
		// background-image: url('~@/static/images/box/main-box.jpg');
		background-size: 100%;
		margin-top: 180rpx;
	}
	
	.cu-card.case.no-card .image {
		width: 350rpx;
		height: 350rpx;
		justify-content: center;
		background-image: url('~@/static/images/ligth.png');
		background-size: 100%;
		margin: 50rpx auto !important;
	}
	
	.exhibit .screen-swiper {
	    min-height: 711rpx;
	}
	
	.ui-tabbar-view-box {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
	}
	
	.tower-swiper .tower-item {
		transform: scale(calc(0.5 + var(--index) / 10));
		margin-left: calc(var(--left) * 100upx - 150upx);
		z-index: var(--index);
	}
	
	.main-title {
		padding: 30rpx 30rpx;
		font-size: 34rpx;
		color: #1C1C1C;
	}
	.tag-tip{
		position: absolute;
		top: 38rpx;
		right: 0;
		z-index: 99;
	}
	
	.tip {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
		font-size: 24rpx;
		color: #A5A3A2;
	}
	
	.rmb {
		font-size: 26rpx;
		color: #E3BE83;
	}
	
	.bg {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		// height: 400rpx;
		z-index: -2;
		
		&-img {
			width: 100%;
			height: 100%;
		}
	}
	
	.hover {
		opacity: 0.7;
	}
	
	.info {
		position: relative;
		// padding: 0 15rpx;
		width: 100%;
		min-height: 1060rpx;
		bottom: 150rpx;
		
		&-bg {
			position: absolute;
			width: 100%;
			height: 256rpx;
			z-index: -1;
		}
		
		&-content {
			padding: 70rpx 50rpx 0 50rpx;
			margin-bottom: 50rpx;
			display: flex;
			flex-direction: row;
			align-items: center;
		}
		
		&-avatar {
			margin-right: 30rpx;
			display: flex;
			flex-direction: row;
			align-items: center;
			justify-content: center;
			width: 85rpx;
			height: 85rpx;
			background-color: #fff;
			padding: 1rpx;
			border-radius: 50%;
			
			&-pic {
				width: 83rpx;
				height: 83rpx;
			}
		}
		
		&-name {
			display: flex;
			flex-direction: column;
			color: #FFFFFF;
			font-size: 35rpx;
		}
		
		&-level {
			margin-top: 15rpx;
			font-size: 24rpx;
		}
		
		&-asset {
			padding: 0 50rpx;
			display: flex;
			flex-direction: row;
			align-items: center;
			
			&-item {
				display: flex;
				flex-direction: column;
			}
		}
	}
	
	.iai {
		&-title {
			margin-bottom: 20rpx;
			margin-right: 100rpx;
			font-size: 24rpx;
			color: #CFB386;
		}
		
		&-value {
			font-size: 35rpx;
			color: #FFFFFF;
		}
	}
	
	.recharge {
		position: relative;
		margin-bottom: 35rpx;
		display: flex;
		flex-direction: row;
		align-items: center;
		
		&-tag {
			position: absolute;
			top: -2rpx;
			z-index: 999;
			left: -2rpx;
			width: 170rpx;
			height: 36rpx;
			display: flex;
			flex-direction: row;
			align-items: center;
			justify-content: center;
			background-image: url('~@/static/images/tag.png');
			background-size: 100%;
			
			&-text {
				font-size: 20rpx;
				color: #FFFFFF;
				text-align: center;
			}
		}
		
		&-item {
			position: relative;
			overflow: hidden;
			margin-left: 15rpx;
			width: 250rpx;
			height: 310rpx;
			flex-shrink: 0;
			display: flex;
			flex-direction: column;
			align-items: center;
			border: solid 2rpx #F2F2F3;
			border-radius: 12rpx;
			
			&-thumb {
				padding: 5px;
				width: 240rpx;
				height: 240rpx;
			}
			
			&-active {
				border-color: #EDD2A9;
				background-color: #FBF1E5;
			}
			
			&-duration {
				margin-bottom: 30rpx;
				font-size: 26rpx;
				color: #1C1C1C;
			}
			
			&-price {
				margin-bottom: 20rpx;
				display: flex;
				flex-direction: row;
				align-items: baseline;
				
				&-text {
					font-size: 48rpx;
					color: #E3BE83;
				}
			}
			
			&-des {
				font-size: 22rpx;
				color: #A5A3A2;
			}
		}
		
		&-item:last-child{
			margin-right: 15rpx;
		}
	}
	
	.button {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
		height: 85rpx;
		border-radius: 50rpx;
		background-image: linear-gradient(#EFCF9E, #E4BF85);
		
		&-text {
			font-size: 30rpx;
			color: #1C1C1C;
		}
	}
	
	.update {
		margin: 35rpx 30rpx 20rpx 30rpx;
	}
	
	.privilege {
		padding: 0 30rpx;
		margin-bottom: 40rpx;
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: space-around;
		flex-wrap: wrap;
		
		&-item {
			margin: 0 23rpx;
			margin-bottom: 25rpx;
			display: flex;
			flex-direction: column;
			align-items: center;
			
			&-pic {
				width: 80rpx;
				height: 80rpx;
				margin-bottom: 20rpx;
			}
			
			&-text {
				font-size: 24rpx;
				color: #383738;
			}
		}
	}
	
	.level {
		position: relative;
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
		width: 750rpx;
		height: 466rpx;
		background-image: url('~@/static/images/vip-bg.png');
		background-size: 100%;
		
		&-rate {
			position: absolute;
			top: 43rpx;
			left: 33rpx;
			display: flex;
			flex-direction: row;
			align-items: center;
			
			&-text1 {
				margin-right: 20rpx;
				color: #1C1C1C;
				font-size: 34rpx;
			}
			
			&-text2 {
				font-size: 34rpx;
				color: #C07C07;
			}
		}
		
		&-info {
			// position: absolute;
			// top: 188rpx;
			// left: 350rpx;
			margin-top: 70rpx;
			display: flex;
			flex-direction: column;
			align-items: center;
			
			&-icon {
				margin-bottom: 20rpx;
				width: 48rpx;
				height: 38rpx;
			}
			
			&-text {
				margin-bottom: 20rpx;
				font-size: 26rpx;
				color: #292929;
			}
			
			&-integral {
				font-size: 40rpx;
				color: #C07C07;
			}
		}
		
		&-distance {
			position: absolute;
			left: 30rpx;
			right: 30rpx;
			bottom: 35rpx;
			display: flex;
			flex-direction: row;
			align-items: center;
			justify-content: space-between;
			
			&-text {
				font-size: 26rpx;
				color: #292929;
			}
			
			&-button {
				padding: 0 30rpx;
				height: 60rpx;
			}
		}
	}
</style>
