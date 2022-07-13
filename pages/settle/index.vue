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
			<view class="car-list radius" v-for="(item, index) in goods_info" :key="index">
				<view class="cu-item list padding-sm" :key="item.id">
					<image class="car-img" :src="item.goods_thumb" mode="aspectFill"></image>
					<view class="car-mes centerboth">
						<view class="mes-box">
							<view class="g-name line2">{{item.goods_name}}</view>
							<view class="gz-box text-sm" v-if="item.sku_name">规格：<text>{{item.sku_name}}</text></view>
							<view class="flex justify-between align-center margin-top-sm">
								<view>
									<cn-money :money="item.goods_price" :size="36"></cn-money>
								</view>
								<view class="num-box centerboth">
									<view>x {{item.number}}</view>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
			<view class="text-black car-list text-df" style="margin-bottom: 130rpx;">
				<view class="flex justify-between padding-sm bg-white">
					<view>运费</view>
					<view>包邮</view>
				</view>
				
				<view class="flex justify-between padding-sm bg-white">
					<view>运费保证</view>
					<view><text class="cu-tag line-red sm radius margin-right-sm">赠</text>运费险</view>
				</view>
				
				<view class="flex justify-between padding-sm bg-white" @tap="couponModalTap">
					<view>优惠券</view>
					<view class="text-center">
						<cn-money :money="coupon.coupon_amount" :size="36"></cn-money>
						<text class="cuIcon-right icon-text margin-left-sm" />
					</view>
				</view>
				
				<view class="flex justify-between padding-sm bg-white">
					<view>订单备注</view>
					<view><input class="padding-left-xs" v-model="remark" placeholder="选填,60字以内"></input></view>
				</view>
			</view>
		</view>
		
		<!-- 底部操作栏 -->
		<view class="car-bottom-btn padding-lg">
			<view class="all-cost centerboth text-lg">
				合计:<cn-money :money="allAmount" :size="42"></cn-money>
			</view>
			<view class="car-btn-box centerboth">
				<button class="cu-btn bg-black margin-right" @tap="submitTap()">提交订单</button>
			</view>
		</view>
		
		<!--领券-->
		<view class="cu-modal bottom-modal bottom-modal-box" :class="couponModal?'show':''">
			<view class="cu-dialog bg-white">
				<!--标题-->
				<view class="text-black justify-between margin-tb text-lg title-bar">
					<text>领券</text>
					<text class="cuIcon-close close-icon" @tap="couponModalTap"></text>
				</view>
				
				<!--内容区域-->
				<view class="modal-content">
					<!--商品券-->
					<scroll-view scroll-y="true">
						<view class="ui-sponsored-card-view bg-white" v-if="coupon_lists.length > 0" v-for="(coupon,index) in coupon_lists" :key="index">
							<view class="card-price-view bg-white" style="text-align: left !important;">
								<view class="text-red price-left-view">
									<cn-money :money="coupon.coupon_amount" :size="43"></cn-money>
								</view>
								<view class="name-content-view">
									<view class="text-cut text-red">{{coupon.name}}</view>
									<view class="text-xs">{{coupon.remark}}</view>
								</view>
								<view class="btn-right-view">
									<button class="cu-btn bg-red round sm" @tap="selectTap(coupon)">立即使用</button>
								</view>
							</view>
							<view class="card-num-view bg-white flex justify-between">
								<view class="text-xs padding-left-xs" style="text-align: left !important;" v-if="coupon.full_amount > 0">满{{coupon.full_amount}}可用</view>
								<view class="text-xs" style="text-align: right !important;">{{coupon.effective_start_time}} - {{coupon.effective_end_time}}</view>
							</view>
						</view>
						<view v-else class="text-center padding">
							暂无可用优惠券
						</view>
					</scroll-view>
				</view>
				
				<!--公共按钮-->
				<view class="footer-fixed padding-lg">
					<view class="flex flex-direction">
						<button class="cu-btn bg-black lg" @tap="couponModalTap()">确定</button>
					</view>
				</view>
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
				address_info:{
					id: 0
				},
				goods_info: [],
				allNumber: 0,
				allAmount: 0,
				cart_ids: '',
				goods_id: 0,
				number: 0,
				sku_id: '',
				remark: '',
				type: null,
				coupon:{
					coupon_amount: 0
				},
				coupon_lists: [],
				couponModal: false,
			}
		},
		onLoad(e) {
			this.base_init(e);
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
				var params = {}
				if(e.address_id !== undefined){
					params.address_id = e.address_id;
				}
				if(e.cart_ids !== undefined && e.cart_ids !== ''){
					params.cart_ids = e.cart_ids;
					this.cart_ids = e.cart_ids;
				} else{
					params.goods_id = e.goods_id;
					this.goods_id = e.goods_id;
					params.number = e.number;
					this.number = e.number;
					if(e.sku_id !== undefined){
						params.sku_id = e.sku_id;
						this.sku_id = e.sku_id;
					}
				}
				this.$Http.get('/cart/info', params).then(res => {
					if (res.statusCode !== 200) {
						return false;
					}
					this.goods_info = res.data.goods_info;
					this.address_info = res.data.address_info;
					var allNumber = 0;
					var allAmount = 0;
					res.data.goods_info.forEach(function(item){
						allNumber += item.number;
						allAmount += item.number * item.goods_price;
					});
					this.allNumber = allNumber;
					allAmount = allAmount - this.coupon.coupon_amount;
					this.allAmount = allAmount.toFixed(2);
					this.coupon_lists = res.data.coupon;
				})
			},
			addressTap() {
				uni.navigateTo({
					url: '/pages/address/index?order_type=order&cart_ids=' + this.cart_ids + '&goods_id=' + this.goods_id + '&sku_id=' + this.sku_id + '&number=' + this.number
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
					remark: this.remark,
					address_id: this.address_info.id,
				}
				if(this.cart_ids !== ''){
					params.cart_ids = this.cart_ids;
					params.type = 'shopCart';
				}
				if(this.goods_id !== 0){
					params.goods_id = this.goods_id;
					params.type = 'goods';
				}
				if(this.sku_id !== 'undefined'){
					params.sku_id = this.sku_id;
				}
				if(this.number !== 0){
					params.number = this.number;
				}
				if(this.coupon.length > 0){
					params.coupon_id = this.coupon.id;
				}
				this.$Http.get('/create/pay', params).then(res => {
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
						  		uni.showLoading({
						  		  title: '跳转中',
						  		})
						  		setTimeout(() => {
						  			uni.redirectTo({
						  				url: '/pages/order/over?type=order&order_id=' + response.order.order_id + '&order_no=' + response.order.order_no + '&is_success=1' 
						  			});
						  		}, 1000);
						  	},
						  	fail: err => {
						  		setTimeout(() => {
						  			uni.redirectTo({
						  				url: '/pages/order/over?type=order&order_id=' + response.order.order_id + '&order_no=' + response.order.order_no + '&is_success=0' 
						  			});
						  		}, 1000);
						  	}
						  });
						}
					})
					
				})
			},
			couponModalTap(){
				this.couponModal = !this.couponModal;
			},
			selectTap(e){
				this.coupon = e;
				this.allAmount = this.allAmount - this.coupon.coupon_amount;
				this.couponModal = !this.couponModal;
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
		height: 100rpx;
		background: #FFFFFF;
		left: 0;
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
