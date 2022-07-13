<template>
	<view class="cu-card article no-card">
		<view class="cu-form-group">
			<view class="title">联系人</view>
			<input placeholder="请输入联系人" v-model="address.contact"></input>
		</view>
		<view class="cu-form-group">
			<view class="title">手机号</view>
			<input placeholder="请输入手机号" v-model="address.moblie"></input>
			<view class="cu-capsule radius">
				<view class='cu-tag bg-black'>
					+86
				</view>
				<view class="cu-tag line-black">
					中国大陆
				</view>
			</view>
		</view>
		<view class="cu-form-group">
			<view class="title">地址选择</view>
			<picker mode="region" @change="regionChangeTap" :value="region">
				<view class="picker">
					{{address.province}}，{{address.city}}，{{address.region}}
				</view>
			</picker>
		</view>
		<view class="cu-form-group">
			<textarea maxlength="140" v-model="address.address" placeholder="详细地址"></textarea>
		</view>
		<view class="cu-form-group margin-top">
			<view class="title">是否默认</view>
			<switch @change="defaultChangeTap" :class="address.is_default?'checked':''" :checked="address.is_default?true:false"></switch>
		</view>
		
		<!-- 底部操作栏 -->
		<view class="bg-white ui-tabbar-view-box">
			<view class="flex flex-direction padding-lg">
				<button class="cu-btn bg-black lg" @tap="saveTap()">保存</button>
			</view>
		</view>
	</view>
</template>

<script>
	import { mapState } from 'vuex';
	export default {
		data() {
			return {
				address: {
					province: '广东省',
					city: '广州市',
					region: '天河区',
					is_default: false
				},
				cart_ids: '',
				type: 0,
				address_lists: [],
				goods_id: '',
				number: 0,
				sku_id: '',
				order_type: 'order',
				order_id: 0,
				region: ['广东省', '广州市', '天河区'],
			};
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
		onLoad(e) {
			this.base_init(e);
		},
		methods: {
			base_init(e) {
				if(e.cart_ids !== undefined){
					this.cart_ids = e.cart_ids;
				}
				if(e.type !== undefined){
					this.type = e.type;
				}
				if(e.goods_id !== undefined){
					this.goods_id = e.goods_id;
				}
				
				if(e.sku_id !== undefined){
					this.sku_id = e.sku_id;
				}
				
				if(e.number !== undefined){
					this.number = e.number;
				}
				
				if(e.order_type !== undefined){
					this.order_type = e.order_type,
					this.number = e.number;
				}
				if(e.order_id !== undefined){
					this.order_id= e.order_id,
					this.number = e.number;
				}
				if(e.id !== undefined){
					var params = {
						id: e.id
					}
					this.$Http.get('/address/detail', params).then(res => {
						if (res.statusCode !== 200) {
							uni.showToast({
								title: res.message,
								icon: 'none'
							});
							return false;
						}
						this.address = res.data;
						this.region = [this.address.province,this.address.city,this.address.region]
					})
				} else {
					uni.setNavigationBarTitle({
						title: '添加地址'
					})
				}
			},
			saveTap() {
				var params = {
					province: this.address.province,
					city: this.address.city,
					region: this.address.region,
					contact: this.address.contact,
					moblie: this.address.moblie,
					address: this.address.address,
					is_default: this.address.is_default,
				}
				var base_url = '';
				if(this.address.id !== undefined){
					params.id = this.address.id;
					base_url = '/address/update';
				} else {
					base_url = '/address/store';
				}
				this.$Http.post(base_url, params).then(res => {
				    if (res.statusCode !== 200) {
						uni.showToast({
							title: '更新成功！',
							icon: 'none'
						})
					}
					var base_url = '';
					if(this.order_type == 'order'){
						var base_url = "/pages/address/index?order_type="+ this.order_type +"&type="+ this.type +"&cart_ids=" + this.cart_ids + "&goods_id=" +this.goods_id + "&sku_id=" + this.sku_id+ "&number=" + this.number;
					} else {
						var base_url = "/pages/address/index?order_type="+ this.order_type +"&type="+ this.type +"&order_id="  + this.order_id
					}
					uni.navigateTo({
						url: base_url
					});
				})
			},
			defaultChangeTap() {
				this.address.is_default = !this.address.is_default;
			},
			regionChangeTap(e) {
				this.region = e.detail.value;
				this.address.province = this.region[0];
				this.address.city = this.region[1];
				this.address.region = this.region[2];
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
