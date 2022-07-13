<template>
	<view class="cn-money-box" :class="original ? 'cn-money-original' : ''" :style="{ padding: padding, margin: margin, fontSize: size + 'rpx', color: color }">
		<view v-if="rmb">
			<text class="cn-money-text" v-if="showUnit">{{ rmbUnit }}</text>
			<text class="cn-money-text">{{ rmbText }}</text>
		</view>
		<view v-else>
			<text class="cn-money-unit" v-if="showUnit && unitPosition=='left'">{{ unit }}</text>
			<text class="cn-money-text">{{ moneyData }}</text>
			<text class="cn-money-decimal" v-if="showZeroDecimal || decimalData!='00'">.{{ decimalData }}</text>
			<text class="cn-money-unit" v-if="showUnit && unitPosition=='right'">{{ unitText }}</text>
		</view>
	</view>
</template>

<script>
export default {
	name: 'cnMoney',
	props: {
		money: {
			type: Number,
			default: 0.0
		},
		//padding
		padding: {
			type: String,
			default: '0'
		},
		margin: {
			type: String,
			default: '0'
		},
		//文字大小 rpx
		size: {
			type: Number,
			default: 28
		},
		color: {
			type: String,
			default: '#e41f19'
		},
		unit: {
			type: String,
			default: '¥'
		},
		showUnit: {
			type: Boolean,
			default: true
		},
		unitPosition: {
			type: String,
			default: 'left'	// right
		},
		showZeroDecimal: {
			type: Boolean,
			default: true
		},
		thousandth: {
			type: Boolean,
			default: false
		},
		original: {
			type: Boolean,
			default: false
		},
		rmb: {
			type: Boolean,
			default: false
		}
	},
	data() {
		return {
			moneyData: '0',
			decimalData: '00',
			unitText: '元',
			rmbUnit: '人民币',
			rmbText: ''
		};
	},
	watch: {
		money(val) {
			this.splitMoney();
		},
		unit(val){
			if('$'==val){
				this.rmbUnit = '美金';
				this.unitText = '美元';
			}else if('£'==val){
				this.rmbUnit = '英镑';
				this.unitText = '镑';
			}else if('€'==val){
				this.rmbUnit = '欧元';
				this.unitText = '欧';
			}else if('₩'==val){
				this.rmbUnit = '韩元';
				this.unitText = '韩元';
			}else{
				this.rmbUnit = '人民币';
				
				this.unitText = '元';
			}
		}
	},
	created() {
		this.splitMoney();
	},
	methods: {
		splitMoney: function() {
			var m = this.money;
			if (!m) {
				m = 0.0;
			}
			var p = m.toFixed(2).toString();
			if (p.indexOf('.') > -1) {
				var pps = p.split('.');
				this.moneyData = pps[0];
				this.decimalData = pps[1];
			} else {
				this.moneyData = p;
				this.decimalData = '00';
			}
			if(this.thousandth){
				this.moneyData = parseInt(this.moneyData).toLocaleString();
			}
			if(this.rmb){
				this.toRmb();
			}
		},
		toRmb: function(){
			const fraction = ['角', '分'];
			const digit = ['零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖'];
			const unit = [
				['圆', '万', '亿'],
				['', '拾', '佰', '仟'],
			];
			let num = Math.abs(parseFloat(this.money));
			let s = '';
			fraction.forEach((item, index) => {
				s += (digit[Math.floor(num * 10 * (10 ** index)) % 10] + item).replace(/零./, '');
			});
			s = s || '整';
			num = Math.floor(num);
			for (let i = 0; i < unit[0].length && num > 0; i += 1) {
				let p = '';
				for (let j = 0; j < unit[1].length && num > 0; j += 1) {
					p = digit[num % 10] + unit[1][j] + p;
					num = Math.floor(num / 10);
				}
				s = p.replace(/(零.)*零$/, '').replace(/^$/, '零') + unit[0][i] + s;
			}

			this.rmbText = s.replace(/(零.)*零圆/, '圆').replace(/(零.)+/g, '零').replace(/^整$/, '零圆整');
		}
	}
};
</script>

<style lang="scss" scoped>
/* color start*/
.cn-money-box {
	box-sizing: border-box;
	display: inline-flex;
	align-items: flex-end;
	justify-items: flex-end;
	justify-content: flex-start;

	.cn-money-unit {
		font-size: 0.65rem;
		margin: 0;
		padding: 0;
	}

	.cn-money-text {
		margin: 0;
		padding: 0;
	}

	.cn-money-decimal {
		font-size: 0.75rem;
		margin: 0;
		padding: 0;
	}
}
.cn-money-original {
	text-decoration: line-through;
}
</style>
