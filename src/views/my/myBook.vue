<style lang="scss">
	@import '~@/assets/scss/index.scss';
	.myBook{
		@include pageHaveHeight4Scroll();
		.myBookTabs {
			color: $mainTextColor;
			.list{
				.item{
					margin-top: 1px;
					display: flex;
					flex-direction: row;
					align-content: center;
					align-items: center;
					padding: $boxPadding2;
					background-color: $main-box-color;
					border-top: 1px solid $mainBorderColor;
					.flex{
						flex: 1;
						
					}
					.flexRight{
						flex: 0 0 60px;
						text-align: right;
						font-size: $fs-16;
					}
					.operatorBox{
						flex: 0 0 100px;
						text-align: center;
						font-size: $fs-12;
						.van-count-down{
							color: $mainTextColor !important;
						}
						// .countDownTime{
						// 	color: $mainTextColor !important;
						// }
						.cancelBtn,.showDetailBtn{
							text-decoration: underline;
						}
					}
				}
			}
		}
		.textAdornColor{
			color: $main-adorn-color !important;
		}
	}
	
</style>
<template>
	<div class="myBook">
		<m-header>
			<i class="leftBox iconfont iconfont-left-arrow" @click="back"></i>
			<div class="text">
				我的账本
			</div>
			<i class="rightBox icon"></i>
		</m-header>
		<div class="myBookTabs">
			<!-- <van-search
			  v-model="value"
			  placeholder="请输入搜索关键词"
			  show-action
			  shape="round"
			  @search="onSearch"
			>
			  <div slot="action" @click="onSearch">搜索</div>
			</van-search> -->
			<van-pull-refresh v-model="loading" @refresh="refreshEvent">
				<van-tabs v-model="activeName" :background="$api.tabBgColor" :color="$api.tabActiveColor" :title-active-color="$api.tabActiveColor"
			 :title-inactive-color="$api.tabTextColor" :border="false" @change="tabChange" animated sticky>
					<van-tab title="团队算力" name="calculation">
						<van-list v-model="loading2" :finished="finished2" finished-text="没有更多了" @load="onLoad2">
						<div class="list">
							<div class="item" v-for="item in list2" :key="item.id">
								<div class="flex">
									<div class="line">{{item.createTime}}</div>
									<div class="line margT6">{{item.type | calculationType}}后拥有算力 {{item.currentCalculationNum}}</div>
								</div>
								<div class="flexRight">{{item.addOrReduce}} {{item.calculation}}</div>
							</div>
						</div>
						</van-list>
					</van-tab>
					<van-tab title="帮扶券" name="ticket">
						<van-list v-model="loading4" :finished="finished4" finished-text="没有更多了" @load="onLoad4">
						<div class="list">
							<div class="item" v-for="item in list4" :key="item.id">
								<div class="flex">
									<div class="line">{{item.createTime}}</div>
									<!-- <div class="line margT6">从<i class="mainAdornColor">{{item.fromUserName}}</i>到<i class="mainAdornColor">{{item.toUserName}}</i></div> -->
									<div class="line margT6"><i :class="item.type==12?'mainAdornColor':''">{{item.type | platformBookType}}</i><i class="mainAdornColor" v-if="item.type==7">给{{item.toUserName}}</i>后拥有券 {{item.currentTicketNum}}</div>
								</div>
								<div class="flexRight">{{item.addOrReduce}} {{item.platformTicket}}</div>
							</div>
						</div>
						</van-list>
					</van-tab>
					<van-tab title="贡献值" name="contribution">
						<van-list v-model="loading3" :finished="finished3" finished-text="没有更多了" @load="onLoad3">
							<div class="list">
								<div class="item" v-for="item in list3" :key="item.id">
									<div class="flex">
										<div class="line">{{item.createTime}}</div>
										<div class="line margT6" :class="item.type==20?'red':item.type==15?'red2':item.type==22?'green':''">{{item.type | contributeType}}后拥有贡献值 {{item.currentContributionValue}}</div>
									</div>
									<div class="flexRight">{{item.addOrReduce}} {{item.contributionValue}}</div>
								</div>
							</div>
						</van-list>
					</van-tab>
					<van-tab title="矿石" name="mineral">
						<van-list v-model="loading1" :finished="finished1" finished-text="没有更多了" @load="onLoad1">
						<div class="list">
							<div class="item" v-for="item in list1" :key="item.id">
								<div class="flex">
									<div class="line">{{item.createTime}}</div>
									<div class="line margT6" v-if="item.type==2||item.type==3">从<i class="mainAdornColor">{{item.fromUserName}}</i>到<i class="mainAdornColor">{{item.toUserName}}</i></div>
									<div class="line margT6">{{item.type | mineralBookType}}后拥有矿石数 {{item.currentMineralNum}}</div>
								</div>
								<div class="flexRight">{{item.addOrReduce}} {{item.number}}</div>
							</div>
						</div>
						</van-list>
					</van-tab>
					<van-tab title="爱心值" name="loveValue">
						<van-list v-model="loading5" :finished="finished5" finished-text="没有更多了" @load="onLoad5">
						<div class="list">
							<div class="item" v-for="item in list5" :key="item.id">
								<div class="flex">
									<div class="line">{{item.createTime}}</div>
									<div class="line margT6"><i :class="item.type==0?'red':item.type==1?'green':item.type==2?'blue':item.type==3?'yellow':''">{{item.type | loveValueBookType}}</i>后拥有爱心值 {{item.currentNum}}</div>
								</div>
								<div class="flexRight">{{item.addOrReduce}} {{item.num}}</div>
							</div>
						</div>
						</van-list>
					</van-tab>
				</van-tabs>
			</van-pull-refresh>
		</div>
		<!-- </van-pull-refresh> -->
	</div>
</template>

<script>
	import mHeader from '@/components/Header.vue';
	// import mFullscreen from '@/components/Fullscreen.vue';
	export default {
		data() {
			return {
				userId:"",
				isShowSkeleton:true,
				loading: true,
				currentPage1: 1,
				currentPage2: 1,
				currentPage3: 1,
				currentPage4: 1,
				currentPage5: 1,
				pageSize:20,
				activeName:'mineral',
				loading1:false,
				finished1:false,
				loading2:false,
				finished2:false,
				loading3:false,
				finished3:false,
				loading4:false,
				finished4:false,
				loading5:false,
				finished5:false,
				list1:[],
				list2:[],
				list3:[],
				list4:[],
				list5:[],
				list:[{
					id:0,
					createTime:'2019-12-12 12:12:12',
					type:-1,
					pm:'+',
					contributionValue:'20'
				}],
				mineralList:[{
					id:0,
					createTime:'2019-12-12 12:12:12',
					from:'矿机产出',
					addOrReduce:'+',
					number:'20'
				}]
			}
		},
		components: {
			mHeader
		},
		create() {
			
		},
		mounted() {
			let _this = this;
			_this.$cookies.set("isRefreshUserInfo",1,_this.$api.cookiesTime);
			if (_this.$cookies.isKey("tab_name_book")) {
				_this.activeName = _this.$cookies.get("tab_name_book");
			}else{
				_this.activeName = "mineral";
			}
			_this.userId = _this.$cookies.get('userId');
			if(_this.$utils.isNUll(_this.userId)){
				_this.$toast(_this.$api.loginAgainTipText);
				_this.$router.replace('login');
				return;
			}
		},
		methods: {
			back(){
				// this.$router.replace('my');
				this.$router.go(-1);
			},
			refreshEvent() {
				// console.log("refresh1")
				let _this = this;
				if(_this.activeName == 'mineral'){
					console.log("refresh1");
					_this.currentPage1 = 1;
					_this.list1 = [];
					_this.finished1 = false;
					_this.onLoad1();
				}else if(_this.activeName == 'calculation'){
					console.log("refresh2");
					_this.currentPage2 = 1;
					_this.list2 = [];
					_this.finished2 = false;
					_this.onLoad2();
				}else if(_this.activeName == 'contribution'){
					_this.currentPage3 = 1;
					_this.list3 = [];
					_this.finished3 = false;
					_this.onLoad3();
				}else if(_this.activeName == 'ticket'){
					_this.currentPage4 = 1;
					_this.list4 = [];
					_this.finished4 = false;
					_this.onLoad4();
				}else if(_this.activeName == 'loveValue'){
					_this.currentPage5 = 1;
					_this.list5 = [];
					_this.finished5 = false;
					_this.onLoad5();
				}
			},
			tabChange(name, title) {
				let _this = this;
				// console.log('name', name)
				// _this.activeName = name;
				_this.$cookies.set("tab_name_book", name, _this.$api.cookiesTime)
			},
			onLoad1(){
				//console.log('load1')
				let _this = this;
				let params = {
					pageNo: _this.currentPage1,
					pageSize: _this.pageSize,
					userId: _this.userId,
					/* day:7 */
				}
				// console.log('params',params);
				_this.loading1 = true;
				_this.$ajax.ajax(_this.$api.getMineralBookList, 'GET', params, function(res) {
					_this.loading = false;
					if (res.code == _this.$api.CODE_OK) {
						let list = res.data.list;
						_this.list1.push(...list);
						_this.loading1 = false;
						if(res.data.endRow == res.data.total){
							_this.finished1 = true;
						}else{
							_this.currentPage1 = _this.currentPage1 + 1;
							//console.log('_this.currentPage1',_this.currentPage1);
						}
					}else{
						_this.list1 = _this.list;
						_this.loading1 = false;
						_this.finished1 = true;
						_this.$toast(res.message);
					}
				},function(){
					//_this.loading = false;
					_this.loading1 = false;
					//_this.finished1 = true;
				})
				/* ,function(){
					//_this.loading = false;
					_this.loading1 = false;
					_this.finished1 = true;
				} */
				// 异步更新数据
				// setTimeout(() => {
				// 	// 加载状态结束
				// 	_this.loading1 = false;
				// 	_this.finished1 = true;
				// }, 500);
			},
			onLoad2(){
				console.log('load2')
				let _this = this;
				// 异步更新数据
				let params = {
					pageNo: _this.currentPage2,
					pageSize: _this.pageSize,
					userId: _this.userId
				}
				_this.loading2 = true;
				_this.$ajax.ajax(_this.$api.getCalculationPowerPageList, 'GET', params, function(res) {
					// // console.log('res', res);
					_this.loading = false;
					if (res.code == _this.$api.CODE_OK) {
						// // console.log('_this.list1',_this.list1);
						// // console.log('res.data.list',res.data.list)
						let list = res.data.list;
						_this.list2.push(...list);
						_this.loading2 = false;
						// console.log('res.data.endRow '+res.data.endRow+' res.data.total '+res.data.total)
						if(res.data.endRow == res.data.total){
							_this.finished2 = true;
							// console.log('res.data.endRow == res.data.total');
						}else{
							_this.currentPage2 = _this.currentPage2 + 1;
						}
					}else{
						_this.list2 = _this.list;
						_this.loading2 = false;
						_this.finished2 = true;
						_this.$toast(res.message);
					}
				},function(){
					_this.loading = false;
					_this.loading2 = false;
					//_this.finished2 = true;
				})
				// setTimeout(() => {
				// 	// 加载状态结束
				// 	_this.loading2 = false;
				// 	_this.finished2 = true;
				// }, 500);
			},
			onLoad3(){
				// console.log('load3',"getAssistContributionValueListByUserId")
				let _this = this;
				let params = {
					pageNo: _this.currentPage3,
					pageSize: _this.pageSize,
					userId: _this.userId
				}
				_this.loading3 = true;
				_this.$ajax.ajax(_this.$api.getAssistContributionValueList, 'GET', params, function(res) {
					_this.loading = false;
					if (res.code == _this.$api.CODE_OK) {
						let list = res.data.list;
						_this.list3.push(...list);
						_this.loading3 = false;
						if(res.data.endRow == res.data.total){
							_this.finished3 = true;
						}else{
							_this.currentPage3 = _this.currentPage3 + 1;
						}
					}else{
						_this.list3 = _this.list;
						_this.loading3 = false;
						_this.finished3 = true;
						_this.$toast(res.message);
					}
				},function(){
					_this.loading = false;
					_this.loading3 = false;
					//_this.finished3 = true;
				})
				// 异步更新数据
				// setTimeout(() => {
				// 	// 加载状态结束
				// 	_this.loading3 = false;
				// 	_this.finished3 = true;
				// }, 500);
				
			},
			onLoad4(){
				let _this = this;
				let params = {
					pageNo: _this.currentPage4,
					pageSize: _this.pageSize,
					userId: _this.userId
				}
				_this.loading4 = true;
				_this.$ajax.ajax(_this.$api.getAssistPlateformTicketAccountBookList, 'GET', params, function(res) {
					_this.loading = false;
					if (res.code == _this.$api.CODE_OK) {
						let list = res.data.list;
						_this.list4.push(...list);
						_this.loading4 = false;
						if(res.data.endRow == res.data.total){
							_this.finished4 = true;
						}else{
							_this.currentPage4 = _this.currentPage4 + 1;
						}
					}else{
						_this.list4 = _this.list;
						_this.loading4 = false;
						_this.finished4 = true;
						_this.$toast(res.message);
					}
				},function(){
					_this.loading = false;
					_this.loading4 = false;
				})
			},
			onLoad5(){
				let _this = this;
				let params = {
					pageNo: _this.currentPage5,
					pageSize: _this.pageSize,
					userId: _this.userId
				}
				_this.loading5 = true;
				_this.$ajax.ajax(_this.$api.getAssistLoveValueList, 'GET', params, function(res) {
					_this.loading = false;
					if (res.code == _this.$api.CODE_OK) {
						let list = res.data.list;
						_this.list5.push(...list);
						_this.loading5 = false;
						if(res.data.endRow == res.data.total){
							_this.finished5 = true;
						}else{
							_this.currentPage5 = _this.currentPage5 + 1;
						}
					}else{
						_this.list5 = _this.list;
						_this.loading5 = false;
						_this.finished5 = true;
						_this.$toast(res.message);
					}
				},function(){
					_this.loading = false;
					_this.loading5 = false;
				})
			},
		}
	}
</script>


