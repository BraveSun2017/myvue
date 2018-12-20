<template>
	<div class="block pagination-box">
		<el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="[50, 60, 70]" layout="total, sizes,slot" :total="myTotal">
			<span class="myPager" @click="jumpPage('first')" title="首页">&lt&lt</span>
		</el-pagination>
		<el-pagination layout="slot">
			<span class="myPager" @click="jumpPage('prev')" title="上一页">&lt</span>
		</el-pagination>
		<el-pagination layout="slot">
			<em class="nowPage" v-text="'第'+pagerCount+'页'"></em>
		</el-pagination>
		<el-pagination layout="slot">
			<span class="myPager" @click="jumpPage('next')" title="下一页">&gt</span>
		</el-pagination>
		<el-pagination @current-change="handleCurrentChange" :current-page="currentPage" layout="slot, jumper">
			<span class="myPager" @click="jumpPage('last')" title="末页">&gt&gt</span>
		</el-pagination>
	</div>
</template>

<script>
	import bus from '../common/bus';
	export default {
		props: {
			myTotal: Number
		},
		data() {
			return {
				currentPage: 1,
				pagerCount: 1,
				pageSize: 50,
				pageTotal: 0,
				total: 0,
			}
		},
		methods: {
			jumpPage(val) {
				switch(val) {
					case "first": //跳转到首页
						if(this.pagerCount > 1) {
							this.pagerCount = 1;
							this.$emit('changeCurrentPage', 1, this.pageSize);
						} else {
							this.$utils.message(this, '已经是第一页了');
						}
						break;
					case "last": //跳转到末页
						if(this.pagerCount < parseInt(this.myTotal / this.pageSize) + 1) {
							this.pagerCount = parseInt(this.myTotal / this.pageSize) + 1;
							this.$emit('changeCurrentPage', this.pagerCount, this.pageSize);
						} else {
							this.$utils.message(this, '已经是最后一页了');
						}
						break;
					case "prev": //跳转到上一页
						if(this.pagerCount > 1) {
							this.pagerCount = this.pagerCount - 1;
							this.$emit('changeCurrentPage', this.pagerCount, this.pageSize);
						} else {
							this.$utils.message(this, '已经是第一页了');
						}

						break;
					case "next": //跳转到下一页
						if(this.pagerCount < parseInt(this.myTotal / this.pageSize) + 1) {
							this.pagerCount = this.pagerCount + 1;
							this.$emit('changeCurrentPage', this.pagerCount, this.pageSize);
						} else {
							this.$utils.message(this, '已经是最后一页了');
						}
						break;
				}
			},
			//分页当前页
			handleCurrentChange(val) {
				this.pagerCount = val;
				this.$emit('changeCurrentPage', val, this.pageSize);
			},
			//分页改变每页显示条数
			handleSizeChange(val) {
				this.pageSize = val;
				this.$emit('changePageSize', val, this.pagerCount);
			},
		}
	}
</script>

<style scoped>
	.myPager,
	.nowPage {
		color: #606266;
		font-weight: normal;
		cursor: pointer;
	}
	
	.myPager:hover {
		color: #336ed4;
	}
	
	.myPager {
		padding: 0 8px;
	}
	
	.nowPage {
		font-style: normal;
		line-height: 28px;
	}
	
	.pagination-box {
		min-width: 300px;
		float: right;
		margin-top: 5px;
	}
	
	.el-pagination {
		float: left;
	}
	
	.block {
		float: right;
	}
	/*-webkit-box-sizing: border-box;
    white-space: nowrap;
    list-style: none;*/
</style>