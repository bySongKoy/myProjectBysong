<template>
	<view class="container">
		<view class="tabs">
			<u-tabs-swiper ref="tabs" :list="list" :current="current" @change="change" :is-scroll="false" swiperWidth="750">
			</u-tabs-swiper>
		</view>
		<view class="container-tabs__list">
			<swiper class="container-tabs__swiper" :current="current" @change="changeSwiper">
				<swiper-item class="swiper-item">
					<scroll-view scroll-y style="height: 100%;">
						<!-- 未缴费订单列表 -->
						<view class="queryTitle">
						<view class="queryDate">
							<view class="label">
								<view class="Occupied startTime">开始时间</view>
								<!-- <view class="Occupied Selector">
									<text>{{ startTime }}</text>
								</view> -->
								<picker mode="date" :value="date" :start="startDate" :end="endDate" @change="bindDateChange">
									<view class="Occupied Selector">{{sDate}}</view>
								</picker>
							</view>
							<view class="label">
								<view class="Occupied endTime">结束时间</view>
								<!-- <view class="Occupied Selector">
									<text>{{ endTime }}</text>
								</view> -->
								<picker mode="date" :value="date" :start="startDate" :end="endDate" @change="bindDateChange2">
									<view class="Occupied Selector">{{eDate}}</view>
								</picker>
							</view>
						</view>
						
							<view class="queryBtn" @click="search()">
								<button class="uni-btn" type="primary">查询</button>
							</view>
						</view>
						<view class="list-item">
							<view v-for="item in feePayable" :key="item.id">
								<view class="content-box">
										<view class="content-container">
											<view class="content-box-left" @click="showDeatial(item.id, 1)">
												<view class="content-row">
													<view class="label font-bold">
														{{ item.title }}
												    </view>
												</view>
												<view class="content-row">
													<view class="label">报告类型：</view>
													<view class="valueText">
														{{ item.firstDescription }}
													</view>
												</view>
												<view class="content-row">
													<view class="label">送检时间：</view>
													<view class="valueText">
														{{ item.twoDescription }}
													</view>
												</view>
												<view class="content-row">
													<view class="label">报告种类：</view>
													<view class="valueText">
														{{ item.threeDescription }}
													</view>
												</view>
											</view>
											<view class="checkedbox">
												<text class="padding-right-xs">已出</text>
											</view>
										</view>
								</view>
							</view>
						</view>
						<!--未缴费订单列表结束 -->
					</scroll-view>
				</swiper-item>
				<swiper-item class="swiper-item">
					<scroll-view scroll-y style="height: 100%;">
						<!-- 已缴费订单列表 -->
						<!-- <view>已缴费</view> -->
						<view class="queryTitle">
						  <view class="queryDate">
						  	<view class="label">
						  		<view class="Occupied startTime">开始时间</view>
						  		<!-- <view class="Occupied Selector">
						  			<text>{{ startTime }}</text>
						  		</view> -->
						  		<picker mode="date" :value="date" :start="startDate" :end="endDate" @change="bindDateChange">
						  			<view class="Occupied Selector">{{sDate}}</view>
						  		</picker>
						  	</view>
						  	<view class="label">
						  		<view class="Occupied endTime">结束时间</view>
						  		<!-- <view class="Occupied Selector">
						  			<text>{{ endTime }}</text>
						  		</view> -->
						  		<picker mode="date" :value="date" :start="startDate" :end="endDate" @change="bindDateChange2">
						  			<view class="Occupied Selector">{{eDate}}</view>
						  		</picker>
						  	</view>
						  </view>

							<view class="queryBtn" @click="searchTwo()">
								<button class="uni-btn" type="primary">查询</button>
							</view>
						</view>
						<view class="list-item" v-for="item in outpatientQuery" :key="item.id">
							<view class="content-box">
									<view class="content-container">
										<view class="content-box-left" @click="showDeatial(item.id, 2)">
											<view class="content-row">
												<view class="label font-bold">
													{{ item.title }}
											    </view>
											</view>
											<view class="content-row">
												<view class="label">检查类型：</view>
												<view class="valueText">
													{{ item.firstDescription }}
												</view>
											</view>
											<view class="content-row">
												<view class="label">报告日期：</view>
												<view class="valueText">
													{{ item.twoDescription }}
												</view>
											</view>
											<view class="content-row">
												<view class="label">状态：</view>
												<view class="valueText">
													{{ item.threeDescription }}
												</view>
											</view>
										</view>
										<view class="checkedbox">
											<text class="padding-right-xs">已出</text>
										</view>
									</view>
							</view>
						</view>
						<!-- 已缴费订单列表 -->
					</scroll-view>
				</swiper-item>
			</swiper>
		</view>
	</view>
</template>

<script>
	import {
		reqOutpatientQuery,
		reqFeePayable,
		queryPatientReportByCard,
		queryPascReportList,
	} from '@/api/index.js';
	export default {
		data() {
			const currentDate = this.getDate({
				format: true
			})
			
			return {
				flag: false,
				type: 0,
				current: 0,
				startTime: '2020-06-09',
				endTime: '2020-09-09',
				date: currentDate,
				sDate: currentDate,
				eDate: currentDate,
				list: [{
						name: '检验报告单'
					},
					{
						name: '检查报告单'
					}
				],
				feePayable: [],
				outpatientQuery: [
				],
			}
		},
		computed: {
			startDate() {
				return this.getDate('start');
			},
			endDate() {
				return this.getDate('end');
			}
		},
		activated() {
			this.change(0);
			this.getFeePayable();
		},
		created(){
			this.change(0);
			this.getFeePayable();
		},
		methods: {
			// tab栏切换
			change(index) {
				this.current = index;				
				if (index === 0) {
					this.getFeePayable();
				} else {
					this.getOutpatientQuery();
				}
			},
			changeSwiper({detail: {current}}) {
				this.$refs.tabs.setFinishCurrent(current);
				this.current = current;
			},
			getFeePayable(){
				const data = {
					cardType: '09',
					cardNo: '6620200916112434450',
					presNo: '',
					patientId: '',
					startTime: this.sDate,
					endTime: this.eDate
				}
				queryPatientReportByCard(data).then(res => {
					console.log(res); 
					if (res.data.code === 200) {
						this.feePayable = res.data.rows;
					}
				});
			},
			getOutpatientQuery(){
				const data = {
					cardType: '09',
					cardNo: '6620200916112434450',
					presNo: '',
					patientId: '',
					startTime: this.sDate,
					endTime: this.eDate
					// startDay: this.startTime + ' 00:00:01',
					// endDay: this.endTime + ' 23:59:59'
				}
				queryPascReportList(data).then(res => {
					console.log(res);
					if (res.data.code === 200) {
						this.outpatientQuery = res.data.rows;
					}
					console.log(this.outpatientQuery)
				});
			},
			bindDateChange: function(e) {
				this.sDate = e.detail.value
			},
			bindDateChange2: function(e) {
				this.eDate = e.detail.value
			},
			showDeatial (no, type) {
			      console.log(type);
			      if (type === 1) {
			       uni.navigateTo({
			         url: 'lnspection-test?id='+no,
					 });
			      } else {
			        this.$Router.push({
			          name: 'outpatient-feePay-detail',
			          params: {
			            id: no,
			            type: 'outfee'
			          }
			        });
			      }
			},
			getDate(type) {
				const date = new Date();
				let year = date.getFullYear();
				let month = date.getMonth() + 1;
				let day = date.getDate();

				if (type === 'start') {
					year = year - 60;
				} else if (type === 'end') {
					year = year + 2;
				}
				month = month > 9 ? month : '0' + month;;
				day = day > 9 ? day : '0' + day;
				return `${year}-${month}-${day}`;
			},
			search(){
				this.getFeePayable();
			},
			searchTwo(){
				this.getOutpatientQuery();
			}
		},
	
	}
</script>

<style lang="scss" scoped>
	.container {
		/* #ifdef H5 || MP-ALIPAY */
		height: calc(100vh - 24px - 50px);
		/* #endif */
		/* #ifndef H5 */
		height: calc(100vh - 10rpx);
		/* #endif */
		background-color: #f3f3f3;
		padding: 130rpx 30rpx 30rpx;

		.tabs {
			position: fixed;
			/* #ifdef H5 */
			top: 44px;
			/* #endif */
			/* #ifndef H5 */
			top: 0;
			/* #endif */
			left: 0;
			display: flex;
			align-items: center;
			width: 100%;
			height: 100rpx;
			box-sizing: border-box;
			background-color: #fff;
			box-shadow: 0px 0px 10rpx rgba(0, 0, 0, 0.1);
			// border-radius: 0px 0px 10rpx 10rpx;
			z-index: 3;
		}

		.container-tabs__list {
			height: 100%;

			.container-tabs__swiper {
				height: 100%;
			}
		}

		.queryTitle {
			box-sizing: border-box;
			display: flex;
			border-radius: 10rpx;
			margin-bottom: 20rpx;

			.card {
				flex: 1;
				font-family: PingFang SC Medium;
				color: #333333;
				margin: 10px 10px;
				font-size: 14px;
			}

			.queryDate {
				flex: 1;
				font-family: PingFang SC Medium;
				color: #333333;
				margin: 10px 10px;
				font-size: 14px;

				.label {
					border-bottom: 1px solid #b5b5b5;
					box-sizing: border-box;
					display: flex;
					flex-direction: row;
					height: 35%;
					margin: 10px 5px;
					align-items: center;

					.Occupied {
						flex: 1;
					}

					.Selector {
						text-align: center;
						font-weight: bold;
					}
				}
			}

			.queryBtn {
				width: 96px;
				height: 100%;
				margin: 10px 15px 10px 0px;

				.uni-btn {
					height: 100%;
					line-height: 4;
				}
			}
		}

		.content-box {
			font-size: 14px;
		}

		.content-container {
			background-color: #fff;
			border-radius: 10rpx;
			padding: 20rpx;
			margin-bottom: 20rpx;
			display: flex;
			justify-content: space-between;
		}

		.content-box-left {
			display: flex;
			flex-direction: column;
		}

		.content-item {
			display: flex;
			flex-direction: column;
		}

		.content-row {
			display: flex;
			padding: 4rpx;

			.label {
				margin-right: 10rpx;
			}

			.valueText-fee {
				color: #FF0000;
				font-size: 14px;
				// margin-left: 10px;
			}
		}

		.checkedbox {
			display: flex;
			align-items: center;
			justify-content: center;
		}
		.font-bold{
			font-weight: bolder;
		}
	}
</style>
