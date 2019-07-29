<template>
	<div id="lease">
		<div class="le_content" v-show="lang_dlg">
			<img class="title" src="../assets/img/btk.png" />
			<img v-if="center" class="title_text" src="../assets/img/bt-zx.png" />
			<img v-if="record" class="title_text" src="../assets/img/bt-jl.png" />
			<img @click="closeDialog" class="close" src="../assets/img/gb.png" />
			<!-- 租赁中心 -->
			<div v-if="center" class="back_le">
				<div class="le-record" @click="showRecord">
					<img src="../assets/img/jl.png" />
					<span>收购记录</span>
				</div>
				<van-swipe @change="onChange">
					<van-swipe-item v-for="(item,index) in productList" :key="item.id">
						<div class="le_prope">
							<div class="le_left">
								<img src="../assets/img/jz.png" />
								<span>{{item.id}}</span>
							</div>
							<div class="le_right">
								<img v-if="item.id == '一级建筑'" src="../assets/img/jz-1.png" />
								<img v-if="item.id == '二级建筑'" src="../assets/img/jz-2.png" />
								<img v-if="item.id == '三级建筑'" src="../assets/img/jz-3.png" />
								<img v-if="item.id == '四级建筑'" src="../assets/img/jz-4.png" />
								<span>{{item.price}}</span>
							</div>
						</div>
					</van-swipe-item>
					<div class="custom-indicator" slot="indicator">
						{{ current}}/{{count}}
					</div>
				</van-swipe>

				<div class="sub_le">
					<span @click="closeDialog">关闭</span>
					<span @click="showLease">收购</span>
				</div>
			</div>
			<!-- 租赁记录 -->
			<div v-if="record" class="back_record">
				<table>
					<thead>
						<tr>
							<th>等级</th>
							<th>价格</th>
							<th>时间</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="(recordItem,index) in recordList" :key="recordItem.index">
							<td>{{recordItem.level}}</td>
							<td>{{recordItem.spend}}</td>
							<td>{{recordItem.time}}</td>
						</tr>
					</tbody>
				</table>
				<div class="more">
					<paginate :page-count="page_count" :page-range="3" :margin-pages="2" :click-handler="clickCallback" :prev-text="'<'"
					 :next-text="'>'" :container-class="'pagination'" :page-class="'page-item'">
					</paginate>
				</div>
			</div>
			<!-- 租赁 -->
			<div v-if="lease" class="lease_back">
				<div class="lease">
					<h4>交易密码</h4>
					<input type="password" name="" v-model="payment_password" placeholder="请输入交易密码" />
					<div class="sub_le">
						<span @click="closeLease">关闭</span>
						<span @click="productBuy">确认</span>
					</div>

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
				current: 1, //建筑等级
				lease: false,
				productList: [], //建筑列表
				count: '', //建筑列表长度
				payment_password: '', //交易密码
				recordList: [], //记录列表
				page_count: 1, //页数
				pageindex:1 //交易记录默认第一页
			};
		},
		created: function() {

		},
		//数据保存到session
		watch: {


		},
		mounted: function() {
			let that = this;
			that.getProductList();
			that.getLeaseRecord();
		},
		methods: {
			onChange(index) {
				let that = this;
				that.current = index + 1;
			},
			//获取首页信息
			getProductList() {
				let that = this;
				that
					.$http({
						url: "Product/productList",
						method: "post",
						data: {
							token: sessionStorage.getItem("token")
						}
					})
					.then(function(res) {
						if (res.data.code == 0) {
							that.productList = res.data.data; //首页信息
							that.count = that.productList.length
							// console.log(that.count)
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
			//租赁操作
			productBuy() {
				let that = this;
				console.log(that.current)
				if (!that.payment_password || that.payment_password == null) {
					that.$toast('请输入交易密码');
				} else {
					that
						.$http({
							url: "Product/productBuy",
							method: "post",
							data: {
								token: sessionStorage.getItem("token"),
								id: that.current,
								payment_password: that.payment_password,
							}
						})
						.then(function(res) {
							console.log(res)
							if (res.data.code == 0) {
								//成功回调
								that.$toast(res.data.msg);
								that.payment_password = '';
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
			//分页加载
			clickCallback: function(pageNum) {
				let that = this;
				console.log(pageNum)
				that.recordList = [];
				that.pageindex = pageNum;
				that.getLeaseRecord();
			},
			//租赁记录
			getLeaseRecord() {
				let that = this;
				that
					.$http({
						url: "Product/productLog",
						method: "post",
						data: {
							token: sessionStorage.getItem("token"),
							p:that.pageindex
						}
					})
					.then(function(res) {
						console.log(res)
						if (res.data.code == 0) {
							// that.recordList = res.data.data; //首页信息
							// that.count=that.productList.length
							that.recordList = that.recordList.concat(res.data.data.list);
							that.page_count = Math.ceil(res.data.data.count / 4); //总页数
							console.log(that.page_count)
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
			//关闭确认弹窗
			closeDialog: function() {
				var that = this;
				that.lang_dlg = false;
				that.$router.push({
					path: '/homepage'
				})
			},
			//租赁
			showLease: function() {
				var that = this;
				// console.log(id)
				// that.sell_id = id;
				that.lease = true;
			},
			//关闭租赁
			closeLease: function() {
				var that = this;
				that.lease = false;
			},
			//租赁记录
			showRecord: function() {
				var that = this;
				that.center = false;
				console.log(that.center)
				that.record = true;

			},

		}
	};
</script>

<style>
	#lease {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.6);
		overflow: hidden;
		z-index: 100;
	}

	.custom-indicator {
		color: #F07923;
		/* text-shadow: 0.004rem 0.004rem 0.006rem #4e1a0d, 0.004rem 0.004rem 0.01rem #4e1a0d; */
	}

	.le_content {
		position: absolute;
		top: 50%;
		left: 50%;
		margin-left: -2rem;
		margin-top: -1.8rem;
		width: 4.2rem;
		height: 3.68rem;
		background: url(../assets/img/k-1.png) no-repeat center center;
		background-size: 110% 100%;
		display: flex;
		justify-content: center;
	}

	.le_content img.title {
		position: absolute;
		width: 3.4rem;
		top: 0.24rem;
		left: 0.4rem;

	}

	.le_content img.close {
		position: absolute;
		width: 0.46rem;
		right: 0.14rem;
		top: 0.1rem;
	}

	.le_content img.title_text {
		position: absolute;
		width: 1.02rem;
		top: 0.42rem;
		left: 1.6rem;
	}

	/* 租赁中心 */
	.back_le {
		width: 68%;
		margin-top: 0.8rem;
		margin-left: -0.1rem;
		background: url(../assets/img/k-2.png) no-repeat 0 0.16rem;
		background-size: 100% 48%;
		padding-top: 0.2rem;
		/* z-index: 999; */
	}

	.le-record img {
		width: 0.26rem;
		margin-left: 0.2rem;

	}

	.le-record span {
		font-size: 0.14rem;
		color: #F07923;
	}

	.le_prope {
		padding: 0 0.28rem;
		height: 0.8rem;
		display: flex;
		justify-content: space-around;
		align-items: center;
	}

	.le_prope .le_left {
		position: relative;
	}

	.le_prope .le_left img {
		width: 1rem;
	}

	.le_prope .le_left span {
		position: absolute;
		left: 0.28rem;
		top: 0.056rem;
		font-size: 0.12rem;
		color: #fff;
	}

	.le_prope .le_right {
		text-align: center;
	}

	.le_prope .le_right img {
		width: 0.6rem;
		display: block;
	}

	.le_prope .le_right span {
		font-size: 0.16rem;
		color: #F07923;
		/* text-shadow: 0.004rem 0.004rem 0.006rem #4e1a0d, 0.004rem 0.004rem 0.01rem #4e1a0d; */
	}

	.sub_le {
		width: 100%;
		text-align: center;
		margin-top: 0.1rem;

	}

	.sub_le span {
		display: inline-block;
		width: 1.2rem;
		height: 0.66rem;
		background: url(../assets/img/an-2.png) no-repeat center;
		background-size: 100% 100%;
		text-align: center;
		line-height: 0.46rem;
		color: #fcf5de;
		font-size: 0.16rem;
		/* text-shadow: 0.004rem 0.004rem 0.01rem #4e1a0d; */
	}

	/* 出售 */
	.lease_back {
		width: 100%;
		height: 100%;
		overflow: hidden;
		position: absolute;
		top: 0;
		left: 0;
		background: rgba(0, 0, 0, 0.6);
		z-index: 888;
	}

	.lease {
		position: absolute;
		top: 50%;
		left: 50%;
		margin-left: -2rem;
		margin-top: -1rem;
		width: 4rem;
		height: 2rem;
		background: url(../assets/img/b_sell.png) no-repeat center center;
		background-size: 100% 100%;
		text-align: center;

	}

	.lease h4 {
		padding-top: 0.4rem;
		font-size: 0.18rem;
		color: #d1a729;
		right: 0;

	}

	.lease input {
		width: 2.6rem;
		height: 0.5rem;
		background: url(../assets/img/sr.png) no-repeat center;
		background-size: 100% 100%;
		border: none;
		padding-left: 0.2rem;
		font-size: 0.14rem;
		color: #fff;
		margin-top: 0.2rem;
	}

	input::-webkit-input-placeholder {
		/* placeholder颜色  */
		color: #F7FBED;

	}


	/* 租赁记录 */
	.back_record {
		width: 68%;

		margin-top: 0.8rem;
		margin-left: -0.1rem;
		background: url(../assets/img/k-2.png) no-repeat 0 0.16rem;
		background-size: 100% 58%;
		padding-top: 0.2rem;
	}

	.back_record table {
		width: 96%;
		margin: 0 auto;
		padding-top: 0.06rem;
	}

	.back_record table thead {
		font-size: 0.14rem;
		font-weight: normal;
		color: #d2a00c;
		text-align: center;

	}
	.back_record table thead tr th{
		height: 0.32rem;
	}

	.back_record table tbody {
		font-size: 0.14rem;
		color: #F07923;
	/* 	text-shadow: -0.001rem 0.01rem 0.006rem #B26901, 0.01rem 0.001rem 0.008rem #B26901; */
		text-align: center;
	}

	.back_record table tbody tr td {
		padding: 0.02rem 0;
	}

	.more {
		width: 100%;
		position: absolute;
		bottom: 0.8rem;
		text-align: center;

	}

	.more .pagination {
		display: flex;
		width: 70% !important;
		justify-content: center;
	}

	.pagination li {
		margin: 0 0.08rem;
	}

	.pagination li a {
		color: #fff;
		text-shadow: 0.004rem 0.004rem 0.01rem #d1a729;
		font-size: 0.2rem;
	}
     .pagination .page-item.active{
    	background: #FCCD11;
    }
     .pagination .page-item.active a{
    	 color: #fff;
     }
	.pagination li.page-item {
		margin: 0 0.04rem;
		text-align: center;
		width: 0.2rem;
		height: 0.2rem;
		line-height: 0.2rem;
		border-radius: 50%;
		background: #fff;
		box-shadow: 0.01rem 0.018rem 0.01rem #d1a729;
	}

	.pagination .page-item a {
		display: block;
		font-size: 0.12rem;
		color: #f7c634;
	}
</style>
