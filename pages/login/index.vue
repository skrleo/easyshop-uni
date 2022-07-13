<template>
	<page class="bg-white">
		<view class="login bg-white">
			<view class="flex justify-center align-center" style="margin-top: 80upx;">
				<view>
					<view class="cu-avatar xxl round" style="background-image:url(https://img.easyshop.org.cn/taoke/logo.png);"></view>
					<view class="text-center margin-top">
						<text class="text-black text-bold">电商云+</text>
					</view>
				</view>
			</view>
			<view class="margin-left-lg" style="margin-top: 120upx;">
				<view class="text-lg margin-bottom-sm">
					<text class="text-black line-height">登录后该小程序将获得以下权限</text>
				</view>
				<text class="cuIcon-title text-black"></text>获得你的公开信息（昵称，头像等）
			</view>
			<view class="padding flex flex-direction">
				<button class="cu-btn bg-black margin-tb-sm lg" @tap="loginTap">微信一键登录</button>
			</view>

			<view class="cu-modal" :class="showModal==true?'show':''" style="margin-top: 50upx;">
				<view class="cu-dialog">
					<view class="padding-sm">
						<view class="margin-top-lg margin-left-lg">
							<view class="text-xl margin-bottom-sm">
								<text class="text-black line-height">绑定手机号</text>
							</view>
							请先绑定手机号在进行此操作
						</view>
						<view class="padding flex flex-direction">
							<button class="cu-btn bg-green margin-tb-sm lg round" open-type="getPhoneNumber" @getphonenumber="getPhoneNumber"
							 withCredentials="true">微信用户一键绑定</button>
						</view>
					</view>
				</view>
			</view>

			<view class="flex align-center justify-center text-muted protocol bg-white" style="margin-top: 410rpx;">
				注册登录即代表您同意<text class="text-red" @tap="protocolTap('user')">《用户服务协议》</text> <text class="text-red" @tap="protocolTap('privacy')">《隐私权政策》</text>
			</view>
		</view>
	</page>
</template>


<script>
	import {
		mapState
	} from 'vuex'
	export default {
		data() {
			return {
				checked: false,
				showModal: false,
				sessionKey: '',
				openid: '',
				userInfo: {}
			}
		},
		computed: {
			...mapState({
				loginStatus: state => state.loginStatus,
				user: state => state.user,
			})
		},
		onLoad() {
			if (this.loginStatus) {
				uni.navigateBack({
					delta: 1,
					success: function() {
						beforePage.$vm.init(); // 执行前一个页面的刷新
					}
				});
			}
		},
		methods: {
			protocolTap: function(type) {
				uni.navigateTo({
					url: "/pages/h5/index?type=" + type + '&module=login'
				})
			},
			loginTap: function() {
				uni.getUserProfile({  
					lang: 'zh_CN',
					desc:'获取用户信息',
					success: data => {
						this.userInfo = data.userInfo;
						uni.login({
							provider: 'weixin',
							success: info => {
								 this.$Http.get('/oauth/login?code=' + info.code).then(res => {
								  	if (res.statusCode == 200) {
								  		if (res.data.is_login) {
								  			// 修改vuex的state,持久化存储
								  			this.$store.commit('login', res.data)
								  			// 提示和跳转
								  			uni.navigateBack({
								  				delta: 1
								  			});
								  			uni.showToast({
								  				title: '登录成功',
								  				icon: 'none'
								  			});
											return true;
								  		} else {
								  			this.openid = res.data.openid;
								  			this.sessionKey = res.data.session_key;
								  			this.showModal = true;
								  		}
								  	} else {
								  		//联网失败, 结束加载
								  		uni.showToast({
								  			title: "请求异常，请稍后再试！",
								  			icon: "none"
								  		});
										return true;
								  	}
								})
							}
						});
					 },
					fail:err=>{
						uni.showToast({
							title: "微信登录授权失败",
							icon: "none"
						});
					}
				});
			},
			getPhoneNumber: function(e) {
				if (e.detail.errMsg == "getPhoneNumber:ok") {
					// 获取用户信息
					var params = {
						encrypted: e.detail.encryptedData,
						session: this.sessionKey,
						iv: e.detail.iv,
						openid: this.openid,
						avatar_url: this.userInfo.avatarUrl,
						nickname: this.userInfo.nickName,
						sex: this.userInfo.gender,
						country: this.userInfo.country,
						province: this.userInfo.province,
						city: this.userInfo.city,
					}
					this.$Http.post('/oauth/register', params).then(res => {
						if (res.statusCode == 200) {
							this.showModal = false;
							// 修改vuex的state,持久化存储
							this.$store.commit('login', res.data)
							// 提示和跳转
							uni.navigateBack({
								delta: 1
							});
							uni.showToast({
								title: '登录成功',
								icon: 'none'
							});
						} else {
							uni.showToast({
								title: res.message,
								icon: "none"
							});
						}
					})
				} else {
					uni.showToast({
						title: "用户取消登录",
						icon: "none"
					});
					uni.switchTab({
						url: "/pages/index/home"
					});
				}
			}
		}
	}
</script>


<style lang="scss" scoped>
	.login {
		position: absolute;
		height: 100%;
		width: 100%;
		margin: 0 auto;

		.protocol {
			margin-top: 220upx;
			bottom: 0;
			width: 100%;
			height: 160upx;
		}
	}
</style>
