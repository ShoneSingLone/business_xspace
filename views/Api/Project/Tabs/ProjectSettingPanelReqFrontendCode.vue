<style lang="less"></style>
<template>
	<div class="flex vertical height100 overflow-auto">
		<xAutoResizer #default="{ width, height }" class="flex flex1 height1px">
			<xItem
				:configs="form.editor"
				@save="btnOk.onClick"
				:style="{ height: height + 'px' }"
				class="flex1" />
			<div class="x-padding overflow-hidden flex1">
				<xMd :md="cptCode" :style="{ height: height + 'px' }" />
			</div>
		</xAutoResizer>
		<div class="flex middle mt">
			<xGap f />
			<xBtn :configs="btnOk" />
		</div>
	</div>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["APP", "inject_project"],
		data() {
			return {
				mdTips: ``,
				form: defItems({
					editor: {
						label: "",
						type: "textarea",
						itemType: "xItemMonaco",
						language: "javascript",
						value: ""
					}
				})
			};
		},
		watch: {
			"APP.cptProject.requestCode": {
				immediate: true,
				handler() {
					const code =
						this.APP.cptProject.requestCode || _common_utils.RequestCode.toString();
					this.form.editor.value = code;
				}
			}
		},
		computed: {
			cptCode() {
				try {
					const requestCode = this.returnRequestCode();
					const code = requestCode({
						title: "TitleDemo",
						path: "/path/demo",
						method: "GET",
						group_id: "1",
						project_id: "2",
						interface_id: "3",
						camelCase: _.camelCase
					});
					return code;
				} catch (error) {
					return error.message;
				}
			},
			btnOk() {
				const vm = this;
				return {
					label: "确定",
					preset: "blue",
					onClick: async () => {
						try {
							const dataForm = {
								requestCode: this.form.editor.value,
								id: this.APP.cptProjectId
							};
							await _api.xspace.project_update(dataForm);
							this.APP.updateGroupProjectList();
							_.$msg("更新成功");
						} catch (error) {
							_.$msgError(error);
						}
					}
				};
			}
		},
		methods: {
			returnRequestCode() {
				try {
					const fn = new Function("params", `return (${this.form.editor.value})(params)`);
					return fn;
				} catch (error) {
					return () => error.message;
				}
			}
		}
	});
}
</script>
