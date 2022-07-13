<template>
	<view class="bg-white">
		<view class="bg-white" style="height:calc(100vh);border-radius: 18rpx;">
			<mescroll-body ref="mescrollRef" @init="mescrollInit" @down="downCallback" @up="upCallback">
				<scroll-view scroll-y="true">
					<!--盲盒列表-->
					<view class="car-list radius margin-bottom-sm" style="border-radius: 12rpx;" @click="detailTap(item)" v-for="(item, index) in data_lists" :key="index">
						<view class="cu-item list padding-sm no-border" style="border: none !important" :key="item.id">
							<image class="car-img" :src="item.thumb" mode="widthFix"></image>
							<view class="car-mes centerboth">
								<view class="mes-box">
									<view class="text-df text-black text-cut">{{item.title}}</view>
									<view class="flex justify-between margin-top-sm">
										<cn-money :money="item.price" :size="38"></cn-money>
										<view class="text-df">
											已有{{item.buy_num}}人参加
										</view>
									</view>
									<view class="cu-avatar-group margin-top-xs">
										<view class="cu-avatar round df" v-for="(box,ind) in item.blind_box_rule" :key="ind" :style="[{ backgroundImage:'url(' + box.goods_thumb + ')' }]"></view>
									</view>
								</view>
							</view>
						</view>
					</view>
				</scroll-view>
			</mescroll-body>
		</view>
	</view>
</template>

<script>
	import { mapState } from 'vuex';
	import cnMoney from '@/components/cn-money/cn-money';
	import MescrollMixin from "@/components/mescroll-uni/mescroll-mixins.js";
	export default {
		mixins: [MescrollMixin],
		components: {
			cnMoney
		},
		data() {
			return {
				upOption: {
					page: {
						size: 10
					},
					noMoreSize: 5,
					empty: {
						tip: '没有更多了'
					}
				},
				data_lists: [],
				share:{}
			}
		},
		onShareAppMessage(res) {
			return {
				title: this.share.title || '更多惊喜盲盒等你来拆...',
				path: '/pages/activity/lists',
				imageUrl: this.share.thumb || '',
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
		onLoad(e) {
			this.page_init(e);
		},
		onReady() {
			uni.hideLoading()
		},
		methods: {
			page_init(e) {
				var params = {
					module: 'blind_box_lists'
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
			/*下拉刷新的回调 */
			downCallback() {
				this.mescroll.resetUpScroll()
			},
			upCallback(page) {
				this.$Http.get('/blind_box/lists?pageNum=' + page.num + '&pageSize=' + page.size).then(res => {
					if (page.num == 1) this.data_lists = [];
					this.data_lists = this.data_lists.concat(res.lists);
					this.mescroll.endSuccess(res.lists.length);
				}).catch(() => {
					//联网失败, 结束加载
					this.mescroll.endErr();
				})
			},
			detailTap(e) {
				uni.navigateTo({
					url: '/pages/activity/blind_box?id=' + e.id
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
	.bg-image{
		width: 100vw;
		height: 230rpx;
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
		width: 180rpx;
		height: 180rpx;
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
