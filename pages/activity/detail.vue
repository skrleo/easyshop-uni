<template>
	<view>
		<view class="padding-lr padding-bottom-xs top-show text-white" style="background-color: #2B2E3D;">
			<view class="text-white padding-bottom-xs text-lg">
				{{order.status_name}}
			</view>
			<view class="text-white text-df padding-bottom-xs">
				{{order.order_status || '仓库正在努力发货，请耐心等待'}}
			</view>
		</view>
		
		<view class="bg-white desc margin-lr-sm margin-top-xl margin-bottom-sm padding-sm" style="margin-top: 133rpx;border-radius: 12rpx;">
			<view class="align-center">
				<view class="padding-bottom-xs text-black">
					<text class="text-lg">{{order.address.contact}}</text><text class="text-l padding-left-xs">{{order.address.moblie}}</text>
				</view>
				<view class="padding-xs align-center">
					<text class="through">{{order.address.province}}{{order.address.city}}{{order.address.region}}{{order.address.address}}</text>
				</view>
			</view>
		</view>
		<scroll-view scroll-y="true" class="sv" style="height:calc(100vh - 360rpx)">
			<view style="border-radius: 36rpx;overflow: hidden;">
				<view class="car-list radius" style="border-radius: 12rpx;">
					<view class="cu-item list padding-sm no-border" style="border: none !important">
						<image class="car-img" :src="order.goods_thumb" mode="aspectFill"></image>
						<view class="car-mes centerboth">
							<view class="mes-box">
								<view class="g-name line2">{{order.goods_name}}</view>
								<view class="gz-box text-sm" v-if="order.sku_name">规格：<text>{{order.sku_name}}</text></view>
								<view class="flex justify-between align-center margin-top-sm">
									<view>
										<cn-money :money="order.goods_price" :size="36"></cn-money>
									</view>
									<view class="num-box centerboth">
										<view>x {{order.number}}</view>
									</view>
								</view>
							</view>
						</view>
					</view>
					<view class="cu-list no-border text-sm text-gray bg-white">
						<!-- <view class="cu-item flex justify-between" style="padding: 0 20upx 20upx 20upx;">
							<view class="content">
								
							</view>
							<view class="action">
								实付款：<cn-money :money="item.goods_price * item.number" :size="36"></cn-money>
							</view>
						</view> -->
					</view>
				</view>
			</view>
			<view class="text-black car-list text-df sm-border" style="margin-bottom: 130rpx;">
				<view class="flex justify-between padding-sm bg-white">
					<view>订单编号</view>
					<view>{{order.order_no}}</view>
				</view>
				<view class="flex justify-between padding-sm bg-white">
					<view>商品数量</view>
					<view>{{order.number}}</view>
				</view>
				<view class="flex justify-between padding-sm bg-white">
					<view>支付时间</view>
					<view>{{order.payment_time || '无'}}</view>
				</view>
				<view class="flex justify-between padding-sm bg-white">
					<view>邮费</view>
					<view>{{order.post_fee}}</view>
				</view>
				<view class="flex justify-between padding-sm bg-white">
					<view>备注</view>
					<view><input class="padding-left-xs" v-model="order.remark" placeholder="选填,60字以内"></input></view>
				</view>
			</view>
		</scroll-view>
	
		<!-- 底部操作栏 -->
		<view class="flex car-bottom-btn padding-tb justify-between padding-bottom-xs">
			<view class="all-cost centerboth text-df">
				合计: <cn-money :money="order.amount" :size="42"></cn-money>
			</view>
			<view class="align-center" style="margin-top: 16rpx;">
				<button class="cu-btn bg-black margin-right" @click="refundTap(order)" v-if="order.status > 3">申请售后</button>
			</view>
		</view>
	</view>
</template>

<script>
	import cnMoney from '@/components/cn-money/cn-money';
	import { mapState } from 'vuex';
	export default {
		data() {
			return {
				order:{},
			}
		},
		components: {
			cnMoney
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
				  this.order = res.data;
				})
			},
			refundTap(e){
				uni.navigateTo({
					url: '/pages/blind_box/refund?order_id=' + e.order_id
				});
			},
			goodsTap(e){
				uni.navigateTo({
					url: '/pages/goods/detail?goods_id=' + e.goods_id
				});
			}
		}
	}
</script>

<style scoped>
	.top-show {
		position: fixed;
		top: 0;
		width: 100%;
		z-index: 9999;
	}
	.centerboth {
	    display:flex;
	    display: -webkit-flex;
	    align-items:center;
	    -webkit-align-items:center;
	    justify-content: center;
	    -webkit-justify-content: center;
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
		padding: 0 20rpx;
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
		padding: 0 10rpx 0 40rpx;
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
</style>
