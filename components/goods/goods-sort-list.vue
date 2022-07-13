<template>
	<view class="ui-goods-list-box" v-if="list_data.length > 0">
		<view class="cu-list menu-avatar">
			<block v-for="(item,index) in list_data" :key="index">
				<view class="cu-item" @tap="listTap(item,index)">
					<view class="cu-avatar radius lg" :style="[{backgroundImage:'url('+ item.goods_thumb +')'}]"></view>
					<view class="content">
						<view class="text-black" style="height: 78upx;">
							<view class="text-cut-2 text-l text-black">
								<text>{{item.goods_name}}</text>
							</view>
						</view>
						<view class="flex justify-between margin-top-xl">
							<view>
								<cn-money :money="item.goods_price" :size="36"></cn-money>
								<text class="text-gray through text-xs padding-left-xs" v-if="item.line_price">原价￥{{item.line_price}}</text>
							</view>
							<view>
								<text class="cu-tag line-orange text-sm sm radius margin-left-sm" style="margin-top: 9upx;" v-if='item.has_coupon'>券</text>
								<text class="text-sm padding-left-xs">{{item.goods_sales}}已售</text>
							</view>
						</view>
					</view>
				</view>
			</block>
		</view>
	</view>
</template>

<script>
	import _tool from '@/utils/tools.js';
	import cnMoney from '@/components/cn-money/cn-money';
	export default {
		name: 'goods-sort-list',
		props: {
			list_data: {
				type: Array,
				default: () => {
					return []
				}
			},
			show: {
				type: Boolean,
				default: true
			}
		},
		components: {
			cnMoney,
		},
		methods: {
			listTap(data,index) {
				this.$emit('listTap', {
					data,
					index
				});
			}
		}
	}
</script>

<style lang="scss" scoped>
	.ui-goods-list-box {
		height: auto;
		.cu-list.menu-avatar > .cu-item {
			height: 218.18rpx;
			&:after {
				width: 0;
				height: 0;
				border-bottom: 0;
			}
			.cu-avatar.lg {
				width: 181.81rpx;
				height: 181.81rpx;
				font-size: 3em;
			}
			.content {
				left: 236.36rpx;
				width: calc(100% - 94.54rpx - 59.99rpx - 119.99rpx);
				line-height: 1.7em;
				.cu-tag.sm {
					display: inline-block;
					margin-left: 0;
					height: 29.09rpx;
					font-size: 18.54rpx;
					line-height: 29.09rpx;
					margin-right: 9.09rpx;
					margin-bottom: 9.09rpx;
				}
				view:first-child {
					font-size: 29.09rpx;
					display: inherit;
					align-items: inherit;
				}
				.through {
					text-decoration:line-through;
				}
				.ui-tag-view {
					text {
						margin-right: 9.09rpx;
					}
				}
			}
		}
	}
</style>
