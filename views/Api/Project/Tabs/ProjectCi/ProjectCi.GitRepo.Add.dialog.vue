<template>
	<xDialog>
		<xCard>
			<xForm col="1" ref="form">
				<!-- 项目别名 -->
				<xItem :configs="form.alias" />
				<!-- 项目地址 -->
				<xItem :configs="form.git_address" />
				<xItem :configs="form.username" />
				<xItem :configs="form.password" />
			</xForm>
		</xCard>
		<template #footer>
			<xBtn :configs="btnOk" />
			<xBtn @click="closeModal">{{ i18n("取消") }}</xBtn>
		</template>
	</xDialog>
</template>
<script lang="ts">
export default async function ({ project_id, onSuccess }) {
	const { useDialogProps } = await _.$importVue("/common/utils/hooks.vue");
	return defineComponent({
		props: useDialogProps(),
		data() {
			return {
				form: {
					alias: {
						value: "boundless-vue-project",
						label: i18n("项目别名"),
						rules: [_rules.required()]
					},
					git_address: {
						value: "",
						label: i18n("git地址"),
						rules: [_rules.required()],
						tips: "目前只支持http"
					},
					username: {
						value: "",
						label: i18n("用户名"),
						rules: [_rules.required()],
						msg: "用户名及密码不可展示，请务必填写正确、匹配"
					},
					password: {
						value: "",
						label: i18n("密码"),
						rules: [_rules.required()],
						msg: "用户名及密码不可展示，请务必填写正确、匹配"
					}
				}
			};
		},
		computed: {
			btnOk() {
				const vm = this;
				return {
					label: i18n("确定"),
					preset: "blue",
					async onClick() {
						const [error] = await _.$validateForm(vm.$el);
						if (error) {
							return;
						}
						try {
							const { errcode, message } = await _api.xspace.apiCicdGitAddressAdd({
								project_id: project_id,
								..._.$pickFormValues(vm.form)
							});
							if (_.$isSame(errcode, 0)) {
								_.$msg(i18n("添加代码仓库成功"));
								onSuccess();
								vm.closeModal();
							} else {
								_.$msgError(message);
							}
						} catch (error) {
							_.$msgError(error);
						}
					}
				};
			}
		},
		methods: {}
	});
}
</script>
