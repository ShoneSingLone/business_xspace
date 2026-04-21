<template>
	<div class="x-page-view flex1 height100" id="ProjectSettingPanelDataImportExport">
		<div class="flex height100 width100">
			<xCard header="数据导入" class="flex1 height100">
				<xForm col="1">
					<xItem :configs="form.curImportType" />
					<xItem :configs="form.dataSync" />
					<xItem
						:configs="form.importBy"
						:importType="form.curImportType.value"
						:dataSync="form.dataSync.value"
						span="full"
						style="
							--xItem-controller-flex: unset;
							--xItem-controller-width: 100%;
							--xItem-layout-justify-content: flex-start;
						" />
				</xForm>
			</xCard>
			<xGap l />
			<xCard header="数据导出" class="flex1 height100">
				<xForm>
					<xItem :configs="form.curImportType" />
				</xForm>
			</xCard>
		</div>
	</div>
</template>

<script lang="ts">
export default async function () {
	const { useTabName } = await _.$importVue("/common/utils/hooks.vue");
	return {
		name: "ProjectSettingPanelDataImportExportVue",
		inject: ["APP", "inject_project"],
		data() {
			const vm = this;
			return {
				form: defItems({
					swaggerURL: {
						placeholder:
							"GET 请求URL地址，返回JSON数据，如：http://127.0.0.1:8080/api/swagger.json",
						value: "http://localhost:3001/api/swagger-doc",
						rules: [
							{
								validator: () => "",
								trigger: ["change", "blur"]
							}
						]
					},
					curImportType: {
						value: "swagger",
						label: "数据采用的标准",
						itemType: "xItemRadioGroup",
						isButton: true,
						options: [
							{
								label: "Swagger",
								value: "swagger"
							}
						]
					},
					dataSync: {
						value: "normal",
						label: "数据覆盖模式",
						itemType: "xItemRadioGroup",
						isButton: true,
						minWidth: 100,
						options: [
							{ label: "普通模式", value: "normal" },
							{ label: "智能合并", value: "good" },
							{ label: "完全覆盖", value: "merge" }
						],
						tips() {
							return hDiv([
								hDiv({}, [
									h("xTag", { class: "mr" }, ["普通模式"]),
									hSpan({}, ["不导入已存在的接口"])
								]),
								hDiv({}, [
									h("xTag", { class: "mr mt" }, ["智能合并"]),
									hSpan({}, [
										"已存在的接口，将合并返回数据的 response，适用于导入了 swagger 数据，保留对数据结构的改动 "
									])
								]),
								hDiv({}, [
									h("xTag", { class: "mr mt" }, ["完全覆盖"]),
									hSpan({}, [
										"不保留旧数据，完全使用新数据，适用于接口定义完全交给后端定义"
									])
								])
							]);
						}
					},
					importBy: {
						value: "2",
						label: "导入方式",
						itemType: "xspaceItemInterfaceImportType"
					}
				})
			};
		},
		computed: {
			cptParams() {
				return _.$pickFormValues(this.form);
			}
		}
	};
}
</script>

<style lang="scss">
#ProjectSettingPanelDataImportExport {
	.xBlock + .xBlock {
		margin-left: 20px;
		margin-top: 0;
	}
}
</style>
