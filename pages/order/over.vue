<template>
	<view style="height:calc(100vh)" class="bg-white">
		<view class="login bg-white margin-top-lg padding-top-xl" v-if="is_success">
			<view class="margin-top-sm padding-tb align-center">
				<view class="text-center margin-top-lg">
					<button class="cu-btn cuIcon lg bg-green" style="margin-left: 18rpx;">
						<text class="cuIcon-check"></text>
					</button>
				</view>
				<view class="text-center text-df text-bold margin-top-lg">
					<text class="margin-left-sm text-black">支付成功</text>
					<!-- <text class="margin-left-sm text-black" v-else>提交成功</text> -->
				</view>
			</view>
			
			<view class="padding flex justify-center margin-top-lg">
				<button style="width: 320rpx;" class="cu-btn bg-black margin-tb-sm df round" @tap="checkTap">查看订单</button>
			</view>
		</view>
		
		<view class="login bg-white margin-top-lg padding-top-xl" v-else>
			<view class="margin-top-sm padding-tb align-center">
				<view class="text-center margin-top-lg">
					<button class="cu-btn cuIcon lg bg-red" style="margin-left: 18rpx;">
						<text class="cuIcon-close"></text>
					</button>
				</view>
				<view class="text-center text-df text-bold margin-top-lg">
					<text class="margin-left-sm text-black">支付失败</text>
					<!-- <text class="margin-left-sm text-black" v-else>提交失败</text> -->
				</view>
			</view>
			
			<view class="padding flex justify-center margin-top-lg">
				<button style="width: 320rpx;" class="cu-btn bg-black margin-tb-sm df round" v-if="type === 'order'" @tap="payTap">重新支付</button>
			</view>
		</view>
		
	</view>
</template>

<script>
	export default {
		data() {
			return {
				type: '',
				order_id: 0,
				order_no: '',
				is_success: true
			}
		},
		onLoad(e) {
			this.type = e.type;
			this.order_id = e.order_id;
			this.order_no = e.order_no;
			this.is_success = e.is_success == 1 ? true : false;
		},
		methods: {
			checkTap(){
				var base_url = '';
				if(this.type == 'order'){
					base_url = '/pages/order/detail?order_id=' + this.order_id
				} else {
					base_url = '/pages/activity/detail?order_id=' + this.order_id
				}
				uni.navigateTo({
					url: base_url
				});
			},
			payTap(){
				var params = {
					order_id: this.order_id
				}
				this.$Http.get('/order/pay', params).then(res => {
					if (res.statusCode !== 200) {
						uni.showToast({
							title: '创建订单失败',
							icon: 'none'
						})
					}else{
						uni.requestPayment({
							provider: 'wxpay',
							nonceStr: res.data.nonceStr,
							package: res.data.package,
							paySign:res.data.paySign,
							signType: res.data.signType,
							timeStamp: res.data.timeStamp,
							success: info => {
								uni.showLoading({
								  title: '更新中...',
								})
								setTimeout(() => {
									uni.redirectTo({
										url: '/pages/order/over?order_id=' + res.data.order_id + '&order_no=' + res.data.order_no + '&is_success=1' 
									});
								}, 1000);
							},
							fail: err => {
								setTimeout(() => {
									uni.redirectTo({
										url: '/pages/order/over?order_id=' + res.data.order_id + '&order_no=' + res.data.order_no + '&is_success=0' 
									});
								}, 1000);
							}
						});
					}
				})
			}
		}
	}
</script>

<style lang="scss" scoped>
	.login {
		position: absolute;
		height: 100%;
		width: 100%;
		margin: 0 auto;

		.protocol {
			margin-top: 220upx;
			bottom: 0;
			width: 100%;
			height: 160upx;
		}
	}
</style>
