<template>
	<view class="modal-box" :class="show?'show':''">
		<view class="dialog bg-white radius padding-bottom">
			<image class="img" :class="[show?'animation-shake':'',show?'animation-slide-bottom':'']" :style="[{animationDelay: '0.3s'}]" src="/static/images/blind_box.png" lazy-load mode="widthFix"></image>
			<view class="text-center df">
				盲盒开奖中...
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name: 'modal-img',
		props: {
			prize: {
				type: Object, // 对象
				default: function () {
					return {}
				}
			},
			show: {
				type: Boolean,
				default: false
			}
		},
		methods: {
			prizeEvent() {
				this.$emit('prizeTap');
			}
		}
	}
</script>

<style lang="scss" scoped>
	.modal-box {
	    position: fixed;
		opacity: 0;
		top: inherit;
		left: inherit;
		right: inherit;
		bottom: inherit;
		z-index: 999999;
	    text-align: center;
	    background: rgba(0, 0, 0, 0.6);
	    transition: all 0.3s;
		pointer-events: none;
		&::before {
		    content: "\200B";
		    display: inline-block;
		    height: 100%;
		    vertical-align: middle;
		}
		.dialog {
			position: relative;
			display: inline-block;
			vertical-align: middle;
			width: 418.18rpx;
			height: 418.18rpx;
			z-index: 999999;
			.img {
				width: 358rpx;
				border-radius: 3%;
			}
			.close {
			    color: #dadada;
			    top: 18.18rpx;
			    position: relative;
			    font-size: 54.54rpx;	
			}
		}
	}
	.modal-box.show {
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		opacity: 1;
		pointer-events: auto;
	}
</style>
