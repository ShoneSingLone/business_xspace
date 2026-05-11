<template>
	<xDialog :style="labelStyle">
		<xCard>
			<xForm col="1">
				<xItem :configs="form.uid" />
				<xItem :configs="form.username" />
				<xItem :configs="form.email" />
				<xItem :configs="form.role" />
			</xForm>
		</xCard>
		<template #footer>
			<xBtn :configs="btnOk" />
			<xBtn @click="closeModal">{{ i18n("cancel") }}</xBtn>
		</template>
	</xDialog>
</template>

<script lang="ts">
export default async function ({ closeModal, onOk, initialMember }) {
	const { useDialogProps } = await _.$importVue("/common/utils/hooks.vue");

	return defineComponent({
		props: useDialogProps(),
		data() {
			return {
				form: {
					uid: defItem({
						value: initialMember?.uid || "",
						label: "UID",
						rules: [_rules.required()]
					}),
					username: defItem({
						value: initialMember?.username || "",
						label: i18n("名称"),
						rules: [_rules.required()]
					}),
					email: defItem({
						value: initialMember?.email || "",
						label: i18n("邮箱")
					}),
					role: defItem({
						value: initialMember?.role || "dev",
						itemType: "xItemSelect",
						label: i18n("角色"),
						options: [
							{ label: i18n("组长"), value: "owner" },
							{ label: i18n("开发者"), value: "dev" },
							{ label: i18n("访客"), value: "guest" }
						]
					})
				}
			};
		},
		computed: {
			labelStyle() {
				return {
					"--xItem-label-width": "120px"
				};
			},
			cptFormData() {
				return _.$pickFormValues(this.form);
			},
			btnOk() {
				const vm = this;
				return {
					label: i18n("ok"),
					preset: "blue",
					async onClick() {
						const [hasError] = await _.$validateForm(vm.$el);
						if (hasError) return;
						const { uid, username, email, role } = vm.cptFormData;
						const member = {
							uid: Number(uid),
							username: String(username || ""),
							email: String(email || ""),
							role
						};
						if (onOk) await onOk({ member });
						closeModal();
					}
				};
			}
		}
	});
}
</script>
