<template>
	<div id="order">
		<div class="order_content" v-show="lang_dlg">
			<img class="title" src="../assets/img/btk.png" />
			<img class="title_text" src="../assets/img/bt-gd.png" />
			<img @click="closeDialog" class="close" src="../assets/img/gb.png" />
			<!-- 租赁中心 -->
			<div v-if="center" class="back_order">
				<mt-navbar v-model="selected">
					<mt-tab-item @click.native="navTap(1)" id="1">买入金币</mt-tab-item>
					<mt-tab-item @click.native="navTap(2)" id="2">卖出金币</mt-tab-item>
				</mt-navbar>
				
				<!-- tab-container -->
				<mt-tab-container v-model="selected">
					<mt-tab-container-item id="1">					
						<div class="order_con">
							<div class="or_info">
								<span>剩余金币：</span>
								<span>{{assets}}</span>
							</div>
							<div class="or_info">
								<span>买入数量：</span>
								<input type="text" v-model="number" placeholder="请输入买入数量" />
							</div>
							<div class="or_info">
								<span>买入单价：</span>
								<input type="text" v-model="buy_price" placeholder="" disabled="disabled"/>
							</div>
						</div>
					</mt-tab-container-item>
					<mt-tab-container-item id="2">
					<div class="order_con">
						<div class="or_info">
							<span>剩余金币：</span>
							<span>{{assets}}</span>
						</div>
						<div class="or_info">
							<span>卖出数量：</span>
							<input type="text" v-model="number" placeholder="请输入卖出数量" />
						</div>
						<div class="or_info">
							<span>卖出单价：</span>
							<input type="text" v-model="sell_price" placeholder="" disabled="disabled" />
						</div>
					</div>
					</mt-tab-container-item>
				</mt-tab-container>
				<div class="sub_order">
					<span @click="entryorder">挂单</span>
				</div>
			</div>
		</div>
	</div>

</template>

<script>
	export default {
		components: {

		},
		data() {
			return {
				lang_dlg: true,
				center: true,
				record: false,
				id:"1",//买入、卖出切换
				selected:"1",
				type:'',//挂单类型
				price:'',//价格
				number:'',//数量
				assets:'',//剩余金币
				buy_price:'',//挂买单价
				sell_price:'',//挂卖单价
				
			};
		},
		created: function() {

		},
		//数据保存到session
		watch: {


		},
		mounted: function() {
            let that = this; 
			that.getAssets()
		},
		methods: {
			//选择相应的tab，初始化数据
			navTap(i) {	 
			  let that = this;
			  that.id = i;
			  console.log(that.id)
			},
			 //获取剩余金币
			getAssets() {
				let that = this;
				that
					.$http({
						url: "Trade/userAssets",
						method: "post",
						data: {
							token: sessionStorage.getItem("token"),
						}
					})
					.then(function(res) {
						console.log(res)
						if (res.data.code == 0) {
							//成功回调
							  that.assets = res.data.data.assets;
							  that.buy_price = res.data.data.buy_price;
							  that.sell_price = res.data.data.sell_price;
							  
							  console.log(that.assets)
						} else {
							//失败
							that.$toast(res.data.msg);
						}
					})
					.catch(function(error) {
						that.$toast({
							message: "网络连接失败",
							position: "bottom",
							duration: 5000
						});
					});
			},
			 //挂单操作
			entryorder() {
				let that = this;
				let id = that.id;
				that.type = id == 1 ? 1 : 2;
				if(that.type == 1){
					that.price = that.buy_price;
				}else{
					that.price = that.sell_price;
				}
				if(!that.number || that.number == null) {
					that.$toast('请输入数量');
				}else if(!that.price || that.price == null){
					that.$toast('请输入价格');
				}else{
					that
						.$http({
							url: "Trade/issueList",
							method: "post",
							data: {
								token: sessionStorage.getItem("token"),
								type: that.type,
								number:that.number,
								price:that.price						
							}
						})
						.then(function(res) {
							console.log(res)
							if (res.data.code == 0) {
								//成功回调
								that.$toast(res.data.msg);
								that.getAssets();
								that.number = '';
								that.price = '';
							} else {
								//失败
								that.$toast(res.data.msg);
							}
							// Indicator.close();
						})
						.catch(function(error) {
							// Indicator.close();
							that.$toast({
								message: "网络连接失败",
								position: "bottom",
								duration: 5000
							});
						});
				}				
			},
			//关闭确认弹窗
			closeDialog: function() {
				var that = this;
				that.lang_dlg = false;
				that.$router.push({
					path: '/homepage'
				})
			},
		}
	};
</script>

<style>
	#order {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.6);
		overflow: hidden;
		z-index: 100;
	}

	.order_content {
		position: absolute;
		top: 50%;
		left: 50%;
		margin-left: -2.1rem;
		margin-top: -2rem;
		width: 4.2rem;
		height: 4rem;
		background: url(../assets/img/k-1.png) no-repeat center center;
		background-size: 110% 100%;
		display: flex;
		justify-content: center;
	}
	.order_content img.title {
		position: absolute;
		width: 4rem;
		top: 0.26rem;
		left: 0.1rem;
		/* z-index: 99; */
	}
	.order_content img.close {
		position: absolute;
		width: 0.46rem;
		right: 0.1rem;
		top: 0.1rem;
		z-index: 100;
	}
	.order_content img.title_text {
		position: absolute;
		width: 0.66rem;
		top: 0.42rem;
		left: 1.8rem;
	}
	/* 租赁中心 */
	.back_order {
		width: 68%;
		margin-top: 0.8rem;
		margin-left: -0.1rem;
		background: url(../assets/img/k-2.png) no-repeat 0 0.22rem;
		background-size: 100% 60%;
		padding-top: 0.3rem;
		position: relative;
		
	}
	.back_order .mint-navbar{
		margin-left: 0.2rem;
		width: 1.6rem;
		background: none;
		height: 0.28rem;
	}
	.back_order .mint-navbar .mint-tab-item{
		width: 0.64rem;
		margin: 0 0.02rem;
		height: 100%;
		background: url(../assets/img/qh-2.png) no-repeat center;
		background-size: 96% 70%;
	}
	.back_order .mint-navbar .mint-tab-item .mint-tab-item-label{
		line-height: 0.02rem;
		
	}
	.back_order .mint-navbar .mint-tab-item .mint-tab-item-label{
		font-size: 0.1rem;
		-webkit-transform-origin-x: 0;
		-webkit-transform: scale(0.90);
		color: #F7FBED;
	}
    .back_order .mint-navbar .mint-tab-item.is-selected{
		border: none;
		background: url(../assets/img/qh-1.png) no-repeat center;
		background-size: 100% 80%;
		
	}
	.back_order .mint-navbar .mint-tab-item.is-selected .mint-tab-item-label{
		 font-size: 0.12rem;
		
	 }
	 .order_con{
		 width: 100%;
		 margin-top: 0.2rem;
	 }
	.order_con .or_info {
			padding-left: 0.2rem;
			font-size: 0.16rem;
			color: #F07923;
			/* text-shadow: -0.001rem 0.01rem 0.006rem #B26901, 0.01rem 0.001rem 0.008rem #B26901;		 */	
			display: flex;
			align-items: center;
			margin-bottom: 0.14rem;
		}
	
		.order_con .or_info span:nth-child(1) {
			width: 32%;
			display: inline-block;
			text-align: left;
	
		}
	
		.order_con .or_info input {
			width: 60%;
			background: url(../assets/img/sr.png) no-repeat center center;
			background-size: 100% 100%;
			border: none;
			font-size: 0.14rem;
			padding-left: 0.1rem;
			/* color: #fff; */
			height: 0.32rem;
		}
		input::-webkit-input-placeholder {
	    /* placeholder颜色  */
	    color: #F7FBED;
		
	}
	 
	.sub_order {
		width: 100%;
		text-align: center;
		margin-top: 0.04rem;
	}

	.sub_order span {
		display: inline-block;
		width: 1.2rem;
		height: 0.66rem;
		background: url(../assets/img/an-2.png) no-repeat center;
		background-size: 100% 100%;
		text-align: center;
		line-height: 0.46rem;
		color: #fcf5de;
		font-size: 0.16rem;
		text-shadow: 0.004rem 0.004rem 0.01rem #4e1a0d;
	}
</style>
