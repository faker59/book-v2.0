<template>
	<div class="app-container">
		<el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" label-width="68px">
			<el-form-item label="名称" prop="pptName">
				<el-input v-model="queryParams.pptName" placeholder="请输入名称" clearable size="small"
					@keyup.enter.native="handleQuery" />
			</el-form-item>

			<el-form-item label="类型" prop="pptType">
				<el-select v-model="queryParams.pptType" placeholder="请选择类型" clearable size="small">
					<el-option v-for="dict in pptTypeOptions" :key="dict.dictValue" :label="dict.dictLabel"
						:value="dict.dictValue" />
				</el-select>
			</el-form-item>
			<el-form-item>
				<el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
				<el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
			</el-form-item>
		</el-form>

		<el-row :gutter="10" class="mb8">
			<el-col :span="1.5">
				<el-button type="primary" plain icon="el-icon-plus" size="mini" @click="handleAdd"
					v-hasPermi="['ppt:ppt:add']">新增</el-button>
			</el-col>
			<el-col :span="1.5">
				<el-button type="success" plain icon="el-icon-edit" size="mini" :disabled="single" @click="handleUpdate"
					v-hasPermi="['ppt:ppt:edit']">修改</el-button>
			</el-col>
			<el-col :span="1.5">
				<el-button type="danger" plain icon="el-icon-delete" size="mini" :disabled="multiple"
					@click="handleDelete" v-hasPermi="['ppt:ppt:remove']">删除</el-button>
			</el-col>
			<el-col :span="1.5">
				<el-button type="warning" plain icon="el-icon-download" size="mini" :loading="exportLoading"
					@click="handleExport" v-hasPermi="['ppt:ppt:export']">导出</el-button>
			</el-col>
			<right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
		</el-row>

		<el-table v-loading="loading" :data="pptList" @selection-change="handleSelectionChange"
              :default-sort = "{prop: 'pptDesc', order: 'descending'}"
    >
			<el-table-column type="selection" width="55" align="center" />
			<el-table-column label="ID" align="center" prop="pptId" />
			<el-table-column label="名称" align="center" prop="pptName" />
			<el-table-column label="图片" align="center" prop="pptPic">
				<template slot-scope="scope">
					<el-tooltip class="item" effect="dark" placement="top">
						<el-image slot="content" :src="baseUrl+scope.row.pptPic" style="width: 200px;height: 200px">
						</el-image>
						<el-image :src="baseUrl+scope.row.pptPic"></el-image>
					</el-tooltip>
				</template>
			</el-table-column>
			<el-table-column label="跳转路径" align="center" prop="pptUrl" />
			<el-table-column label="排序" align="center" prop="pptDesc"  sortable/>
			<el-table-column label="类型" align="center" prop="pptType">
				<template slot-scope="scope">
					<dict-tag :options="pptTypeOptions" :value="scope.row.pptType" />
				</template>
			</el-table-column>
			<el-table-column label="操作" align="center" class-name="small-padding fixed-width">
				<template slot-scope="scope">
					<el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)"
						v-hasPermi="['ppt:ppt:edit']">修改</el-button>
					<el-button size="mini" type="text" icon="el-icon-delete" @click="handleDelete(scope.row)"
						v-hasPermi="['ppt:ppt:remove']">删除</el-button>
				</template>
			</el-table-column>
		</el-table>

		<pagination v-show="total>0" :total="total" :page.sync="queryParams.pageNum" :limit.sync="queryParams.pageSize"
			@pagination="getList" />

		<!-- 添加或修改前台首页幻灯片对话框 -->
		<el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
			<el-form ref="form" :model="form" :rules="rules" label-width="80px">
				<el-form-item label="名称" prop="pptName">
					<el-input v-model="form.pptName" placeholder="请输入名称" />
				</el-form-item>
				<el-form-item label="图片">
					<imageUpload v-model="form.pptPic" />
				</el-form-item>
				<el-form-item label="跳转路径" prop="pptUrl">
					<el-input v-model="form.pptUrl" placeholder="请输入跳转路径" />
				</el-form-item>
				<el-form-item label="排序" prop="pptDesc">
					<el-input v-model="form.pptDesc" placeholder="请输入排序" />
				</el-form-item>
				<el-form-item label="类型" prop="pptType">
					<el-select v-model="form.pptType" placeholder="请选择类型">
						<el-option v-for="dict in pptTypeOptions" :key="dict.dictValue" :label="dict.dictLabel"
							:value="parseInt(dict.dictValue)"></el-option>
					</el-select>
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button type="primary" @click="submitForm">确 定</el-button>
				<el-button @click="cancel">取 消</el-button>
			</div>
		</el-dialog>
	</div>
</template>

<script>
	import {
		listPpt,
		getPpt,
		delPpt,
		addPpt,
		updatePpt,
		exportPpt
	} from "@/api/ppt/ppt";

  import img from '../../../static/imageUrls/imageUrls.json'


	export default {
		name: "Ppt",
		data() {
			return {
        //图片存放处
        imageGroups: img.imageUrls,
				baseUrl: process.env.VUE_APP_BASE_API,
				// 遮罩层
				loading: true,
				// 导出遮罩层
				exportLoading: false,
				// 选中数组
				ids: [],
				// 非单个禁用
				single: true,
				// 非多个禁用
				multiple: true,
				// 显示搜索条件
				showSearch: true,
				// 总条数
				total: 0,
				// 前台首页幻灯片表格数据
				pptList: [],
				// 弹出层标题
				title: "",
				// 是否显示弹出层
				open: false,
				// 类型字典
				pptTypeOptions: [],
				// 查询参数
				queryParams: {
					pageNum: 1,
					pageSize: 10,
					pptName: null,
					pptDesc: null,
					pptType: null
				},
				// 表单参数
				form: {},
				// 表单校验
				rules: {}
			};
		},
		created() {
			this.getList();
			this.getDicts("ppt_type").then(response => {
				this.pptTypeOptions = response.data;
			});
		},
		methods: {
			/** 查询前台首页幻灯片列表 */
			getList() {
				this.loading = true;
				listPpt(this.queryParams).then(response => {
					this.pptList = response.rows;
					this.total = response.total;
					this.loading = false;
				});
			},
			// 取消按钮
			cancel() {
				this.open = false;
				this.reset();
			},
			// 表单重置
			reset() {
				this.form = {
					pptId: null,
					pptName: null,
					pptPic: null,
					pptUrl: null,
					pptDesc: null,
					pptType: null
				};
				this.resetForm("form");
			},
			/** 搜索按钮操作 */
			handleQuery() {
				this.queryParams.pageNum = 1;
				this.getList();
			},
			/** 重置按钮操作 */
			resetQuery() {
				this.resetForm("queryForm");
				this.handleQuery();
			},
			// 多选框选中数据
			handleSelectionChange(selection) {
				this.ids = selection.map(item => item.pptId)
				this.single = selection.length !== 1
				this.multiple = !selection.length
			},
			/** 新增按钮操作 */
			handleAdd() {
				this.reset();
				this.open = true;
				this.title = "添加前台首页幻灯片";
			},
			/** 修改按钮操作 */
			handleUpdate(row) {
				this.reset();
				const pptId = row.pptId || this.ids
				getPpt(pptId).then(response => {
					this.form = response.data;
					this.open = true;
					this.title = "修改前台首页幻灯片";
				});
			},
			/** 提交按钮 */
			submitForm() {
				this.$refs["form"].validate(valid => {
					if (valid) {
						if (this.form.pptId != null) {
							updatePpt(this.form).then(response => {
								this.msgSuccess("修改成功");
								this.open = false;
								this.getList();
							});
						} else {
							addPpt(this.form).then(response => {
								this.msgSuccess("新增成功");
								this.open = false;
								this.getList();
							});
						}
					}
				});
			},
			/** 删除按钮操作 */
			handleDelete(row) {
				const pptIds = row.pptId || this.ids;
				this.$confirm('是否确认删除前台首页幻灯片编号为"' + pptIds + '"的数据项?', "警告", {
					confirmButtonText: "确定",
					cancelButtonText: "取消",
					type: "warning"
				}).then(function() {
					return delPpt(pptIds);
				}).then(() => {
					this.getList();
					this.msgSuccess("删除成功");
				}).catch(() => {});
			},
			/** 导出按钮操作 */
			handleExport() {
				const queryParams = this.queryParams;
				this.$confirm('是否确认导出所有前台首页幻灯片数据项?', "警告", {
					confirmButtonText: "确定",
					cancelButtonText: "取消",
					type: "warning"
				}).then(() => {
					this.exportLoading = true;
					return exportPpt(queryParams);
				}).then(response => {
					this.download(response.msg);
					this.exportLoading = false;
				}).catch(() => {});
			}
		}
	};
</script>
