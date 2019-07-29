<template>
	<div id="subbranch">
		<div class="sub_content" v-show="lang_dlg">
			<img class="title" src="../assets/img/btk.png" />
			<img  class="title_text" src="../assets/img/bt-fd.png" />
			<img @click="closeDialog" class="close" src="../assets/img/gb.png" />
			<!-- 我的分店 -->
			<div class="back_sub">
				<div class="sub_title">
					<div class="sub_left">
						<span>共{{subordinate_count}}家分店</span>
					</div>
					<div class="sub_right">
						<span>总资产:{{all_assets}}</span>
					</div>
				</div>
				<table>
					<thead>
						<tr>
							<th>用户名</th>
							<th>等级</th>
							<th>注册时间</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="(item,index) in subList" :key="item.id">
							<td @click="showSubInfo(item.id)">{{item.username}}</td>
							<td>{{item.level}}</td>
							<td>{{item.create_time}}</td>
						</tr>
					</tbody>
				</table>
				<div class="moreSub">
					<paginate 
					:page-count="page_count" 
					:page-range="2" 
					:margin-pages="2" 
					:click-handler="clickCallback" 
					 :prev-text="'<'"
					 :next-text="'>'" 
					 :container-class="'pagination'" 
					 :page-class="'page-item'">
					</paginate>
				</div>
			</div>
					<!-- 分店信息-->
			<div v-if="subInfo" class="subInfo_back">
				<div class="subInfo">
					<div class="su_info">
						<p><img :src="submsgList.head"></p>
						<span>{{submsgList.name}}</span>						
					</div>
					<h4>经营收益：{{submsgList.money}}</h4>
					<!-- 密码输入框 -->
					<div class="sub_pa">
						<span @click="closeSell">关闭</span>
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
				subList:[],//分店列表
				page_count:1,//总页数
				pageindex:1,//默认第一页
				subordinate_count:'',//分店数量
				all_assets:'',//总资产
				subInfo:false,//分店信息
				sub_id:'',//分店id
				submsgList:[],
			};
		},
		created: function() {

		},
		//数据保存到session
		watch: {


		},
		mounted: function() {
           let that = this;
		   that.getSubordinate();
		  
		},
		methods: {
			//关闭确认弹窗
			closeDialog: function() {
				var that = this;
				that.lang_dlg = false;
				that.$router.push({
					path: '/homepage'
				})
			},
			clickCallback:function(pageNum){
				let that = this;
				console.log(pageNum)
				that.pageindex = pageNum;
				that.subList = [];
				that.getSubordinate();
			},
			//获取我的分店
			getSubordinate() {
				let that = this;
				that
					.$http({
						url: "Personal/subordinate",
						method: "post",
						data: {
							token: sessionStorage.getItem("token"),
							p:that.pageindex
						}
					})
					.then(function(res) {
						// console.log(res)
						if (res.data.code == 0) {
							// that.recordList = res.data.data; //首页信息
							// that.count=that.productList.length
							that.subList = that.subList.concat(res.data.data.list);
							that.page_count = Math.ceil(res.data.data.count / 4); //总页数
							that.subordinate_count = res.data.data.subordinate_count;
							that.all_assets = res.data.data.all_assets;
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
			//获取分店信息
			getSubordinateMsg() {
				let that = this;
				console.log(that.sub_id)
				that
					.$http({
						url: "Personal/subordinateMsg",
						method: "post",
						data: {
							token: sessionStorage.getItem("token"),
							id:that.sub_id
						}
					})
					.then(function(res) {
						console.log(res)
						if (res.data.code == 0) {
							that.submsgList = res.data.data; //分店信息
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
			//取消出售
			closeSell: function() {
				var that = this;
				that.subInfo = false;
			},
			//支付
			showSubInfo: function(id) {
				var that = this;
				that.sub_id = id;
				console.log(id)
				that.subInfo = true;
				that.getSubordinateMsg();
			},
		}
	};
</script>

<style>
	#subbranch {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.6);
		overflow: hidden;
		z-index: 100;
	}

	.sub_content {
		position: absolute;
		top: 50%;
		left: 50%;
		margin-left: -3rem;
		margin-top: -2.1rem;
		width: 6rem;
		height: 4.2rem;
		background: url(../assets/img/k-1.png) no-repeat center center;
		background-size: 110% 100%;
		display: flex;
		justify-content: center;
	}

	.sub_content img.title {
		position: absolute;
		width: 5.2rem;
		top: 0.02rem;
		left: 0.46rem;
		z-index: 9;
	}

	.sub_content img.close {
		position: absolute;
		width: 0.46rem;
		right: 0.14rem;
		top: 0.1rem;
	}

	.sub_content img.title_text {
		position: absolute;
		width: 1.02rem;
		top: 0.36rem;
		left: 2.48rem;
		z-index: 9;
	}
	.back_sub {
		width: 68%;	
		margin-top: 0.8rem;
		margin-left: -0.1rem;
		background: url(../assets/img/k-2.png) no-repeat 0 0.16rem;
		background-size: 100% 61%;
		padding-top: 0.2rem;
	}
	.back_sub .sub_title{
		margin-top: 0.04rem;
		padding: 0 0.32rem;
		display: flex;
		justify-content: space-between;
	}
	.back_sub .sub_title .sub_left,
	.back_sub .sub_title .sub_right
	{
		width: 1.4rem;
		height: 0.28rem;
		background: url(../assets/img/jz.png) no-repeat center;
		background-size: 100% 100%;
	}
	.back_sub .sub_title .sub_right{
		background: url(../assets/img/zc.png) no-repeat center;
		background-size: 100% 100%;
	}
	.back_sub .sub_title .sub_left,
	.back_sub .sub_title .sub_right
	{
		position: relative;
	}
	.back_sub .sub_title .sub_left span,
	.back_sub .sub_title .sub_right span
	{
		position: absolute;
		left: 0.38rem;
		top: 0;
		font-size: 0.12rem;
		line-height: 0.32rem;
		-webkit-transform-origin-x: 0;
		-webkit-transform: scale(0.8);
		color: #fff;
	}
	.back_sub table {
		width: 96%;
		margin: 0 auto;
		padding-top: 0.06rem;
	}

	.back_sub table thead {
		font-size: 0.14rem;
		font-weight: normal;
		color: #EE661C;
		text-align: center;

	}
	.back_sub table thead tr th{
		height: 0.32rem;
	}
	.back_sub table tbody {
		
		font-size: 0.12rem;
		color: #F07923;
		/* text-shadow: -0.001rem 0.01rem 0.006rem #B26901, 0.01rem 0.001rem 0.008rem #B26901; */
		text-align: center;
	}

	.back_sub table tbody tr td {
		padding: 0.032rem 0;
		
	}
	.moreSub{
		width: 100%;
		position: absolute;
		bottom: 0.88rem;
		text-align: center;
	   
	}
	.moreSub ul.pagination{
		display: flex;
		width: 70% !important;
		justify-content: center;
	}
	.moreSub ul.pagination li{
		margin: 0 0.08rem;
	}
	.moreSub ul.pagination li a{
		color:#fff ;
		text-shadow: 0.004rem 0.004rem 0.01rem #d1a729;
		font-size: 0.2rem;
	}
	.moreSub .pagination .page-item.active{
		background: #FCCD11;
	}
	.moreSub .pagination .page-item.active a{
		 color: #fff;
	 }
	.moreSub ul.pagination li.page-item {		
		margin: 0 0.04rem;
		text-align: center;
		width: 0.2rem;
		height: 0.2rem;
		line-height: 0.2rem;
		border-radius: 50%;
		background: #fff;
		box-shadow:0.01rem 0.018rem 0.01rem #d1a729;
	}
	.moreSub .pagination .page-item a{
		display: block;
		font-size: 0.12rem;
		color: #f7c634;		
	}
	.subInfo_back {
		width: 100%;
		height: 100%;
		overflow: hidden;
		position: absolute;
		top: 0;
		left: 0;
		background: rgba(0, 0, 0, 0.6);
		z-index: 888;
	}
	.subInfo {
		position: absolute;
		top: 50%;
		left: 50%;
		margin-left: -1.7rem;
		margin-top: -1rem;
		width: 3.4rem;
		height: 2rem;
		background: url(../assets/img/b_sell.png) no-repeat center center;
		background-size: 100% 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
	
	}
	.sub_pa {
		width: 100%;
		text-align: center;
		margin-top: 0.18rem;
		position: absolute;
		bottom: -0.26rem;
	}
	.sub_pa span {
		display: inline-block;
		width: 1.2rem;
		height: 0.68rem;
		background: url(../assets/img/an-2.png) no-repeat center;
		background-size: 100% 100%;
		text-align: center;
		line-height: 0.46rem;
		color: #fcf5de;
		font-size: 0.16rem;
		text-shadow: -0.01rem 0.012rem 0.008rem #af3415, 0.004rem 0.004rem 0.008rem #af3415, 0.004rem 0.004rem 0.008rem #af3415;
	}
	
	.subInfo .su_info {
		font-size: 0.16rem;
		font-weight: bold;
		color: #D09049;
		display: flex;
		align-items: center;
		margin-top: 0.4rem;
		margin-bottom: 0.2rem;
	}
	.subInfo h4{
		font-size: 0.16rem;
		font-weight: normal;
		color: #EBBF40;
		text-shadow: -0.004rem 0.004rem 0.01rem #4e1a0d, 0.004rem 0.004rem 0.008rem #4e1a0d;		
	}
	.subInfo .su_info p {
		width: 0.6rem;
		height: 0.6rem;
		background: #E1BE86;
		border-radius: 50%;
		position: relative;
		margin-right: 0.04rem;
	}
	.subInfo .su_info p img{
		width: 90%;
		position: absolute;
		top: 0.04rem;		
	}
</style>
