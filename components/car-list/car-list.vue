<template>
	<view class="car-count">
		<view class="cu-list margin-sm bg-white radius" style="overflow: hidden;border-radius: 12rpx;" v-for="(item, index) in carList" :key="index">
			<view class="cu-item flex justify-between car-list" :class="modalName=='move-box-'+ index?'move-cur':''" @touchstart="ListTouchStart" @touchmove="ListTouchMove"
			 @touchend="ListTouchEnd" :data-target="'move-box-' + index">
				 <view class="cu-item list" :key="item.id">
				 	<view class="btn centerboth text-xl" v-if="item.selected==true" @tap="setCheckedTap(index)">
				 		<text class="cuIcon-roundcheckfill text-black"></text>
				 	</view>
				 	<view class="btn centerboth text-xl" v-else @tap="setCheckedTap(index)">
				 		<text class="cuIcon-roundcheck text-black"></text>
				 	</view>
				 	<image class="car-img" :src="item.goods_thumb" mode="aspectFill"></image>
				 	<view class="car-mes centerboth">
				 		<view class="mes-box">
				 			<view class="g-name line2" @tap="checkTap(item)">{{item.goods_name}}</view>
				 			<view class="gz-box text-sm" v-if="item.sku_name">规格：<text>{{item.sku_name}}</text></view>
				 			<view class="flex justify-between align-center margin-top-sm">
				 				<view>
									<cn-money :money="item.goods_price" :size="36"></cn-money>
								</view>
				 				<view class="num-box centerboth">
				 					<text class="cuIcon-move text-black" @tap="changeNum(index, 0)"></text>
				 					<view>{{item.number}}</view>
				 					<text class="cuIcon-add text-black" @tap="changeNum(index, 1)"></text>
				 				</view>
				 			</view>
				 		</view>
				 	</view>
				</view>
				<view class="move">
					<view class="bg-red" @tap="deleteTap(index, item)">删除</view>
				</view>
			</view>
		</view>
		
		<view class="car-bottom-btn">
			<view class="centerboth" @tap="allCheckedTap">
				<view class="text-xl">
					<text v-if="allSel==true" class="cuIcon-roundcheckfill margin-right-sm text-black"></text>
					<text v-else class="cuIcon-roundcheck text-black margin-right-sm"></text>
				</view>
				全选
			</view>
			
			<view class="all-cost centerboth text-df">
				合计: <cn-money :money="allAmount" :size="42"></cn-money>
			</view>
			<view class="car-btn-box centerboth" @tap="goSettleTap()">
				<button class="cu-btn bg-black margin-right">去结算({{allNumber}})</button>
			</view>
		</view>
	</view>
</template>
<script>
	import cnMoney from '@/components/cn-money/cn-money';
	export default {
		props: {
			carList:{
				type: Array,
				default(){
					return []
				}
			}
		},
		components: {
			cnMoney,
		},
		data() {
			return {
				maskTitle:'',
				allSel:false,
				allAmount:'0.00',
				allNumber:0,
				delIds:'',//要删除的购物车id
				btnType:0,//0删除  1结算
				modalName: null, listTouchStart: 0, listTouchDirection: null,
			};
		},
		mounted() {
			this.getAllMount();
		},
		methods: {
			goSettleTap() {
				var ids = '';
				this.carList.forEach(function(item){
					if (item.selected == true) {
						ids += item.id + ',';
					}
				});
				if(ids == ''){
					uni.showToast({
						title:'请选择要结算的商品',
						icon:'none'
					})
					return false;
				}
				this.$emit('settleTap',ids.substring(0, ids.lastIndexOf(',')));
			},
			setCheckedTap: function(index) {
				var that = this;
				let carList = that.carList;
				carList[index].selected = !carList[index].selected;
				this.setAllCheckedTap();
				that.$emit('setCheckedTap',carList);
			},
			setAllCheckedTap:function(){//是否全选
				let shopNum = 0;
				for(let i=0;i<this.carList.length;i++){
					if(this.carList[i].selected == true){
						shopNum = shopNum+1;
					}
				}
				if(shopNum == this.carList.length && shopNum>0){
					this.allSel = true;
				}else{
					this.allSel = false;
				}
				this.getAllMount();
			},
			allCheckedTap: function() {//全选
				var that = this;
				this.allSel = !this.allSel;
				var allSel = this.allSel;
				this.carList.forEach(function(item){
					item.selected = allSel;
				});
				var carList = that.carList;
				that.getAllMount();
				that.$emit('allCheckedTap',carList);
			},
			getAllMount: function() { //计算选中总价
				var that = this;
				let allPrice = 0;
				var selNum = 0;
				
				this.carList.forEach(function(item){
					if (item.selected == true) {
						selNum += item.number;
						allPrice += item.number * item.goods_price;
					}
				});
				
				that.allNumber = selNum;
				that.allAmount = allPrice.toFixed(2);
			},
			getCarIds: function() { //获取要结算的商品
				var that = this;
				var carList = that.carList;
				var haveSel = [];
				for (let i = 0; i < carList.length; i++) {
					if (carList[i].selected == true) {
						haveSel.push(carList[i].id);
					}
				}
				if (haveSel.length == 0) {
					uni.showToast({
						title:'请选择要结算的商品',
						icon:'none'
					})
					return false;
				}
				let cartIds = haveSel.join(',');
				return cartIds;
			},
			checkTap: function(e) {
				this.$emit('checkTap', e);
			},
			changeNum: function(index, type) {
				var that = this;
				var carList = that.carList;
				if (type == 0) { //减少
					if (carList[index].number <= 1) {
						return false;
					}
					carList[index].number = carList[index].number - 1;
				} else { //增加
					if (carList[index].number >= carList[index].stock) {
						uni.showToast({
							title: '库存不足',
							icon: 'none'
						})
						return false;
					}
					carList[index].number = carList[index].number + 1;
				}
				that.$emit('changeNum', carList, index, type);
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
			deleteTap(index, e){
				this.$emit('deleteTap', index, e);
			}
		}
	}
</script>

<style scoped>
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
		padding-bottom: 32rpx;
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
		/* padding: 24rpx 24rpx 0 24rpx; */
	}
	
	.car-list .list {
		width: 100%;
		padding: 10rpx 15rpx 10rpx 0;
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
