<template>
	<div class="table">
		<div class="crumbs">
			<el-breadcrumb separator="/">
				<el-breadcrumb-item><i class="el-icon-lx-cascades"></i> 基础表格</el-breadcrumb-item>
			</el-breadcrumb>
		</div>
		<div class="container">
			<div class="handle-box">
				<el-button type="primary" icon="search" @click="search">查询</el-button>
				<el-button type="primary" icon="search" @click="search">详情</el-button>
				<el-button type="primary" icon="search" @click="search">修改</el-button>
				<el-button type="primary" icon="search" @click="search">审核</el-button>
				<el-button type="primary" icon="search" @click="search">退审</el-button>
				<el-button type="primary" icon="search" @click="search">汇总</el-button>
				<el-button type="primary" icon="search" @click="search">退回</el-button>
				<el-button type="primary" class="report" icon="search" @click="search">上报</el-button>
			</div>
			<div class="handle-box">
				<!--<el-button type="primary" icon="delete" class="handle-del mr10" @click="delAll">批量删除</el-button>-->
				<span>机构：</span>
				<el-select v-model="select_cate" placeholder="筛选省份" class="handle-select mr10">
					<el-option key="1" label="广东省" value="广东省"></el-option>
					<el-option key="2" label="湖南省" value="湖南省"></el-option>
				</el-select>
				<el-radio-group v-model="form.type">
					<el-radio label="步步高" name="type"></el-radio>
					<el-radio label="小天才" name="type"></el-radio>
					<el-radio label="imoo" name="type"></el-radio>
				</el-radio-group>
				<span class="demonstration">报表周期</span>
				<el-date-picker v-model="value4" type="month" placeholder="默认当月">
				</el-date-picker>
				<!--<el-input v-model="select_word" placeholder="筛选关键词" class="handle-input mr10"></el-input>-->
			</div>
			<el-table :data="data" border class="table" ref="multipleTable" @selection-change="handleSelectionChange">
				<el-table-column type="selection" width="55" align="center"></el-table-column>
				<el-table-column prop="date" label="报表名称" width="150">
				</el-table-column>
				<el-table-column prop="name" label="报表状态" width="120">
				</el-table-column>
				<el-table-column prop="address" label="机构名称-文号" :formatter="formatter">
				</el-table-column>
				<el-table-column prop="address" label="创建时间" :formatter="formatter">
				</el-table-column>
			</el-table>
			<!--<div class="pagination">
				<el-pagination background @current-change="handleCurrentChange" layout="prev, pager, next" :total="1000">
				</el-pagination>
			</div>-->
			<vPagenation @changeCurrentPage="handleCurrentChange" @changePageSize="handleSizeChange" :myTotal="pageTotal"></vPagenation>
		</div>

		<!-- 编辑弹出框 -->
		<el-dialog title="编辑" :visible.sync="editVisible" width="30%">
			<el-form ref="form" :model="form" label-width="50px">
				<el-form-item label="日期">
					<el-date-picker type="date" placeholder="选择日期" v-model="form.date" value-format="yyyy-MM-dd" style="width: 100%;"></el-date-picker>
				</el-form-item>
				<el-form-item label="姓名">
					<el-input v-model="form.name"></el-input>
				</el-form-item>
				<el-form-item label="姓名">
					<el-input v-model="form.name"></el-input>
				</el-form-item>
				<el-form-item label="地址">
					<el-input v-model="form.address"></el-input>
				</el-form-item>

			</el-form>
			<span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
		</el-dialog>

		<!-- 删除提示框 -->
		<el-dialog title="提示" :visible.sync="delVisible" width="300px" center>
			<div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
			<span slot="footer" class="dialog-footer">
                <el-button @click="delVisible = false">取 消</el-button>
                <el-button type="primary" @click="deleteRow">确 定</el-button>
            </span>
		</el-dialog>
	</div>
</template>

<script>
	import bus from '../common/bus';
	import vPagenation from '../common/Pagenation.vue';
	export default {
		name: 'basetable',
		data() {
			return {
				currentPage: 1,
				pagerCount: 1,
				pageSize: 50,
				pageTotal: 0,
				url: './static/vuetable.json',
				tableData: [],
				cur_page: 1,
				multipleSelection: [],
				select_cate: '',
				select_word: '',
				del_list: [],
				is_search: false,
				editVisible: false,
				delVisible: false,
				form: {
					name: '',
					date: '',
					address: ''
				},
				idx: -1
			}
		},
		components: {
			vPagenation,
		},
		created() {
			this.getData();
		},
		computed: {
			data() {
				return this.tableData.filter((d) => {
					let is_del = false;
					for(let i = 0; i < this.del_list.length; i++) {
						if(d.name === this.del_list[i].name) {
							is_del = true;
							break;
						}
					}
					if(!is_del) {
						if(d.address.indexOf(this.select_cate) > -1 &&
							(d.name.indexOf(this.select_word) > -1 ||
								d.address.indexOf(this.select_word) > -1)
						) {
							return d;
						}
					}
				})
			}
		},
		methods: {
			//分页当前页
			handleCurrentChange(val) {
				var this_ = this;
				this.pagerCount = val;
				var data = {
					'pagerCount': val,
					'pageSize': this.pageSize,
				};
				this_.search(this_, data);
			},
			//分页改变每页显示条数
			handleSizeChange(val) {
				var this_ = this;
				this.pageSize = val;
				var data = {
					'pageSize': val,
					'pagerCount': this.pagerCount,
				};
				this_.search(this_, data);
			},
			// 分页导航
			//			handleCurrentChange(val) {
			//				this.cur_page = val;
			//				this.getData();
			//			},
			// 获取 easy-mock 的模拟数据
			getData() {
				// 开发环境使用 easy-mock 数据，正式环境使用 json 文件
				if(process.env.NODE_ENV === 'development') {
					this.url = '/ms/table/list';
				};
				this.$axios.post(this.url, {
					page: this.cur_page
				}).then((res) => {
					this.tableData = res.data.list;
				})
			},
			search() {
				this.is_search = true;
			},
			formatter(row, column) {
				return row.address;
			},
			filterTag(value, row) {
				return row.tag === value;
			},
			handleEdit(index, row) {
				this.idx = index;
				const item = this.tableData[index];
				this.form = {
					name: item.name,
					date: item.date,
					address: item.address
				}
				this.editVisible = true;
			},
			handleDelete(index, row) {
				this.idx = index;
				this.delVisible = true;
			},
			delAll() {
				const length = this.multipleSelection.length;
				let str = '';
				this.del_list = this.del_list.concat(this.multipleSelection);
				for(let i = 0; i < length; i++) {
					str += this.multipleSelection[i].name + ' ';
				}
				this.$message.error('删除了' + str);
				this.multipleSelection = [];
			},
			handleSelectionChange(val) {
				this.multipleSelection = val;
			},
			// 保存编辑
			saveEdit() {
				this.$set(this.tableData, this.idx, this.form);
				this.editVisible = false;
				this.$message.success(`修改第 ${this.idx+1} 行成功`);
			},
			// 确定删除
			deleteRow() {
				this.tableData.splice(this.idx, 1);
				this.$message.success('删除成功');
				this.delVisible = false;
			}
		}
	}
</script>

<style scoped>
	.handle-box {
		margin-bottom: 20px;
	}
	
	.handle-select {
		width: 120px;
	}
	
	.handle-input {
		width: 300px;
		display: inline-block;
	}
	
	.del-dialog-cnt {
		font-size: 16px;
		text-align: center
	}
	
	.table {
		width: 100%;
		font-size: 14px;
	}
	
	.red {
		color: #ff0000;
	}
	.report{float: right;}
</style>