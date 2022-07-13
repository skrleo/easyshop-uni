<template>
	<view class="cu-card article no-card">
		<mescroll-body ref="mescrollRef" @init="mescrollInit" @down="downCallback" @up="upCallback">
			<scroll-view scroll-y="true">
				<view class="cu-list margin-sm bg-white" style="overflow: hidden;" v-for="(address,index) in address_lists" :key="index">
					<view class="cu-item flex justify-between" :class="modalName=='move-box-'+ index?'move-cur':''" @touchstart="ListTouchStart" @touchmove="ListTouchMove"
					 @touchend="ListTouchEnd" :data-target="'move-box-' + index">
						 <view class="flex justify-between align-center padding-sm" style="width: 100%;" @tap="checkedTap(address)">
							<view>
								<view class="padding-xs text-black">
									<text class="text-lg">{{address.contact}}</text><text class="text-l padding-left-xs">{{address.moblie}}</text>
								</view>
								<view class="padding-xs align-center">
									<button class="cu-tag bg-black radius sm margin-right-sm" v-if="address.is_default == 1">默认</button>
									<text class="through  margin-right-sm">{{address.province}}</text>
									<text class="margin-right-sm">{{address.city}}</text>
									<text class="margin-right-sm">{{address.region}}</text>
									<text class="margin-right-sm">{{address.address}}</text>
								</view>
							</view>
							<view class="line-height text-xl">
								<view class="cuIcon-edit text-black" @tap="detailTap(address)"></view>
							</view>
						</view>
						<view class="move">
							<view class="bg-red" @tap="deleteTap(index, address)">删除</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</mescroll-body>
		
		<!-- 底部操作栏 -->
		<view class="bg-white ui-tabbar-view-box">
			<view class="flex flex-direction padding-lg">
				<button class="cu-btn bg-black lg" @tap="detailTap()">添加收货地址</button>
			</view>
		</view>
	</view>
</template>

<script>
	
	import { mapState } from 'vuex';
	import MescrollMixin from "@/components/mescroll-uni/mescroll-mixins.js";
    export default {
		mixins: [MescrollMixin],
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
				cart_ids: '',
				type: 0,
				address_lists: [],
				goods_id: '',
				number: 0,
				sku_id: '',
				order_type: 'order',
				order_id: 0,
				modalName: null, listTouchStart: 0, listTouchDirection: null,
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
			}
		},
		methods: {
			/*下拉刷新的回调 */
			downCallback() {
				this.mescroll.resetUpScroll()
			},
			upCallback(page) {
				this.$Http.get('/address/lists?pageNum=' + page.num + '&pageSize=' + page.size).then(res => {
					if (page.num == 1) this.address_lists = [];
					this.address_lists = this.address_lists.concat(res.lists);
					this.mescroll.endSuccess(res.lists.length);
				}).catch(() => {
					//联网失败, 结束加载
					this.mescroll.endErr();
				})
			},
			detailTap(item = ''){
				var base_url = '';
				if(this.order_type == 'order'){
					var base_url = "/pages/address/detail?order_type="+ this.order_type +"&type="+ this.type +"&cart_ids=" + this.cart_ids + "&goods_id=" +this.goods_id + "&sku_id=" + this.sku_id+ "&number=" + this.number;
				} else {
					var base_url = "/pages/address/detail?order_type="+ this.order_type +"&type="+ this.type +"&order_id="  + this.order_id
				}
				if(item !== ''){
					base_url = base_url + "&id=" + item.id
				}
				uni.navigateTo({
					url: base_url
				});
			},
			base_init(e) {
				if(e.cart_ids !== undefined){
					this.type= 1,
					this.cart_ids = e.cart_ids;
				}
				if(e.goods_id !== undefined){
					this.type= 1,
					this.goods_id = e.goods_id;
				}
				
				if(e.sku_id !== undefined){
					this.type= 1,
					this.sku_id = e.sku_id;
				}
				
				if(e.number !== undefined){
					this.type= 1,
					this.number = e.number;
				}
				
				if(e.order_type !== undefined){
					this.type= 1,
					this.order_type = e.order_type,
					this.number = e.number;
				}
				if(e.order_id !== undefined){
					this.type= 1,
					this.order_id= e.order_id,
					this.number = e.number;
				}
			},
			checkedTap(e){
				if(this.type == 1){
					var base_url = '';
					if(this.order_type == 'order'){
						var base_url = "/pages/settle/index?address_id=" + e.id + "&cart_ids=" + this.cart_ids + "&goods_id=" +this.goods_id + "&sku_id=" + this.sku_id+ "&number=" + this.number;
					} else {
						var base_url = "/pages/activity/extract?address_id=" + e.id + "&order_id="  + this.order_id
					}
					uni.navigateTo({
						url: base_url
					});
				}
			},
			// ListTouch触摸开始
			ListTouchStart(e) {
				this.listTouchStart = e.touches[0].pageX
			},
			// ListTouch计算方向
			ListTouchMove(e) {
				this.listTouchDirection = e.touches[0].pageX - this.listTouchStart > 0 ? 'right' : 'left'
			},
			// ListTouch计算滚动
			ListTouchEnd(e) {
				if (this.listTouchDirection == 'left') {
					this.modalName = e.currentTarget.dataset.target
				} else {
					this.modalName = null
				}
				this.listTouchDirection = null
			},
			deleteTap(index, e) {
				var params = {
					id: e.id
				}
				this.$Http.get('/address/delete', params).then(res => {
					if (res.statusCode !== 200) {
						uni.showToast({
							title: res.message,
							icon: 'none'
						});
						return false;
					} else {
						uni.showToast({
							title: '删除成功',
							icon: 'none'
						});
					}
					this.address_lists.splice(index, 1);
				})
			}
		}
	}
</script>

<style lang='scss'>
	.ui-tabbar-view-box {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
	}
</style>
