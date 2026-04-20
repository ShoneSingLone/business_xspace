<template>
	<div class="flex vertical" id="xspaceItemInterfaceImportType">
		<xItem :configs="radio" v-model="x_item_value" />
		<div
			class="upload-container mt8"
			@dragover.prevent
			@drop.prevent="onDrop"
			@click="openFileSelector"
			v-if="_.$isSame(x_item_value, 1)">
			<div class="upload-content">
				<xIcon class="icon-upload" icon="_upload"></xIcon>
				<div class="upload-text">将JSON文件拖放到此处或点击上传</div>
			</div>
		</div>
		<div class="flex mt8" v-else>
			<xItem :configs="swaggerURL" style="--xItem-wrapper-width: 300px" class="mr4" />
			<xBtn @click="importByUrl" preset="primary">导入</xBtn>
		</div>
	</div>
</template>

<script lang="ts">
export default async function () {
	const { mixins } = await _.$importVue("/common/ui-x/common/ItemMixins.vue");
	return defineComponent({
		inject: ["APP", "inject_project"],
		mixins: [mixins],
		props: ["options"],
		setup() {
			const vm = this;

			const cpt_import_type = computed(() => {
				const type = vm.$xItemAttr("importType");
				return type;
			});
			const cpt_data_sync = computed(() => {
				const dataSync = vm.$xItemAttr("dataSync");
				return dataSync;
			});

			async function check_before_overwrite_interface(originData) {
				let typeid = vm.APP.cptProjectId;
				let apiCollections = originData.apis.map(item => {
					return {
						method: item.method,
						path: item.path
					};
				});

				let resToMerge = await _api.xspace.log_update({
					type: "project",
					typeid,
					apis: apiCollections
				});

				if (resToMerge) {
					_.$openModal({
						title: i18n("确认数据同步"),
						url: "@/views/Api/Project/Tabs/ProjectSettingConfirmMergeInterface.dialog.vue",
						parent: vm,
						domainData: resToMerge.data,
						originData,
						all_category: vm.inject_project.all_category,
						dataSync: cpt_data_sync.value
					});
				}
			}

			return {
				cptIsImportFile() {
					return this.form.importBy.value === "1";
				},
				onDrop(event) {
					const [file] = event.dataTransfer.files;
					if (!file) {
						return;
					}
					return vm.handleImport(file);
				},
				async openFileSelector() {
					/* 导入本地文件 */
					try {
						/* 读取本地文件 */
						const [file] = await _.$openFileSelector();
						if (!file) {
							return;
						}
						vm.handleImport(file);
					} catch (error) {
						_.$msgError(error);
					}
				},
				async handleImport(file) {
					try {
						const json = await _.$readFileAsText(file);
						const res = await _api.xspace.getSwaggerDataByUrl({
							type: cpt_import_type.value,
							json
						});
						if (res) {
							await check_before_overwrite_interface(res.data);
						}
					} catch (error) {
						_.$msgError(error);
					}
				},

				async importByUrl() {
					const [error] = await _.$validateForm(vm.$el);
					if (error) {
						return;
					}

					const res = await _api.xspace.getSwaggerDataByUrl({
						url: encodeURI(encodeURI(vm.swaggerURL.value)),
						type: cpt_import_type.value
					});

					if (res) {
						await check_before_overwrite_interface(res.data);
					}
				},

				radio: ref(
					defItem({
						itemType: "xItemRadioGroup",
						isButton: true,
						options: [
							{ label: "上传JSON 文件", value: "1" },
							{ label: "通过URL请求获取", value: "2" }
						]
					})
				),
				swaggerURL: ref(
					defItem({
						value: (() => {
							const url = new URL(window.location.origin);
							url.pathname = "/api/swagger-doc";
							return url.href;
						})(),
						placeholder: "请输入URL",
						rules: [_rules.required()]
					})
				)
			};
		}
	});
}
</script>

<style lang="less">
#xspaceItemInterfaceImportType {
	.upload-container {
		width: 100%;
		height: 150px;
		border: 2px dashed #dcdfe6;
		border-radius: 6px;
		cursor: pointer;
		position: relative;
		overflow: hidden;
		transition: border-color 0.3s;
	}
	.upload-container:hover {
		border-color: #409eff;
	}
	.file-input {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		opacity: 0;
		cursor: pointer;
	}
	.upload-content {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		text-align: center;
		color: #909399;
		.icon-upload {
			width: 64px;
			height: 64px;
		}
	}
	.upload-content i {
		font-size: 28px;
		margin-bottom: 10px;
	}
	.upload-text {
		font-size: 14px;
	}
}
</style>
