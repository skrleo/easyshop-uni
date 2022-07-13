<template>
	<view>
		<view class="cu-bar order_type_list bg-white">
			<view class="cu-item flex-sub text-center" :class="index==TabCur?'text-black text-bold cur':''" v-for="(item,index) in nav_list" :key="index" @tap="tabSelect" :data-id="item.value">
				{{item.name}}
			</view>
		</view>
		<view class="bg-white" style="height:calc(100vh - 50rpx)">
			<mescroll-body ref="mescrollRef" @init="mescrollInit" @down="downCallback" @up="upCallback">
				<scroll-view scroll-y="true">
					<!--商品券-->
					<view class="ui-sponsored-card-view bg-white" v-for="(coupon,index) in coupon_lists" :key="index">
						<view class="card-price-view bg-white">
							<view class="text-red price-left-view">
								<cn-money :money="coupon.coupon_amount" :size="43"></cn-money>
							</view>
							<view class="name-content-view">
								<view class="text-cut text-red">{{coupon.name}}</view>
								<view class="text-xs">{{coupon.remark}}</view>
							</view>
							<view class="btn-right-view">
								<button class="cu-btn bg-red round sm" @click="toUserTap(coupon)">去使用</button>
							</view>
						</view>
						<view class="card-num-view bg-white flex justify-between padding-lr">
							<view class="text-xs" v-if="coupon.full_amount > 0">满{{coupon.full_amount}}可用</view>
							<view class="text-xs text-right">{{coupon.effective_start_time}} - {{coupon.effective_end_time}}</view>
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
				nav_list: [{
					name: '未使用',
					value: 0,
				},{
					name: '已使用',
					value: 1
				},{
					name: '已过期',
					value: 2
				}],
				TabCur: 0,
				upOption: {
					page: {
						size: 10
					},
					noMoreSize: 5,
					empty: {
						tip: '没有更多了'
					}
				},
				coupon_lists: [],
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
			/*下拉刷新的回调 */
			downCallback() {
				this.mescroll.resetUpScroll()
			},
			upCallback(page) {
				this.$Http.get('/coupon/lists?pageNum=' + page.num + '&pageSize=' + page.size+'&status=' + this.TabCur).then(res => {
					if (page.num == 1) this.coupon_lists = [];
					this.coupon_lists = this.coupon_lists.concat(res.lists);
					this.mescroll.endSuccess(res.lists.length);
				}).catch(() => {
					//联网失败, 结束加载
					this.mescroll.endErr();
				})
			},
			toUserTap(e){
				uni.switchTab({
					url: "/pages/index/home"
				});
			},
			tabSelect(e) {
				this.TabCur = e.currentTarget.dataset.id;
				this.scrollLeft = (e.currentTarget.dataset.id - 1) * 60;
				this.coupon_lists = []; // 先清空列表,显示加载进度
				this.mescroll.resetUpScroll();
			},
		}
	}
</script>

<style lang='scss'>

	.ui-sponsored-card-view {
		position: relative;
		background-color: #FFFFFF;
		padding: 3rpx 27.27rpx 0;

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
				padding-left: 173.63rpx;
				/* padding-right: 145.45rpx; */
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
				padding-right: 12.72rpx;
			}

			text {
				position: absolute;
				right: 17.27rpx;
				top: 14.54rpx;
			}
		}
	}
</style>
