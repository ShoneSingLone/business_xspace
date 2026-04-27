<script lang="ts">
export default async function () {
	return {
		props: {
			show: {
				type: Boolean,
				default: false
			},
			creatingProject: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				createProjectForm: {
					name: "",
					description: "",
					visibility: "private"
				},
				createProjectFormConfigs: {
					name: {
						label: "项目名称",
						value: "",
						placeholder: "请输入项目名称",
						rules: [{ required: true, message: "请输入项目名称" }]
					},
					description: {
						label: "项目描述",
						type: "textarea",
						value: "",
						placeholder: "请输入项目描述"
					},
					visibility: {
						label: "项目可见性",
						type: "select",
						value: "private",
						options: [
							{ label: "私有", value: "private" },
							{ label: "内部", value: "internal" },
							{ label: "公开", value: "public" }
						]
					}
				}
			};
		},
		watch: {
			show(newVal) {
				if (newVal) {
					this.resetForm();
				}
			}
		},
		methods: {
			resetForm() {
				this.createProjectForm = {
					name: "",
					description: "",
					visibility: "private"
				};
				this.createProjectFormConfigs.name.value = "";
				this.createProjectFormConfigs.description.value = "";
				this.createProjectFormConfigs.visibility.value = "private";
			},
			close() {
				this.$emit("close");
			},
			create() {
				this.$emit("create", { ...this.createProjectForm });
			},
			updateName(val) {
				this.createProjectForm.name = val;
				this.createProjectFormConfigs.name.value = val;
			},
			updateDescription(val) {
				this.createProjectForm.description = val;
				this.createProjectFormConfigs.description.value = val;
			},
			updateVisibility(val) {
				this.createProjectForm.visibility = val;
				this.createProjectFormConfigs.visibility.value = val;
			}
		}
	};
}
</script>

<template>
	<xDialog v-if="show" title="创建个人项目" @close="close">
		<xForm col="1">
			<xItem :configs="createProjectFormConfigs.name" @input="updateName" />
			<xItem :configs="createProjectFormConfigs.description" @input="updateDescription" />
			<xItem :configs="createProjectFormConfigs.visibility" @input="updateVisibility" />
		</xForm>
		<template #footer>
			<xBtn @click="close">取消</xBtn>
			<xBtn :configs="{
				label: '创建',
				preset: 'blue',
				disabled: creatingProject,
				onClick: create
			}" />
		</template>
	</xDialog>
</template>