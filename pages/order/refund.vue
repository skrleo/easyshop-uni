<template>
	<view>
		<scroll-view scroll-y="true" class="sv" style="height:calc(100vh - 173rpx)">
			<view class="car-list" style="border-radius: 36rpx;overflow: hidden;margin-top: 12rpx;">
				<view class="radius" style="border-radius: 12rpx;" v-for="(item, index) in order.order_goods" :key="index">
					<view class="cu-item list padding-sm no-border" style="border: none !important" :key="item.id">
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
				<view class="cu-list menu no-border text-black bg-white">
					<view class="cu-item">
						<view class="content">
							<text class="text-black">退款金额</text>
						</view>
						<view class="action">
							<cn-money :money="order.amount" :size="36"></cn-money>
						</view>
					</view>
				</view>
				<view class="cu-form-group" style="border: none !important">
					<view class="title">退款原因</view>
					<picker @change="pickerChange" :value="index" :range="picker">
						<view class="picker">
							{{index>-1?picker[index]:'请选择退款原因'}}
						</view>
					</picker>
				</view>
				<view class="cu-form-group" style="border: none !important">
					<textarea maxlength="-1" v-model="remark" placeholder="请详细说明您的退款原因"></textarea>
				</view>
				<view class="cu-form-group" style="border: none !important">
					<view class="grid col-4 grid-square flex-sub">
						<view class="bg-img" v-for="(item,index) in imgList" :key="index" @tap="viewImageTap" :data-url="imgList[index]">
						 <image :src="imgList[index]" mode="aspectFill"></image>
							<view class="cu-tag bg-red" @tap.stop="delImgTap" :data-index="index">
								<text class='cuIcon-close'></text>
							</view>
						</view>
						<view class="solids" @tap="chooseImageTap" v-if="imgList.length<4">
							<text class='cuIcon-cameraadd'></text>
						</view>
					</view>
				</view>
			</view>
		</scroll-view>
		
		<!-- 底部操作栏 -->
		<view class="bg-white ui-tabbar-view-box padding-bottom-xs">
			<view class="flex flex-direction padding-lg">
				<button class="cu-btn bg-black lg" @tap="saveTap(order)">提交申请</button>
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
				index: -1,
				order:{},
				imgList: [],
				remark:'',
				picker: ['订单信息拍错', '我不想要了', '电话/地址信息填写错误', '缺货', '其他'],
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
				this.$Http.get('/order/detail', params).then(res => {
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
					url: '/pages/order/refund'
				});
			},
			pickerChange(e) {
				this.index = e.detail.value
			},
			chooseImageTap() {
				uni.chooseImage({
					count: 4, //默认9
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album'], //从相册选择
					success: (res) => {
						// console.log('res:', res)
						// this.curTotal++;
						// this.imgList.push(res.tempFilePaths[0]);
						// const tempFilePaths = res.tempFilePaths[0];
						// // 图片上传
						// const uploadTask = uni.uploadFile({
						// 	url : 'http://22.189.25.91:9988/admin/sys-file/upload', // post请求地址
						// 	filePath: tempFilePaths,
						// 	name: 'file',  // 待确认
						// 	header: {
						// 		'Content-Type': 'multipart/form-data',
						// 		'Authorization': getApp().globalData.token || 'Basic YXBwOmFwcA=='
						// 	},
						// 	success: function (uploadFileRes) {
						// 		console.log('Success:', uploadFileRes);
						// 		this.imgsID.push(JSON.parse(uploadFileRes.data).data.fileId);
						// 		console.log('this.imgsID:', this.imgsID)
						// 	},
						// 	fail: function (uploadFileFail) {
						// 		console.log('Error:', uploadFileFail.data);
						// 	},
						// 	complete: ()=> {
						// 		console.log('Complete:');
						// 	}
						// });
						if (this.imgList.length != 0) {
							this.imgList = this.imgList.concat(res.tempFilePaths)
						} else {
							this.imgList = res.tempFilePaths
						}
					}
				});
			},
			viewImageTap(e) {
				uni.previewImage({
					urls: this.imgList,
					current: e.currentTarget.dataset.url
				});
			},
			delImgTap(e) {
				this.imgList.splice(e.currentTarget.dataset.index, 1)
			},
			saveTap(e){
				var params = {
				  order_id: e.order_id,
				  reason: this.picker[this.index],
				  remark: this.remark || '',
				  images: this.imgList,
				}
				this.$Http.post('/order/refund', params).then(res => {
					if (res.statusCode !== 200) {
						uni.showToast({
							title: res.message,
							icon: 'none'
						});
						return false;
					}
					setTimeout(() => {
						uni.showToast({
							title: '提交成功！请耐心等待客服处理',
							icon: 'none'
						});
						uni.navigateTo({
							url: '/pages/order/lists'
						});
					}, 1500);
				})
			}
		}
	}
</script>

<style scoped>
	
	.ui-tabbar-view-box {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
	}
	
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
