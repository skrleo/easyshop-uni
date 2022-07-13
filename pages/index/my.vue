<template>
	<view>
		<view class="pure_top">
			<view class="pure_top_box">
				<view class="user_info_box padding-lr flex justify-between">
					<view class="flex justify-start" v-if="user">
						<view class="cu-avatar round lg align-center" v-bind:style="{ 'background-image': 'url('+ user.avatar_url +')'}"></view>
						<view class="content flex-sub padding-left-xs margin-top-xs">
							<view class="text-grey">{{user.nickname}}<text class="cuIcon-refresh margin-left-sm" @tap="refreshTap"></text></view>
							<view class="text-gray text-sm flex justify-between" v-if="user.mobile">
								手机号：{{ this.$Tool.formatMobile(user.mobile)}}
							</view>
						</view>
					</view>
					<!-- <view class="flex-sub padding-sm margin-xs" v-if="user.invite_code">
						<view class="cu-capsule round fr">
							<view class='cu-tag bg-orange'>
								邀请码
							</view>
							<view class="cu-tag line-orange">
								{{user.invite_code}}
							</view>
						</view>
					</view> -->
				</view>
				<view class="cu-card case user_info_box">
					<view class="cu-item shadow">
						<view class="cu-item">
							<view class="content flex-sub padding bg-orange flex justify-between" style="height: 120px;">
								<view class="text-white">{{userInfo.vip_level || '普通会员'}}</view>
								<view class="margin-tb-sm text-center" @tap="walletTap">
									<button class="cu-btn round sm shadow bg-black">查看权益</button>
								</view>
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		<view class="cu-list menu sm-border card-menu margin-top">
			<view class="cu-item arrow">
				<view class="content">
					<text class="text-black">我的订单</text>
				</view>
				<view class="action" @click="orderTap(0)">
					<text class="text-grey text-sm">查看全部订单</text>
				</view>
			</view>
			<view class="cu-list grid col-4 no-border">
				<view class="cu-item" v-for="(item,index) in orderTypes" :key="index" @click="orderTap(item.value)">
					<view :class="['cuIcon-' + item.icon,'text-' + item.color]">
						<view class="cu-tag badge" v-if="item.number!=0">
							<block>{{item.number>99?'99+':item.number}}</block>
						</view>
					</view>
					<text>{{item.label}}</text>
				</view>
			</view>
		</view>
		
		<view class="padding" v-if="banner_list.length > 0">
			<swiper class="screen-swiper round-dot" :indicator-dots="true" :circular="true" :autoplay="true" interval="5000"
			 duration="500">
				<swiper-item v-for="(item,index) in banner_list" :key="index" @click="bannerTap(item)">
					<image :src="item.url" mode="aspectFill" class="swiper-image"></image>
				</swiper-item>
			</swiper>
		</view>

		<view class="cu-list menu sm-border card-menu margin-tb">
			<view class="cu-item arrow">
				<navigator hover-class="none" class="content" url="/pages/coupon/index" open-type="redirect">
					<text class="cuIcon-ticket text-black"></text>
					<text class="text-black">我的优惠劵</text>
				</navigator>
			</view>
			<view class="cu-item arrow">
				<navigator hover-class="none" class="content" url="/pages/help/index" open-type="redirect">
					<text class="cuIcon-question text-black"></text>
					<text class="text-black">常见问题</text>
				</navigator>
			</view>
			<view class="cu-item arrow">
				<button class="cu-btn content" open-type="feedback">
					<text class="cuIcon-edit text-black"></text>
					<text class="text-black">意见反馈</text>
				</button>
			</view>
			<view class="cu-item arrow">
				<navigator hover-class="none" class="content" url="/pages/address/index" open-type="redirect">
					<text class="cuIcon-addressbook text-black"></text>
					<text class="text-black">收货地址</text>
				</navigator>
			</view>
			<!-- <view class="cu-item arrow">
				<navigator hover-class="none" class="content" url="/pages/about/index" open-type="redirect">
					<text class="cuIcon-addressbook text-black"></text>
					<text class="text-black">关于我们</text>
				</navigator>
			</view> -->
		</view>

		<view class="cu-list menu sm-border card-menu margin-top-lg">
			<view class="cu-item text-center" @click="logout">
				<view class="content">
					<text class="text-red">退出当前账号</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	import {mapState} from 'vuex'
	export default {
		data() {
			return {
				anmiaton: '',
				info:{},
				userInfo:{},
				orderTypes: [{
						label: "待付款",
						icon: "present",
						value: 1,
						number: 0
					},
					{
						label: "待发货",
						icon: "send",
						color: '7A7E83',
						value: 2,
						number: 0
					},
					{
						label: "待收货",
						icon: "deliver",
						color: '7A7E83',
						value: 3,
						number: 0
					},
					{
						label: "退款/售后",
						icon: "refund",
						color: '7A7E83',
						value: 4,
						number: 0
					}
				],
				dotStyle: false,
				towerStart: 0,
				direction: '',
				banner_list: [],
			}
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
			this.base_init();
		},
		methods: {
			base_init() {
				var params = {}
				this.$Http.get('/user/personal', params).then(res => {
					if (res.statusCode == 200) {
						this.userInfo = res.data.user;
						var orderInfo = res.data.order;
						this.orderTypes.forEach((item)=>{
							const target = orderInfo.find(i => i.type === item.value);
							item.number = target ? target.number : 0;
						});
					}
				})
			},
			jumpTap(type) {
				uni.navigateTo({
					url: "/pages/rich/index?type=" + type
				});
			},
			refreshTap() {
				uni.getUserProfile({
					lang: 'zh_CN',
					desc:'获取用户信息',
					success: info => {
						var params = {
							avatar_url: info.userInfo.avatarUrl,
							nickname: info.userInfo.nickName,
							sex: info.userInfo.gender,
							country: info.userInfo.country,
							province: info.userInfo.province,
							city: info.userInfo.city,
						}
						this.$Http.post('/oauth/sync', params).then(res => {
							if (res.statusCode == 200) {
								// 更新用户资料
								this.$store.commit('sync', params)
							} else {
								uni.showToast({
									title: "更新资料失败",
									icon: "none"
								});
							}
						})
					},
					fail:err=>{
						uni.showToast({
							title: "更新资料失败",
							icon: "none"
						});
					}
				});
			},
			walletTap() {
				uni.navigateTo({
					url: "/pages/member/index"
				});
			},
			orderTap(index = 0) {
				uni.navigateTo({
					url: "/pages/order/lists?TabCur=" + index
				});
			},
			clearTap() {
				uni.showToast({
					title: '清除缓存成功',
					icon: 'none'
				});
			},
			bannerTap() {
				var options = {};
				if (item.jump_type !== 2) {
					options = {
						url: item.jump_url
					}
				} else {
					options = {
						app_id: item.app_id,
						path: item.jump_url,
					}
				}
				if (item.is_login) {
					this.checkAuth(() => {
						this.navigateTo(options, item.jump_type);
					});
				} else {
					this.navigateTo(options, item.jump_type);
				}
			},
			// 退出登录
			logout() {
				uni.showModal({
					content: '是否要退出登录',
					success: (res) => {
						if (res.confirm) {
							this.$store.commit('logout')
							uni.navigateTo({
								url: '../login/index',
							});
							uni.showToast({
								title: '退出登录成功',
								icon: 'none'
							});
						}
					}
				});
			}
		}
	}
</script>

<style>
	.page {
		height: 100Vh;
		width: 100vw;
	}

	.page.show {
		overflow: hidden;
	}
	
	.screen-swiper{
		height: 188rpx;
		min-height: 188rpx !important;
	}

	.switch-sex::after {
		content: "\e716";
	}

	.switch-sex::before {
		content: "\e7a9";
	}

	.switch-music::after {
		content: "\e66a";
	}

	.switch-music::before {
		content: "\e6db";
	}

	.pure_top {
		width: 100%;
		height: 160px;
		position: relative;
		overflow: hidden;
	}

	.pure_top_box {
		content: '';
		width: 120%;
		height: 148px;
		position: absolute;
		left: -10%;
		top: 0;
		overflow: hidden;
		border-radius: 0 0 50% 50%;
		background-color: #2B2E3D;
	}

	.user_info_box {
		width: 80%;
		margin: 0 auto;
	}
</style>
