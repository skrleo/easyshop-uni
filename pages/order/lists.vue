<template>
	<view>
		<view class="cu-bar order_type_list bg-white">
			<view class="cu-item flex-sub text-center" :class="index==TabCur?'text-black text-bold cur':''" v-for="(item,index) in nav_list" :key="index" @tap="tabSelect" :data-id="item.value">
				{{item.name}}
			</view>
		</view>
		<view class="order_lists_box">
			<mescroll-body ref="mescrollRef" @init="mescrollInit" @down="downCallback" @up="upCallback">
				<view class="cu-card article margin-sm radius" v-for="(item,index) in order_lists" :key="index">
					<view class="bg-white flex justify-between padding-sm">
						<view class="text-l">订单号: {{item.order_no}}</view>
						<view class="text-l">{{item.status_name}}</view>
					</view>
					<view class="car-list sm-border" @click="detailTap(item)" v-for="(goods, index) in item.order_goods" :key="index">
						<view class="cu-item list" :key="goods.id">
							<image class="car-img" :src="goods.goods_thumb" mode="aspectFill"></image>
							<view class="car-mes centerboth">
								<view class="mes-box">
									<view class="g-name line2">
									<text class="cu-tag bg-gradual-red radius sm margin-right-xs" v-if="item.order_type === 2">
										<text>盲盒</text>
									</text>{{goods.goods_name}}</view>
									<view class="gz-box text-sm" v-if="goods.sku_name">规格：<text>{{goods.sku_name}}</text></view>
									<view class="flex justify-between align-center margin-top-sm">
										<view class="text-price">{{goods.goods_price}}</view>
										<view class="num-box centerboth">
											<view>x {{goods.number}}</view>
										</view>
									</view>
								</view>
							</view>
						</view>
					</view>
					
					<view class="flex justify-between padding-sm bg-white align-center">
						<view class="text-df line-height text-black">
							实付： <cn-money :money="item.amount" :size="38"></cn-money>
						</view>
						<view>
							<button class="cu-btn round line-black sm margin-right-sm" v-if="item.status == '1'" @click="closeTap(index, item)">取消订单</button>
							<button class="cu-btn round bg-black sm margin-right-sm" v-if="item.status == '1'" @click="payTap(item)">去付款</button>
							<button class="cu-btn round bg-black sm margin-right-sm" v-if="item.status == '2' || item.status == '3'" @click="refundTap(item)">申请退款</button>
							<button class="cu-btn round bg-black sm margin-right-sm" v-if="item.status == '4'" @click="refundTap(item)">申请售后</button>
						</view>
					</view>
				</view>
			</mescroll-body>
		</view>
	</view>
</template>

<script>
	import cnMoney from '@/components/cn-money/cn-money';
	import MescrollMixin from "@/components/mescroll-uni/mescroll-mixins.js";
	import MescrollBody from "@/components/mescroll-uni/mescroll-body.vue";
	export default {
		mixins: [MescrollMixin],
		components: {
			cnMoney,
			MescrollBody
		},
		data() {
			return {
				nav_list: [{
					name: '全部',
					value: 0,
				},{
					name: '待付款',
					value: 1
				},{
					name: '待发货',
					value: 2
				},{
					name: '待收货',
					value: 3
				},{
					name: '退款/售后',
					value: 4
				}],
				TabCur: 0,
				scrollLeft: 0,
				order_lists:[]
			}
		},
		onLoad(e) {
			if(e.TabCur !== undefined){
				this.TabCur = e.TabCur;
			}else{
				this.TabCur = 0;
			}
		},
		methods: {
			downCallback() {
				this.mescroll.resetUpScroll();
			},
			upCallback(page) {
				this.$Http.get('/order/lists?pageNum='+page.num+'&pageSize='+page.size+'&status=' + this.TabCur).then(res => {
					if(page.num == 1) this.order_lists = [];
					this.order_lists=this.order_lists.concat(res.lists);
					this.mescroll.endSuccess(res.lists.length);
				}).catch(()=>{
					//联网失败, 结束加载
					this.mescroll.endErr();
				})
			},
			tabSelect(e) {
				this.TabCur = e.currentTarget.dataset.id;
				this.scrollLeft = (e.currentTarget.dataset.id - 1) * 60;
				this.order_lists = []; // 先清空列表,显示加载进度
				this.mescroll.resetUpScroll();
			},
			detailTap(e) {
				uni.navigateTo({
					url: '/pages/order/detail?order_id=' + e.order_id
				});
			},
			goodsTap(e){
				uni.navigateTo({
					url: '/pages/goods/detail?goods_id=' + e.goods_id
				});
			},
			refundTap(e){
				uni.navigateTo({
					url: '/pages/order/refund?order_id=' + e.order_id
				});
			},
			closeTap(index, e) {
				uni.showModal({
				    title: '提示',
				    content: '确认是否取消该订单',
					success: res => {
				        if (res.confirm) {
				            var params = {
				            	order_id: e.order_id
				            }
				            this.$Http.get('/order/close', params).then(res => {
				            	if (res.statusCode !== 200) {
				            		uni.showToast({
				            			title: '订单关闭失败',
				            			icon: 'none'
				            		})
				            	}
				            	// this.order_lists.splice(index, 1);
				            	this.order_lists = []; // 先清空列表,显示加载进度
				            	this.mescroll.resetUpScroll();
				            })
				        }
				    }
				});
			},
			payTap(e) {
				var params = {
					order_id: e.order_id
				}
				this.$Http.get('/order/pay', params).then(res => {
					if (res.statusCode !== 200) {
						uni.showToast({
							title: '订单支付失败',
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

<style lang="scss">
	
	@import "../../static/style/sort_list.scss";
	
	.order_type_list {
		position: fixed;
		top: 0;
		width: 100%;
		z-index: 9999;
	}
	.order_lists_box {
		margin-top: 100upx;
	}
	.cu-item {
		margin-top: 0 !important;
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
		padding: 0 24rpx;
	}
	
	

	.all-sel-btn {
		height: 100%;
		float: left;
	}
	
	.car-list {
		width: 100%;
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
</style>
	