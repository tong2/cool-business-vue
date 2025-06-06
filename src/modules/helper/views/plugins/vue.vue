<template>
	<el-scrollbar class="plugins__wrapper">
		<div class="plugins">
			<el-tabs v-model="tab.active" type="card" @tab-change="tab.onChange">
				<el-tab-pane label="已安装插件" name="installed" />
				<el-tab-pane label="插件市场" name="shop" />
			</el-tabs>

			<el-row :gutter="10">
				<el-col
					v-for="(item, index) in list"
					:key="index"
					:xs="24"
					:sm="12"
					:md="8"
					:lg="6"
				>
					<div class="scope">
						<div class="c">
							<img class="logo" :src="item.logo" />

							<div class="det">
								<div class="tag">
									<el-tag size="small" effect="dark">{{ item.name }}</el-tag>
									<el-tag size="small" effect="dark" type="success"
										>v{{ item.version || '1.0.0' }}</el-tag
									>
								</div>

								<p class="title">
									{{ item.label || '未知' }}
								</p>

								<p class="desc">
									{{ item.description || '暂无描述' }}
								</p>

								<div class="author">
									<span>{{ item.author || 'Ta' }}：</span>
									<span>{{ item.updateTime || '2024-01-01' }}</span>
								</div>
							</div>
						</div>

						<div class="f">
							<cl-flex1 />
							<el-button round @click="det.open(item)" v-if="item.demo">
								示例
							</el-button>

							<el-button
								type="success"
								round
								@click="det.openDoc(item.doc)"
								v-if="item.doc"
							>
								文档
							</el-button>
						</div>
					</div>
				</el-col>
			</el-row>

			<cl-dialog v-model="det.visible" :title="det.title" width="60%">
				<el-tabs v-model="det.active" type="card" @tab-change="tab.onChange">
					<el-tab-pane
						v-for="(item, index) in det.tabs"
						:key="index"
						:label="item.name"
						:name="index"
						:lazy="index != 0"
					>
						<component :is="item.component" />
					</el-tab-pane>
				</el-tabs>
			</cl-dialog>
		</div>
	</el-scrollbar>
</template>

<script lang="ts" setup name="helper-plugins-vue">
import { reactive, nextTick, markRaw } from 'vue';
import { module, useCool } from '/@/cool';
import { isEmpty, isFunction, isString, startsWith } from 'lodash-es';

const { router } = useCool();

// 选项卡
const tab = reactive({
	active: 'installed',

	onChange(val: any) {
		if (val == 'shop') {
			nextTick(() => {
				tab.active = 'installed';
				window.open('https://cool-js.com/plugin');
			});
		}
	}
});

// 插件列表
const list = module.list
	.filter(e => e.type == 'plugins')
	.map(e => {
		if (e.author == 'COOL') {
			e.logo = 'https://cool-js.com/logo.png';
		}

		return {
			...e,
			children: e.views
		};
	});

// 插件详情
const det = reactive({
	visible: false,
	title: '',
	active: 0,
	tabs: [] as any[],

	async open(item: any) {
		det.active = 0;

		if (isString(item.demo)) {
			if (startsWith(item.demo, 'http')) {
				window.open(item.demo);
			} else {
				router.push(item.demo);
			}
		} else {
			det.visible = true;
			det.title = item.label;

			det.tabs = await Promise.all(
				(item.demo || []).map(async (e: any) => {
					if (e) {
						const c = await (isFunction(e.component) ? e.component() : e.component);

						return {
							...e,
							component: markRaw(c.default)
						};
					}
				})
			);
		}
	},

	openDoc(url: string) {
		window.open(url);
	}
});
</script>

<style lang="scss" scoped>
@forward '../../static/index.scss';
</style>
