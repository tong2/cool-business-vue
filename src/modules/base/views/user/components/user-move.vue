<template>
	<div class="dept-move">
		<cl-form ref="Form" />
	</div>
</template>

<script lang="ts" name="user-move" setup>
import { useCool } from '/@/cool';
import { ElMessage, ElMessageBox } from 'element-plus';
import { useCrud, useForm } from '@cool-vue/crud';

const { service } = useCool();
const Form = useForm();
const Crud = useCrud();

async function open(ids: any[]) {
	Form.value?.open({
		title: '部门转移',
		width: '500px',
		props: {
			labelWidth: '80px'
		},
		items: [
			{
				label: '选择部门',
				prop: 'departmentId',
				component: {
					name: 'cl-dept-select'
				}
			}
		],
		on: {
			async submit(data, { done, close }) {
				if (!data.departmentId) {
					ElMessage.warning('请选择部门');
					return done();
				}

				await ElMessageBox.confirm('转移到新部门，是否继续？', '提示', {
					type: 'warning'
				})
					.then(() => {
						service.base.sys.user
							.move({
								...data,
								userIds: ids
							})
							.then(() => {
								ElMessage.success('转移成功');
								Crud.value?.refresh();
								close();
							})
							.catch(err => {
								ElMessage.error(err.message);
							});
					})
					.catch(() => null);

				done();
			}
		}
	});
}

defineExpose({
	open
});
</script>
