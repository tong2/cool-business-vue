<template>
	<cl-crud ref="Crud">
		<cl-row>
			<cl-refresh-btn />
			<cl-multi-delete-btn />
			<el-button type="primary" @click="handleExport">导出</el-button>
			<el-button type="success" @click="handleImportMain">导入主表</el-button>
			<el-button type="success" @click="handleImportSku">导入SKU</el-button>
			<cl-flex1 />
			<el-input
				v-model="searchParams.product_id"
				placeholder="搜索宝贝ID"
				clearable
				style="width: 150px; margin-right: 10px"
				@keyup.enter="search"
			/>
			<el-input
				v-model="searchParams.title"
				placeholder="搜索标题"
				clearable
				style="width: 150px; margin-right: 10px"
				@keyup.enter="search"
			/>
			<el-input
				v-model="searchParams.shop_name"
				placeholder="搜索店铺名"
				clearable
				style="width: 150px; margin-right: 10px"
				@keyup.enter="search"
			/>
			<el-input
				v-model="searchParams.category_name"
				placeholder="搜索类目名"
				clearable
				style="width: 150px"
				@keyup.enter="search"
			/>
		</cl-row>

		<cl-row>
			<cl-table ref="Table" />
		</cl-row>

		<cl-row>
			<cl-flex1 />
			<cl-pagination />
		</cl-row>

		<el-dialog title="导入商品主表记录" v-model="importMainDialogVisible" width="30%">
			<el-upload
				:action="importMainAction"
				:before-upload="beforeUpload"
				:on-success="onImportSuccess"
				:on-error="onImportError"
				accept=".xlsx"
			>
				<el-button type="primary">选择 Excel 文件</el-button>
			</el-upload>
		</el-dialog>

		<el-dialog title="导入SKU记录" v-model="importSkuDialogVisible" width="30%">
			<el-upload
				:action="importSkuAction"
				:before-upload="beforeUpload"
				:on-success="onImportSuccess"
				:on-error="onImportError"
				accept=".xlsx"
			>
				<el-button type="primary">选择 Excel 文件</el-button>
			</el-upload>
		</el-dialog>

		<el-dialog title="编辑人工单价" v-model="editDialogVisible" width="30%">
			<el-form :model="editForm" ref="editFormRef" :rules="editRules" label-width="100px">
				<el-form-item label="人工单价" prop="manual_unit_price">
					<el-input
						v-model.number="editForm.manual_unit_price"
						placeholder="请输入人工单价"
						type="number"
						clearable
					/>
				</el-form-item>
			</el-form>
			<template #footer>
				<el-button @click="editDialogVisible = false">取消</el-button>
				<el-button type="primary" @click="submitEdit">确定</el-button>
			</template>
		</el-dialog>
	</cl-crud>
</template>

<script lang="ts" name="product-main" setup>
import { useCrud, useTable } from '@cool-vue/crud';
import { ref, reactive } from 'vue';
import { ElMessage, ElLoading, ElForm, FormInstance, FormRules } from 'element-plus';
import axios from 'axios';

const Table = useTable({
	columns: [
		{ type: 'selection', width: 60 },
		{ label: '宝贝ID', prop: 'product_id', minWidth: 120 },
		{ label: '标题', prop: 'title', minWidth: 150 },
		{ label: '宝贝链接', prop: 'product_link', minWidth: 150 },
		{ label: '图片地址', prop: 'image_url', minWidth: 150 },
		{ label: '价格', prop: 'price', minWidth: 100 },
		{ label: '销量', prop: 'sales_volume', minWidth: 100 },
		{ label: '店铺名', prop: 'shop_name', minWidth: 120 },
		// { label: '店铺链接', prop: 'shop_link', minWidth: 150 },
		{ label: '类目ID', prop: 'category_id', minWidth: 100 },
		{ label: '类目名', prop: 'category_name', minWidth: 120 },
		// { label: 'SKU ID', prop: 'id', minWidth: 100 },
		{ label: 'SKU名称', prop: 'sku_name', minWidth: 120 },
		{ label: 'SKU图片', prop: 'sku_image', minWidth: 150 },
		{ label: '原价', prop: 'original_price', minWidth: 100 },
		{ label: '计算价格', prop: 'calculated_price', minWidth: 100 },
		{ label: '库存', prop: 'stock', minWidth: 100 },
		{ label: '人工单价', prop: 'manual_unit_price', minWidth: 100 },
		{
			label: '操作',
			type: 'op',
			buttons: [
				{
					label: '编辑',
					type: 'primary',
					onClick: row => handleEdit(row)
				},
				'delete'
			],
			minWidth: 120
		}
	]
});

const getDynamicPrefix = () => {
	return import.meta.env.MODE === 'development' ? '/dev' : '/api';
};

const importMainDialogVisible = ref(false);
const importSkuDialogVisible = ref(false);
const importMainAction = `${getDynamicPrefix()}/products/finance/productMain/import`;
const importSkuAction = `${getDynamicPrefix()}/products/finance/productSku/import`;

const editDialogVisible = ref(false);
const editFormRef = ref<FormInstance>();
const editForm = reactive({
	id: 0,
	manual_unit_price: null as number | null
});
const editRules: FormRules = {
	manual_unit_price: [{ type: 'number', message: '请输入有效的数字', trigger: 'blur' }]
};

const searchParams = reactive({
	product_id: '',
	title: '',
	shop_name: '',
	category_name: ''
});

const fetchProductMainList = async (params: any) => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在加载数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const response = await axios.post(
			`${getDynamicPrefix()}/products/finance/productMain/page2`,
			{
				...params,
				...searchParams
			}
		);
		loading.close();
		if (response.data && response.data.code === 1000 && response.data.data) {
			return {
				list: response.data.data.list || [],
				pagination: {
					total: response.data.data.total || 0,
					page: response.data.data.page || 1,
					size: response.data.data.size || 10
				}
			};
		} else {
			ElMessage.error(`获取商品列表失败: ${response.data?.message || '响应格式错误'}`);
			return { list: [], pagination: { total: 0, page: 1, size: 10 } };
		}
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			message = error.response?.data?.message || error.message || '网络错误或服务器无响应';
		}
		ElMessage.error(`获取商品列表失败: ${message}`);
		return { list: [], pagination: { total: 0, page: 1, size: 10 } };
	}
};

const handleExport = async () => {
	const loading = ElLoading.service({
		lock: true,
		text: '正在导出数据...',
		background: 'rgba(0, 0, 0, 0.7)'
	});
	try {
		const params = { ...Crud.value?.search?.params, ...searchParams };
		const response = await axios.get(
			`${getDynamicPrefix()}/products/finance/productMain/export`,
			{
				params,
				responseType: 'blob'
			}
		);

		const url = window.URL.createObjectURL(new Blob([response.data]));
		const link = document.createElement('a');
		link.href = url;
		const contentDisposition = response.headers['content-disposition'];
		let filename = 'product-main.xlsx';
		if (contentDisposition) {
			const filenameMatch = contentDisposition.match(/filename="?(.+)"?/i);
			if (filenameMatch && filenameMatch.length > 1) {
				filename = filenameMatch[1];
			}
		}
		link.setAttribute('download', filename);
		document.body.appendChild(link);
		link.click();
		document.body.removeChild(link);
		window.URL.revokeObjectURL(url);

		ElMessage.success('导出成功');
		loading.close();
	} catch (error: unknown) {
		loading.close();
		let message = '未知错误';
		if (axios.isAxiosError(error)) {
			if (
				error.response &&
				error.response.data instanceof Blob &&
				error.response.data.type.includes('json')
			) {
				try {
					const errorJson = JSON.parse(await error.response.data.text());
					message = errorJson.message || '导出失败';
				} catch (parseError) {
					message = '未知导出错误';
				}
			} else {
				message =
					error.response?.data?.message || error.message || '网络错误或服务器无响应';
			}
		}
		ElMessage.error(`导出失败: ${message}`);
	}
};

const handleImportMain = () => {
	importMainDialogVisible.value = true;
};

const handleImportSku = () => {
	importSkuDialogVisible.value = true;
};

const beforeUpload = (file: File): boolean | Promise<boolean> => {
	const isXlsx =
		file.type === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet';
	if (!isXlsx) {
		ElMessage.error('请上传 .xlsx 文件');
		return false;
	}

	// Read first 4 bytes to verify ZIP header
	return new Promise(resolve => {
		const reader = new FileReader();
		reader.onload = e => {
			const buffer = e.target?.result as ArrayBuffer;
			const uint8Array = new Uint8Array(buffer.slice(0, 4));
			const hex = Array.from(uint8Array)
				.map(b => b.toString(16).padStart(2, '0'))
				.join('');
			console.log('Client-side ZIP header:', hex);
			if (hex.startsWith('504b')) {
				resolve(true);
			} else {
				ElMessage.error('文件不是有效的 .xlsx 格式（无效的 ZIP 头）');
				resolve(false);
			}
		};
		reader.onerror = () => {
			ElMessage.error('无法读取文件');
			resolve(false);
		};
		reader.readAsArrayBuffer(file.slice(0, 4));
	});
};

const onImportSuccess = (response: any) => {
	if (response && response.code === 1000) {
		ElMessage.success(
			`导入成功${response.data?.count ? '，处理 ' + response.data.count + ' 条记录' : ''}`
		);
		importMainDialogVisible.value = false;
		importSkuDialogVisible.value = false;
		Crud.value?.refresh();
	} else {
		ElMessage.error(`导入失败: ${response?.message || '服务器返回错误'}`);
	}
};

const onImportError = (error: any) => {
	let message = '上传失败';
	try {
		const errorResponse = JSON.parse(error.message || '{}');
		message = errorResponse.message || message;
	} catch (e) {
		message = error.message || '未知上传错误';
	}
	ElMessage.error(`导入失败: ${message}`);
};

const handleEdit = (row: any) => {
	editForm.id = row.id;
	editForm.manual_unit_price = row.manual_unit_price;
	editDialogVisible.value = true;
};

const submitEdit = async () => {
	if (!editFormRef.value) return;
	await editFormRef.value.validate(async valid => {
		if (valid) {
			const loading = ElLoading.service({
				lock: true,
				text: '正在保存...',
				background: 'rgba(0, 0, 0, 0.7)'
			});
			try {
				const response = await axios.post(
					`${getDynamicPrefix()}/products/finance/productSku/updateById`,
					{
						id: editForm.id,
						manualUnitPrice: editForm.manual_unit_price
					}
				);
				loading.close();
				if (response.data && response.data.code === 1000) {
					ElMessage.success('更新成功');
					editDialogVisible.value = false;
					Crud.value?.refresh();
				} else {
					ElMessage.error(`更新失败: ${response.data?.message || '服务器返回错误'}`);
				}
			} catch (error: unknown) {
				loading.close();
				let message = '未知错误';
				if (axios.isAxiosError(error)) {
					message =
						error.response?.data?.message || error.message || '网络错误或服务器无响应';
				}
				ElMessage.error(`更新失败: ${message}`);
			}
		}
	});
};

const search = () => {
	Crud.value?.refresh();
};

const Crud = useCrud(
	{
		service: {
			page: fetchProductMainList,
			list: (params: any) =>
				axios
					.post(`${getDynamicPrefix()}/products/finance/productSku/list`, {
						...params,
						...searchParams
					})
					.then(res => res.data.data),
			delete: (data: any) =>
				axios
					.post(`${getDynamicPrefix()}/products/finance/productSku/delete`, data)
					.then(res => res.data)
		}
	},
	(app: any) => {
		app.refresh();
	}
);
</script>

<style scoped></style>
