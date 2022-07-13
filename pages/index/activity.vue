<template>
	<view>
		<view class="cu-bar search bg-white">
			<view class="search-form round">
				<text class="cuIcon-search"></text>
				<input :adjust-position="false" v-model="keyword" type="text" placeholder="搜索分类" confirm-type="search"></input>
			</view>
		</view>
		<view class="VerticalBox">
			<scroll-view class="VerticalNav nav" scroll-y scroll-with-animation :scroll-top="verticalNavTop" style="height:calc(100vh - 100rpx)">
				<view class="cu-item text-df" :class="index==tabCur?'text-green cur':'text-gray'" v-for="(item,index) in primary" :key="index" @tap="TabSelect"
				 :data-id="index">
					{{item.name}}
				</view>
			</scroll-view>
			<scroll-view class="VerticalMain" scroll-y scroll-with-animation style="height:calc(100vh - 100rpx)"
			 :scroll-into-view="'main-'+mainCur" @scroll="VerticalMain">
				<view class="padding-lr" v-for="(item,index) in minor" :key="index" :id="'main-'+index">
					<view class="text-df padding-sm text-center">
						<text class="text-black">{{item.name}}</text>
					</view>
					<view class="grid col-3 grid-square">
						<view class="text-center text-black" v-for="(val,key) in item.data" :key="key" @tap="jumpTap(val)">
							<image :src="val.thumb" mode="widthFix" class="response"></image>
							<view class="text-black">{{val.name}}</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list: [],
				minor:[],
				primary:[],
				tabCur: 0,
				mainCur: 0,
				keyword: '',
				verticalNavTop: 0,
				load: true,
			};
		},
		onLoad() {
			uni.showLoading({
				title: '加载中...',
				mask: true
			});
			let list = [{}];
			for (let i = 0; i < 26; i++) {
				list[i] = {};
				list[i].name = String.fromCharCode(65 + i);
				list[i].id = i;
			}
			this.list = list;
			this.listCur = list[0];
		},
		onReady() {
			uni.hideLoading()
		},
		onLoad(e) {
			this.base_init(e);
		},
		methods: {
			base_init(e) {
				var params = {
					keyword: this.keyword
				}
				this.$Http.get('/goods/category', params).then(res => {
					if (res.statusCode !== 200) {
						return false;
					}
					this.minor = res.data.minor;
					this.primary = res.data.primary;
				})
			},
			TabSelect(e) {
				this.tabCur = e.currentTarget.dataset.id;
				this.mainCur = e.currentTarget.dataset.id;
				this.verticalNavTop = (e.currentTarget.dataset.id - 1) * 50
			},
			jumpTap(item) {
				uni.navigateTo({
					url: "/pages/goods/lists?category_id=" + item.id + "&category_name=" + item.name
				});
			},
			VerticalMain(e) {
				// #ifdef MP-ALIPAY
				   return false  //支付宝小程序暂时不支持双向联动 
				// #endif
				let that = this;
				let tabHeight = 0;
				if (this.load) {
					for (let i = 0; i < this.list.length; i++) {
						let view = uni.createSelectorQuery().select("#main-" + this.list[i].id);
						view.fields({
							size: true
						}, data => {
							this.list[i].top = tabHeight;
							tabHeight = tabHeight + data.height;
							this.list[i].bottom = tabHeight;
						}).exec();
					}
					this.load = false
				}
				let scrollTop = e.detail.scrollTop + 10;
				for (let i = 0; i < this.list.length; i++) {
					if (scrollTop > this.list[i].top && scrollTop < this.list[i].bottom) {
						this.verticalNavTop = (this.list[i].id - 1) * 50
						this.tabCur = this.list[i].id;
						return false
					}
				}
			}
		},
	}
</script>

<style>
	.fixed {
		position: fixed;
		z-index: 99;
	}

	.VerticalNav.nav {
		width: 200upx;
		white-space: initial;
	}

	.VerticalNav.nav .cu-item {
		width: 100%;
		text-align: center;
		background-color: #f1f1f1;
		margin: 0;
		border: none;
		height: 50px;
		position: relative;
	}

	.VerticalNav.nav .cu-item.cur {
		background-color: #fff;
	}

	.VerticalNav.nav .cu-item.cur::after {
		content: "";
		width: 8upx;
		height: 30upx;
		border-radius: 10upx 0 0 10upx;
		position: absolute;
		background-color: currentColor;
		top: 0;
		right: 0upx;
		bottom: 0;
		margin: auto;
	}

	.VerticalBox {
		display: flex;
	}

	.VerticalMain {
		background-color: #fff;
		flex: 1;
	}
	.grid.col-3.grid-square>view {
		height: 210rpx !important
	}
</style>
