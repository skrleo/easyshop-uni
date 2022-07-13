<template>
	<page>
		<view class="swiper-top" v-if="goodsInfo.goods_thumb.length > 0">
			<swiper class="screen-swiper round-dot" :indicator-dots="true" :circular="true" :autoplay="true" interval="5000"
			 duration="500">
				<swiper-item v-for="(item,index) in goodsInfo.goods_thumb" :key="index">
					<image :src="item.url" mode="aspectFill" class="swiper-image" v-if="item.type=='image'"></image>
					<video :src="item.url" autoplay loop muted :show-play-btn="false" :controls="false" objectFit="cover" v-if="item.type=='video'"></video>
				</swiper-item>
			</swiper>
		</view>
		<!--商品简介-->
		<view class="bg-white padding ui-goods-synopsis-view">
			<view class="margin-tb-sm price-view flex align-center">
				<view class="flex-sub">
					<cn-money :money="goodsInfo.goods_price" :size="43"></cn-money>
					<text class="through text-gray text-sm padding-left-xs">原价￥{{goodsInfo.line_price || 0}}</text>
				</view>
				<view class="flex-sub text-right">
					<text class="text-black text-gray text-right">销量 {{goodsInfo.goods_sales || 0}}</text>
				</view>
			</view>
			<view class="text-black">
				<text class="text-lg">{{goodsInfo.goods_name}}</text>
			</view>
			<view class="text-sm margin-top-sm ui-row-view" v-if="goodsInfo.title">
				<!-- <text class="cuIcon-refresharrow text-red icon-text" /> -->
				<text class="text-gray margin-left-xs">{{goodsInfo.title}}</text>
			</view>
			<view class="margin-top-sm">
				<view class='cu-tag radius'>货号:{{goodsInfo.goods_no}}</view>
			</view>
		</view>
		
		<view class="cu-list menu sm-border">
			<!-- 规格 -->
			<view class="cu-item arrow" v-if="sku_info.length > 0" @tap="getSkuTap()">
				<view class="text-sm">选择</view>
				<view class="picker">{{skuname}}</view>
			</view>
			<!-- 服务 -->
			<view class="cu-item arrow" v-if="goodsInfo.service_ids">
				<view class="text-sm">服务</view>
				<view class="picker padding-left-xs">{{goodsInfo.service_ids}}</view>
			</view>
			<!-- 优惠券 -->
			<view class="cu-item arrow" v-if="coupon_lists.length > 0" @tap="couponModalTap()">
				<view class="text-sm">优惠券</view>
				<view class="picker padding-left-xs">
					<view class="cu-capsule round padding-left-xs" v-if="coupon_lists.length > 0" v-for="(coupon,index) in coupon_lists">
						<view class='cu-tag sm bg-red'>券</view>
						<view class="cu-tag sm line-red">{{coupon.coupon_amount}}</view>
					</view>
				</view>
			</view>
		</view>

		<!-- <view class="cu-list bg-white menu margin-top-sm">
			<view class="cu-item arrow" v-if="goodsInfo.service_ids">
				<view class="text-sm">评价(999+)</view>
				<view class="picker padding-left-xs">查看全部</view>
			</view>
			<view style="padding: 0 30rpx 25rpx 30rpx;">
				xxx
			</view>
		</view> -->
		
		<!--商品详情-->
		<view class="margin-top bg-white ui-details-view-box padding-bottom-xl margin-bottom-xl" style="margin-bottom: 120rpx;">
			<view class="padding text-black text-df">商品详情</view>
			<view class="ui-img-box" style="width: 100%;" v-if="contents">
				<rich-text :nodes="contents"></rich-text>
				<!-- <image :src="item" v-for="(item,index) in goodsInfo.content" :key="index" mode="widthFix" /> -->
			</view>
		</view>

		<!-- 底部操作栏 -->
		<view class="bg-white ui-tabbar-view-box padding-bottom-xs">
			<view class="flex justify-between mb-sm">
				<view class="cu-bar tabbar fl">
					<button class="action" open-type="contact">
						<view class="cuIcon-service text-black">
						</view>
						客服
					</button>
					<button class="action" @click="goCarTap()">
						<view class="cuIcon-cart text-black">
							<view class="cu-tag badge" v-if="goodsInfo.carts_count > 0"> {{goodsInfo.carts_count}} </view>
						</view>
						购物车
					</button>
				</view>
				<view class="margin-sm fr">
					<button class="cu-btn line-black margin-lr-sm" @click="addCarTap()">加入购物车</button>
					<button class="cu-btn bg-black margin-left-xs" @click="buyGoods()">立即购买</button>
				</view>
			</view>
		</view>

		<!--选择规格-->
		<view class="cu-modal bottom-modal bottom-modal-box" v-if="sku_info.length > 0" :class="skuModal?'show':''">
			<view class="cu-dialog bg-white">
				<!--标题-->
				<view class="text-black justify-between margin-tb text-lg title-bar">
					<text>选择规格</text>
					<text class="cuIcon-close close-icon" @tap="hideModal"></text>
				</view>
				
				<!--内容区域-->
				<view class="modal-content">
					<!--选择规格-->
					<view class="view-box select">
						<!--商品信息-->
						<view class="cu-list">
							<view class="cu-item flex justify-start align-center">
								<view class="cu-avatar radius xxl" :style="{ 'background-image': 'url('+ skuInfos.goods_thumb +')'}"/>
								<view class="content margin-left-lg text-lg">
									<view class="text-price-view padding-tb-sm">
										<cn-money :money="skuInfos.goods_price" :size="36"></cn-money>
										<text class="text-price text-sm text-gray text-through padding-left-xs">{{skuInfos.line_price}}</text>
										<!-- <text class="cu-tag bg-gradual-red radius sm">
											<text class="cuIcon-hotfill"/>
											<text>秒杀中</text>
										</text> -->
										<text class="text-sm text-black margin-left-sm">库存：{{skuInfos.stock_num}}</text>
									</view>
									<view class="text-black text-sm flex">
										<view class="text-cut">已选: {{skuname}}</view>
									</view>
								</view>
							</view>
						</view>
						
						<!--规格数据-->
						<view class="select-btn-list-boox">
							<view class="select-item" v-for="(item,index) in sku_info" :key="index">
								<view class="text-black">{{item.spec_name}}</view>
								<view class="select-btn">
									<button class="cu-btn" :class="subIndex[index] == key?'light bg-black':''" @click="chooseSkuTap(val,index, key)" v-for="(val,key) in item.spec_value" :key="key" :disabled="stock_num == 0">{{val.spec_value}}</button>
								</view>
							</view>
						</view>
					</view>
					<view class="view-box flex justify-between text-lg text-black margin-top-lg">
						<text>购买数量</text>
						<view class="flex justify-between align-center">
							<text class="cuIcon-move text-black margin-lr-sm" @tap="changeNum(0)"></text>
							<view>{{buy_number}}</view>
							<text class="cuIcon-add text-black margin-lr-sm" @tap="changeNum(1)"></text>
						</view>
					</view>
				</view>
				
				<!--公共按钮-->
				<view class="footer-fixed padding-lg">
					<view v-if="type == 3">
						<view class="flex">
							<view class="flex-sub">
								<view class="flex flex-direction padding-sm">
									<button class="cu-btn " @click="addCarTap(1)">加入购物车</button>
								</view>
							</view>
							<view class="flex-sub">
								<view class="flex flex-direction padding-sm">
									<button class="cu-btn bg-black" @click="buyGoods(1)">立即购买</button>
								</view>
							</view>
						</view>
					</view>
					<view class="flex flex-direction" v-else>
						<button class="cu-btn bg-black lg" v-if="type == 1" @tap="cartSureTap()">确定</button>
						<button class="cu-btn bg-black lg" v-else-if="type == 2" @click="buyGoods(1)">立即购买</button>
					</view>
				</view>
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
									<button class="cu-btn bg-red round sm" @tap="getCouponTap(coupon)">立即领券</button>
								</view>
							</view>
							<view class="card-num-view bg-white flex justify-between padding-lr">
								<view class="text-xs" style="text-align: left !important;" v-if="coupon.full_amount > 0">满{{coupon.full_amount}}可用</view>
								<view class="text-xs" style="text-align: right !important;">{{coupon.effective_start_time}} - {{coupon.effective_end_time}}</view>
							</view>
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
		
		<!-- 海报分享 -->
		<qrcode-poster ref="poster" :title="goodsInfo.goods_name" @touchmove.stop.prevent="moveHandle" :subTitle="goodsInfo.goods_name"
		 :headerImg="goodsInfo.thumb_url" :price="goodsInfo.goods_price"></qrcode-poster>
	</page>
</template>

<script>
	import cnMoney from '@/components/cn-money/cn-money';
	import qrcodePoster from '@/components/poster/poster';
	import $Tool from '@/utils/tools.js'; //工具函数
	export default {
		data() {
			return {
				skuModal: false,
				cardCur: 0,
				buy_number: 1,
				towerStart: 0,
				direction: '',
				goodsInfo: {},
				tmplIds: [],
				selectArr: [],
				subIndex: [],
				skuname: '',
				skuInfos:{},
				type: 0,
				contents: '',
				sku_info:[],
				coupon_lists:[],
				is_show_model: false,
				couponModal: false,
			};
		},
		components: {
			cnMoney,
			qrcodePoster,
		},
		onLoad(options) {
			uni.showLoading({
				title: '加载中...',
				mask: true
			});
			if (options.scene !== undefined) {
				options = this.$Tool.getJson(decodeURIComponent(options.scene).split('&'));
			}
			this.getGoodsInfo(options);
		},
		onReady() {
			uni.hideLoading()
		},
		onShareAppMessage(res) {
			return {
				title: this.goodsInfo.goods_name,
				path: '/pages/goods/detail?goods_id=' + this.goodsInfo.goods_id,
				imageUrl: this.goodsInfo.thumb_url,
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
			getGoodsInfo(e) {
				var params = {
					goods_id: e.goods_id
				}
				this.$Http.get('/goods/detail', params).then(res => {
					if (res.statusCode !== 200) {
						uni.switchTab({
							url: '/pages/index/home'
						});
						return false;
					}
					
					var content = res.data.content;
					let matchres= content.match(RegExp(/<[img]+\s+(.*?)(?<id>\w*?)[\s'"](.*?)>/g));
					if(matchres){
					  matchres.forEach((item,index)=>{
					    let len = item.length;
					    let _str = item.slice(0,len-2)+' style="width:100%;text-align:center;"/>';
					    content = content.replace(item,_str)
					  })
					}
					this.contents = content;
					this.goodsInfo = res.data;
					this.tmplIds = res.data.tmplIds;
					this.coupon_lists = res.data.coupon;
					if(res.data.sku_info.length > 0){
						this.sku_info = res.data.sku_info;
						for (let i = 0; i < res.data.sku_info.length; i++) {
							this.subIndex[i] = i;
							this.chooseSkuTap(this.sku_info[i]["spec_value"][0], i, 0);
							this.sku_info[i]["spec_value"][0]['checked'] = true;
						}
					}
				})
			},
			//分享海报
			sharePoster() {
				this.checkAuth(() => {
					var params = {
						channel: this.goodsInfo.platform_type,
						sign_key: this.goodsInfo.sign_key
					}
					this.$Http.get('/goods/poster', params).then(res => {
						if (res.statusCode == 200) {
							this.$refs.poster.showCanvas(res.data.path_url);
							this.is_show_model = false
						}
					})
				})
			},
			getCoupon() {
				this.checkAuth(() => {
					
				})
			},
			getChat() {
				uni.navigateTo({
					url: "/pages/chat/index"
				});
			},
			changeNum(type) {
				if(type == 0){
					if(this.buy_number !== 0){
						this.buy_number -= 1;
					}
				} else {
					this.buy_number += 1;
				}
			},
			goCarTap() {
				uni.switchTab({
					url: "/pages/index/cart"
				});
			},
			getSkuTap() {
				this.skuModal = true;
				this.type = 3;
			},
			hideModal() {
				this.skuModal = false;
			},
			buyGoods(type = 0) {
				if(this.sku_info.length > 0 && type == 0){
					this.skuModal = true;
					this.type = 2;
				} else {
					if(this.buy_number <= 0){
						uni.showToast({
							title: '请选择购买数量',
							icon: 'none'
						})
						return false;
					}
					this.checkAuth(() => {
						var sku_ids = '';
						if(this.sku_info.length > 0){
							sku_ids = this.skuInfos.spec_sku_id;
						}
						uni.navigateTo({
							url: "/pages/settle/index?goods_id=" + this.goodsInfo.goods_id + "&sku_id=" + sku_ids+ "&number=" + this.buy_number
						});
					})
				}
			},
			cardSwiper(e) {
				this.cardCur = e.detail.current
			},
			addCarTap(type = 0) {
				if(this.sku_info.length > 0 && type == 0){
					this.skuModal = true;
					this.type = 1;
				} else {
					this.cartSureTap();
				}
			},
			cartSureTap() {
				if(this.buy_number <= 0){
					uni.showToast({
						title: '请选择购买数量',
						icon: 'none'
					})
					return false;
				}
				this.checkAuth(() => {
					this.skuModal = false;
					var sku_id = '';
					if(this.skuInfos !== undefined){
						sku_id = this.skuInfos.spec_sku_id;
					}
					var params = {
						sku_id: sku_id,
						number: this.buy_number,
						goods_id: this.goodsInfo.goods_id,
					}
					this.$Http.post('/cart/store', params).then(res => {
						if (res.statusCode == 200) {
							uni.showToast({
								title: '已加入购物车',
								icon: 'none'
							});
							this.goodsInfo.carts_count += 1;
						} else {
							uni.showToast({
								title: '加入购物车失败',
								icon: 'none'
							})
						}
					})
				})
			},
			chooseSkuTap(val, index, key) {
				if(this.selectArr[index] !== undefined){
					this.subIndex[index]
					this.$set(this.sku_info[index]["spec_value"][this.subIndex[index]],'checked', false);
				}
				this.subIndex[index] = key;
				this.selectArr[index] = val;
				var sku_ids = '';
				var skuname = '';
				var sku_info = {};
				var goods_spec = this.goodsInfo.goods_spec;
				this.$set(this.sku_info[index]["spec_value"][key],'checked', true);
				this.selectArr.forEach(function(item){
					skuname += item.spec_value + ';';
					goods_spec.forEach(function(val){
						if(val.spec_id == item.spec_id && val.spec_value_id == item.id){
							sku_ids += val.id + ',';
						}
					});
				});
				
				this.skuname = skuname.substring(0,skuname.lastIndexOf(';'));
				// 获取规格信息
				this.goodsInfo.goods_sku.forEach(function(val){
					if(val.spec_sku_id == sku_ids.substring(0,sku_ids.lastIndexOf(','))){
						sku_info = val;
					}
				});
				this.skuInfos = sku_info;
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
			couponModalTap(){
				this.couponModal = !this.couponModal;
			},
			getCouponTap(e) {
				var params = {
					coupon_id: e.id
				}
				this.$Http.get('/reap/coupon', params).then(res => {
					if (res.statusCode == 200) {
						uni.showToast({
							title: '领取成功',
							icon: 'none'
						});
						this.couponModal = !this.couponModal;
					} else {
						uni.showToast({
							title: res.message,
							icon: 'none'
						})
					}
				})
			}
		}
	}
</script>

<style lang='scss'>
	.swiper-top {
		width: 100%;
	}

	.swiper-image {
		width: 100%;
		height: auto;
	}

	.screen-swiper {
		width: 100%;
		height: 708upx;
	}

	.tower-swiper .tower-item {
		transform: scale(calc(0.5 + var(--index) / 10));
		margin-left: calc(var(--left) * 100upx - 150upx);
		z-index: var(--index);
		height: 708upx;
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

	.ui-selll-user-view-box {
		.cu-list.menu-avatar>.cu-item {
			height: 99.99rpx;

			.content {
				left: 127.27rpx;
				width: calc(100% - 94.54rpx - 127.27rpx);

				.cu-tag.sm {
					padding: 0;
					width: 27.27rpx;
					height: 27.27rpx;
					text-align: center;
					line-height: 27.27rpx;
				}
			}

			&:after {
				width: 0;
				height: 0;
				border-bottom: 0;
			}
		}

		.cu-list.menu-avatar>.cu-item>.cu-avatar {
			width: 72.72rpx;
			height: 72.72rpx;
		}

		.ui-grid-tab-view {
			position: relative;
			width: 100%;

			.grid {
				border-radius: 9.09rpx;

				.item-view {
					position: relative;
					padding: 14.54rpx 0;
				}

				.item-view+.item-view {
					&:before {
						content: "";
						position: absolute;
						background: #e2e2e2;
						height: 45.45rpx;
						top: 27.27rpx;
						width: 2rpx;
						z-index: 0;
						left: 0;
					}
				}
			}
		}

		.ui-goods-swiper-view {
			.goods-item {
				position: relative;
				text-align: center;

				.text-price-view {
					position: absolute;
					bottom: 3.63rpx;
					left: 36.36rpx;
					color: #fff;
				}
			}

			.screen-swiper {
				min-height: 145.45rpx;
				height: 172.72rpx;
			}
		}
	}

	.ui-details-view-box {
		position: relative;

		.title-view {
			padding: 9.09rpx 0 18.18rpx;
		}

		.col-item {
			margin: 9.09rpx 0;
		}

		.ui-text-content-view {
			margin: 27.27rpx 0;
		}

		.ui-img-box {
			position: relative;
			width: 100%;

			image {
				width: 100%;
				border-radius: 18.18rpx;
			}

			image+image {
				margin-top: 18.18rpx;
			}
		}
	}

	.ui-tabbar-view-box {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
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
