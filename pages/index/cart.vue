<template>
    <view>
		<mescroll-body ref="mescrollRef" @init="mescrollInit" @down="downCallback" @up="upCallback">
			<scroll-view scroll-y="true">
				<car-list ref="mycar" :carList="cart_lists" v-if="cart_lists.length > 0" @checkTap="checkTap" @deleteTap="deleteTap"  @settleTap="settleTap" @setCheckedTap="setCheckedTap" @allCheckedTap="allCheckedTap" @changeNum="changeNum"></car-list>
			</scroll-view>
		</mescroll-body>
    </view>
</template>

  <script>
	import { mapState } from 'vuex';
	import carList from '@/components/car-list/car-list';
	import MescrollMixin from "@/components/mescroll-uni/mescroll-mixins.js";
    export default {
		mixins: [MescrollMixin],
        components:{
            carList
        },
        data() {
            return {
                maskTitle:'',
				upOption: {
					page: {
						size: 10
					},
					noMoreSize: 5,
					empty: {
						tip: '没有更多了'
					}
				},
				cart_lists: [],
                carList:[]
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
				// uni.navigateTo({
				// 	url: '/pages/login/index'
				// });
				// return false;
			}
			this.cart_lists = []; // 先清空列表,显示加载进度
			this.mescroll.resetUpScroll();
		},
        methods: {
			/*下拉刷新的回调 */
			downCallback() {
				this.mescroll.resetUpScroll()
			},
			upCallback(page) {
				this.$Http.get('/cart/lists?pageNum=' + page.num + '&pageSize=' + page.size).then(res => {
					if(res.statusCode === 200){
						if (page.num == 1) this.cart_lists = [];
						this.cart_lists = this.cart_lists.concat(res.lists);
						this.mescroll.endSuccess(res.lists.length);
					} else {
						this.mescroll.endSuccess(0);
					}
				}).catch(() => {
					//联网失败, 结束加载
					this.mescroll.endErr();
				})
			},
            settleTap:function(ids) {
				uni.navigateTo({
					url: "/pages/settle/index?cart_ids=" + ids
				});
			},
			checkTap: function(e) {
				uni.navigateTo({
					url: "/pages/goods/detail?goods_id=" + e.goods_id
				});
			},
			setCheckedTap:function(carList) {
				var that = this;
				that.cart_lists = carList;
			},
			allCheckedTap:function(carList) {
				var that = this;
			    that.cart_lists = carList;
			},
            changeNum:function(carList, index, type){
				var that = this;
			    var params = {
					id: carList[index].id,
					number: carList[index].number,
			    }
			    this.$Http.post('/cart/update', params).then(res => {
				    if (res.statusCode !== 200) {
						uni.showToast({
							title: '更新数量错误！',
							icon: 'none'
						})
					}
			    });
                that.cart_lists[index].number = carList[index].number;
                this.$refs.mycar.getAllMount();//计算价格展示
            },
			deleteTap(index, e) {
				var params = {
					id: e.id
				}
				this.$Http.get('/cart/delete', params).then(res => {
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
					this.cart_lists.splice(index, 1);
				})
			}
        }
    }
  </script>
  
  <style scoped>
      .container {
          height: 100%;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: space-between;
          box-sizing: border-box;
          overflow: hidden;
      }
      .line1{
          overflow: hidden;
          text-overflow: ellipsis;
          display: box;
          display: -webkit-box;
          line-clamp: 1;
          box-orient: vertical;
          -webkit-line-clamp: 1;
          -webkit-box-orient: vertical;
          word-break: break-all; /* 英文换行问题 */
      }
      .line2 {
          overflow: hidden;
          text-overflow: ellipsis;
          display: box;
          display: -webkit-box;
          line-clamp: 2;
          box-orient: vertical;
          -webkit-line-clamp: 2;
          -webkit-box-orient: vertical;
          word-break: break-all; /* 英文换行问题 */
      }
      .centerboth {
          display:flex;
          display: -webkit-flex;
          align-items:center;
          -webkit-align-items:center;
          justify-content: center;
          -webkit-justify-content: center;
      }
      .clearfix:after {
          content: "";
          display: block;
          visibility: hidden;
          height: 0;
          clear: both;
      }
  
      .clearfix {
          zoom: 1;
      }
      image{
        padding: 0;
        margin: 0;
      }
      textarea {
        width:300rpx;
        height:75rpx;
        display:block;
        position:relative;
      }
      button::after{ border: none; }
      car-list{
          width: 100%;
      }
  </style>