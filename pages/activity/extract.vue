<template>
	<view class="cu-card article">
		<view class="bg-white desc margin-lr-sm margin-top-sm padding-sm radius">
			<view class="flex justify-between align-center" @tap="addressTap()">
				<view v-if="address_info">
					<view class="padding-bottom-xs text-black">
						<text class="text-lg">{{address_info.contact}}</text><text class="text-l padding-left-xs">{{address_info.moblie}}</text>
					</view>
					<view class="padding-tb-xs align-center">
						<button class="cu-tag bg-black radius sm margin-right-sm" v-if="address_info.is_default == 1">默认</button>
						<text class="through">{{address_info.province}}{{address_info.city}}{{address_info.region}}{{address_info.address}}</text>
					</view>
				</view>
				<view class="padding-tb" v-else>
					<view class="padding-bottom-xs text-l text-black align-center">
						请选择收货地址
					</view>
				</view>
				<view class="line-height text-xl">
					<view class="cuIcon-right text-black"></view>
				</view>
			</view>
		</view>
		
		<view class="margin-top-sm radius">
			<view class="car-list radius">
				<view class="cu-item list padding-sm">
					<image class="car-img" :src="blind_box.goods_thumb" mode="aspectFill"></image>
					<view class="car-mes centerboth">
						<view class="mes-box">
							<view class="g-name line2">{{blind_box.goods_name}}</view>
							<view class="gz-box text-sm" v-if="item.sku_name">规格：<text>{{item.sku_name}}</text></view>
							<view class="flex justify-between align-center margin-top-sm">
								<view>
									<cn-money :money="blind_box.goods_price" :size="36"></cn-money>
								</view>
								<view class="num-box centerboth">
									<view>x {{blind_box.number}}</view>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
			<view class="text-black car-list text-df sm-border" style="margin-bottom: 130rpx;">
				<view class="flex justify-between padding-sm bg-white">
					<view>订单运费</view>
					<view><cn-money :money="blind_box.post_fee" :size="24"></cn-money></view>
				</view>
				<view class="flex justify-between padding-sm bg-white">
					<view>商品数量</view>
					<view>{{blind_box.number}}</view>
				</view>
				<view class="flex justify-between padding-sm bg-white">
					<view>发货预估</view>
					<view>1-8工作日内发货</view>
				</view>
				<view class="flex justify-between padding-sm bg-white">
					<view>备注</view>
					<view><input class="padding-left-xs" v-model="remark" placeholder="选填,60字以内"></input></view>
				</view>
			</view>
		</view>
		
		<!-- 底部操作栏 -->
		<view class="car-bottom-btn padding-lg">
			<view class="all-cost centerboth text-df">
				支付金额: <cn-money :money="blind_box.post_fee" :size="42"></cn-money>
			</view>
			<view class="car-btn-box centerboth">
				<button class="cu-btn bg-black margin-right" @tap="submitTap()">立即提交</button>
			</view>
		</view>
	</view>
</template>

<script>
	import cnMoney from '@/components/cn-money/cn-money';
	import { mapState } from 'vuex';
	export default {
		components: {
			cnMoney,
		},
		data() {
			return {
				order_id: 0,
				address_info:{
					id: 0
				},
				order_no: '',
				goods_info: [],
				allNumber: 0,
				allAmount: 0,
				cart_ids: '',
				goods_id: 0,
				number: 0,
				sku_id: '',
				remark: '',
				type: 0,
				blind_box:{}
			}
		},
		onLoad(e) {
			uni.showLoading({
				title: '加载中...',
				mask: true
			});
			this.type = e.type;
			this.order_id = e.order_id;
			if(e.address_id !== undefined){
				this.address_info.id = e.address_id;
			}
			this.base_init(e);
		},
		onReady() {
			uni.hideLoading()
		},
		computed: {
			...mapState({
				loginStatus: state => state.loginStatus,
				user: state => state.user,
			})
		},
		onShow() {
			if (!this.loginStatus) {
				uni.navigateTo({
					url: '/pages/login/index'
				});
				return false;
			}
		},
		methods: {
			base_init(e) {
				var params = {
				  order_id: e.order_id,
				  address_id: this.address_info.id
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
				  this.order_no = this.blind_box.order_no;
				  this.address_info = res.data.address;
				})
			},
			addressTap() {
				uni.navigateTo({
					url: '/pages/address/index?order_type=blind_box&order_id=' + this.order_id
				});
			},
			submitTap() {
				if(this.address_info === null){
					uni.showToast({
						title: '请选择地址',
						icon: 'none'
					});
					return false;
				}
				var params = {
				  remark: this.remark || '',
				  order_id: this.order_id,
				  address_id: this.address_info.id
				}
				this.$Http.get('/blind_box/draw', params).then(res => {
					if (res.statusCode !== 200) {
						uni.showToast({
							title: res.message,
							icon: 'none'
						});
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
									uni.showLoading({
									  title: '跳转中',
									})
									setTimeout(() => {
										uni.redirectTo({
											url: '/pages/order/over?type=box&order_id=' + response.order.order_id + '&order_no=' + response.order.order_no + '&is_success=1' 
										});
									}, 1000);
								},
								fail: err => {
									setTimeout(() => {
										uni.redirectTo({
											url: '/pages/order/over?type=box&order_id=' + response.order.order_id + '&order_no=' + response.order.order_no + '&is_success=0' 
										});
									}, 1000);
								}
							});
						}
					})

				})
			}
		}
	}
</script>

<style lang='scss'>
	.centerboth {
	    display:flex;
	    display: -webkit-flex;
	    align-items:center;
	    -webkit-align-items:center;
	    justify-content: center;
	    -webkit-justify-content: center;
	}
	
	
	.ui-sponsored-card-view {
		position: relative;
		background-color: #FFFFFF;
		/* margin: 7.27rpx 27.27rpx 0; */
	
		.card-price-view {
			position: relative;
			background: #FFF5F5;
			border-radius: 14.54rpx 14.54rpx 0 0;
			padding: 18.18rpx;
	
			.price-left-view {
				position: absolute;
				height: 105.45rpx;
				width: 145.45rpx;
				text-align: center;
				line-height: 125.45rpx;
	
				.price {
					font-size: 45.45rpx;
					font-weight: 400;
				}
			}
	
			.name-content-view {
				position: relative;
				padding-top: 12rpx;
				padding-left: 163.63rpx;
				padding-right: 145.45rpx;
				height: 105.45rpx;
				line-height: 1.8;
				color: #999898;
	
				&::before {
					content: '';
					position: absolute;
					top: -18.18rpx;
					bottom: -18.18rpx;
					margin-left: -18.18rpx;
					border-left: 2rpx dashed #fdbabc;
				}
			}
	
			.btn-right-view {
				position: absolute;
				right: 27.27rpx;
				top: 18.18rpx;
				height: 125.45rpx;
				line-height: 125.45rpx;
			}
		}
	
		.card-num-view {
			position: relative;
			background: #FFECED;
			border-radius: 0 0 14.54rpx 14.54rpx;
			border-top: 2rpx dashed #dedbdb;
			padding: 10.9rpx 17.27rpx;
			color: #999898;
	
			&::before {
				content: '';
				position: absolute;
				width: 36.36rpx;
				height: 36.36rpx;
				background: #ffffff;
				border-radius: 50%;
				top: -18.18rpx;
				left: -18.18rpx;
			}
	
			&::after {
				content: '';
				position: absolute;
				width: 36.36rpx;
				height: 36.36rpx;
				background: #ffffff;
				border-radius: 50%;
				top: -18.18rpx;
				right: -18.18rpx;
			}
	
			view {
				position: relative;
				padding-right: 22.72rpx;
			}
	
			text {
				position: absolute;
				right: 22.27rpx;
				top: 14.54rpx;
			}
		}
	}
	
	.car-add:before{
		content: "\e8a6";
	}
	.car-sub:before{
		content: "\e8a7";
	}
	.car-no:before{
		content: "\e64d";
	}
	.car-unsel:before{
		content: "\e63a";
	}
	.car-sel:before{
		content: "\e639";
	}
	.car-del:before{
		content: "\e622";
	}
	.car-count {
		width: 100%;
		padding-bottom: 120rpx;
	}
	.manage-btn {
		width: 100%;
		height: 75rpx;
		background: #FFFFFF;
		text-align: right;
	}
	
	.manage-btn text {
		line-height: 75rpx;
		padding: 0 24rpx;
	}
	
	.all-sel-btn {
		height: 100%;
		float: left;
	}
	
	.car-list {
		width: 100%;
		padding: 0 20rpx 0 20rpx;
	}
	
	.car-list .list {
		width: 100%;
		padding: 10rpx 15rpx 10rpx 10rpx;
		display: flex;
		background: #FFFFFF;
		border-bottom: 1px solid #F5F5F5;
	}
	.car-list .list:nth-child(1){
		border-top-left-radius: 0;
		border-top-right-radius: 0;
	}
	.car-list .list:nth-last-child(1){
		margin: 0;
	}
	
	.car-list .list .btn {
		width: 70rpx;
		height: 190rpx;
	}

	.car-list .list .car-img {
		width: 190rpx;
		height: 190rpx;
		border-radius: 10rpx;
	}
	
	.car-list .list .car-mes {
		flex: 1;
		margin-left: 20rpx;
	}
	
	.car-list .g-name {
		color: #2a2a2a;
		/* height: 38rpx; */
	}
	
	.car-list .mes-box {
		width: 100%;
	}
	
	.car-list .gz-box {
		color: #999999;
		/* margin: 5rpx 0; */
	}
	
	.car-list .mes-box .price {
		font-weight: bold;
	}
	
	.car-list .price .num {
		font-weight: normal;
		float: right;
	}
	
	.car-list .num-box {
		float: right;
	}
	
	.car-list .num-box view {
		width: 60rpx;
		text-align: center;
		height: 50rpx;
		line-height: 50rpx;
		color: #2A2A2A;
		margin: 0 5rpx;
	}
	
	.price-change-num .price {
		float: left;
	}
	
	.car-bottom-btn {
		position: fixed;
		width: 100%;
		background: #FFFFFF;
		left: 0;
		height: 100rpx;
		bottom: 0;
		z-index: 6;
		box-shadow: 0px -5px 10px -5px #d0d0d0;
		padding: 0 300rpx 0 40rpx;
		display: flex;
		justify-content: space-between;
	}
	.car-btn-box{
		position: absolute;
		right: 0;
		height: 100%;
		z-index: 9;
		top: 0;
	}
	.car-btn-box view{
		width: 150rpx;
		height: 100%;
		cursor: pointer;
	}
	.del-btn{
		background: red;
		color: #FFFFFF;
	}
	.js-btn{
		color: #FFFFFF;
	}
	.shop-mes{
		padding: 20rpx;
		background: #FFFFFF;
		border-bottom: 1px solid #F5F5F5;
	}
	.shop-mes .iconfont{
		margin-right: 20rpx;
	}
	.shop-mes .name-mes{
		float: left;
	}
	
	
	.bottom-modal-box {
		.title-bar {
			position: relative;
			width: 100%;
			.close-icon {
				position: absolute;
				right: 36.36rpx;	
			}
		}
		.modal-content {
			position: relative;
			width: 100%;
			overflow-y: auto;
			height: calc(100vh - 784.54rpx);
			padding: 0 29.09rpx 29.09rpx;
			/* margin-bottom: 118.18rpx; */
			.view-box {
				position: relative;
				width: 100%;
				padding: 0;
				text-align: left;
			}
			.view-box.service {
				.text-view {
					margin-bottom: 18.18rpx;
				}
				.text-list-view {
					position: relative;
					width: 100%;
					/* margin-bottom: 18.18rpx; */
					.text-cut {
						padding-right: 21rpx;
					}
				}
				.text-list-view + .text-view {
					margin-top: 36.36rpx;
				}
			}
			.view-box.promotion {
				.text-view {
					margin-bottom: 18.18rpx;
					.cu-tag {
						position: relative;
						top: -3.63rpx;
					}
				}
				.text-list-view {
					position: relative;
					width: 100%;
					margin-bottom: 18.18rpx;
					.text-cut {
						padding-right: 218.18rpx;
					}
					.text-right-view {
						position: absolute;
						right: 0;
						top: 2rpx;
					}
				}
				.text-list-view + .text-view {
					margin-top: 36.36rpx;
				}
			}
			.view-box.select {
				.cu-list.menu-avatar>.cu-item {
					.content {
						width: calc(100% - 94.54rpx - 59.99rpx);
					}
				}
				.select-btn-list-boox {
					.select-item {
						padding: 18.18rpx 0;
						/* border-bottom: 2rpx solid #f3f3f3; */
						.select-btn {
							position: relative;
							margin-top: 14.54rpx;
							width: 100%;
							.cu-btn {
								font-size: 23.63rpx;
								margin: 5rpx 16.36rpx 5rpx 0;
							}
							.cu-btn.light {
								border: 2rpx solid;	
							}
							.cu-btn[disabled] {
								color: #aaaaaa;
							}
						}
					}
				}
			}
			.footer-fixed {
				padding: 9.09rpx 29.09rpx 29.09rpx;
			}
		}
	}
	.bottom-modal-box.cu-modal.bottom-modal {
		.cu-dialog {
			border-radius: 36.36rpx 36.36rpx 0 0;
		}
	}
</style>
