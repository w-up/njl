<template>
	<div id="ranking">
		<div class="ranking_content" v-show="lang_dlg">
			<img class="title" src="../assets/img/btkp.png" />
			<h2>排行榜</h2>
			<img @click="closeDialog" class="close" src="../assets/img/gb.png" />
			<!-- 我的分店 -->
			<div class="back_jou">
				<table>
					<thead>
						<tr>
							<th>排行</th>
							<th>上榜农友</th>
							<th>总资产</th>
						</tr>
					</thead>
					<tbody>
						<tr v-for="(item,index) in rankList" :key="item.ranking">
							<td>{{item.ranking}}</td>
							<td>{{item.name}}</td>
							<td>{{item.total}}</td>
						</tr>
					</tbody>
				</table>
				<div class="morera">
					<paginate 
					:page-count="pageCount" 
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
				pageindex:1,//默认为第一页
				pageCount:1,//总页数，默认为1
				rankList:[],//排行榜
			};
		},
		created: function() {

		},
		//数据保存到session
		watch: {


		},
		mounted: function() {
             var that = this;
			 that.getRanking()
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
			//分页加载
			clickCallback:function(pageNum){
				var that = this;
				console.log(pageNum)
				that.rankList = [];
				that.pageindex = pageNum;
				that.getRanking();
			},
			//经营日志
			getRanking() {
				let that = this;
				that
					.$http({
						url: "Active/ranking",
						method: "post",
						data: {
							token: sessionStorage.getItem("token"),
							p:that.pageindex
						}
					})
					.then(function(res) {
						console.log(res)
						if (res.data.code == 0) {
							that.rankList = that.rankList.concat(res.data.data.list);
							that.pageCount = Math.ceil(res.data.data.count / 4); //总页数
							console.log(that.pageCount)
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
		}
	};
</script>

<style>
	#ranking {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.6);
		overflow: hidden;
		z-index: 100;
	}

	.ranking_content {
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

	.ranking_content img.title {
		position: absolute;
		width: 3.4rem;
		top: 0.26rem;
		left: 0.36rem;
	}

	.ranking_content img.close {
		position: absolute;
		width: 0.46rem;
		right: 0.14rem;
		top: 0.1rem;
	}

	.ranking_content h2 {
		position: absolute;
		width: 1.02rem;
		top: 0.46rem;
		left: 1.8rem;
		/* font-family:'FZCY'; */
		color: #fff;
		font-size: 0.18rem;
		/* font-weight: 600; */
		text-shadow: -0.001rem 0.001rem 0.008rem #B26901, 0.004rem 0.004rem 0.001rem #B26901;
	
	}
	.back_jou {
		width: 68%;	
		margin-top: 0.8rem;
		margin-left: -0.1rem;
		background: url(../assets/img/k-2.png) no-repeat 0 0.2rem;
		background-size: 100% 56%;
		padding-top: 0.2rem;
	}
	.back_jou table {
		width: 96%;
		margin: 0 auto;
		padding-top: 0.14rem;
	}

	.back_jou table thead {
		font-size: 0.14rem;
		font-weight: normal;
		color: #EE661C;
		text-align: center;

	}

	.back_jou table tbody {
		font-size: 0.14rem;
		color: #F07923;
		/* text-shadow: -0.001rem 0.01rem 0.006rem #B26901, 0.01rem 0.001rem 0.008rem #B26901; */
		text-align: center;
	}

	.back_jou table tbody tr td {
		padding: 0.02rem 0;
	}
	.morera{
		width: 100%;
		position: absolute;
		bottom: 0.8rem;
		text-align: center;
	   
	}
	.morera ul.pagination{
		display: flex;
		width: 70% !important;
		justify-content: center;
	}
	.morera ul.pagination li{
		margin: 0 0.08rem;
	}
	.morera ul.pagination li a{
		color:#fff ;
		text-shadow: 0.004rem 0.004rem 0.001rem #d1a729;
		font-size: 0.2rem;
	}
	.morera .pagination .page-item.active{
		background: #FCCD11;
	}
	.morera .pagination .page-item.active a{
		 color: #fff;
	 }
	.morera ul.pagination li.page-item {		
		margin: 0 0.04rem;
		text-align: center;
		width: 0.2rem;
		height: 0.2rem;
		line-height: 0.2rem;
		border-radius: 50%;
		background: #fff;
		box-shadow:0.01rem 0.018rem 0.01rem #d1a729;
	}
	.morera .pagination .page-item a{
		display: block;
		font-size: 0.12rem;
		color: #f7c634;		
	}
</style>
