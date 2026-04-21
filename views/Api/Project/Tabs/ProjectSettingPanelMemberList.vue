<template>
	<div class="width100 overflow-hidden">
		<xTableVir :columns="columns" :data="cptProjectMembers" />
	</div>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["APP"],
		data() {
			const vm = this;
			return {
				item_role: defItem({
					itemType: "xItemSelect",
					options: [
						{ label: "组长", value: Vue._xspace_var.OWNER },
						{ label: "开发者", value: Vue._xspace_var.DEV },
						{ label: "访客", value: Vue._xspace_var.GUEST }
					],
					onEmitValue({ val: role, xItem }) {
						const { rowIndex, rowData } = xItem.payload;
						if (rowData.role === role) {
							return;
						}
						vm.changeMemberRole({ role, uid: rowData.uid, index: rowIndex });
					}
				}),
				columns: [
					{
						prop: `a`,
						label: `b`,
						headerCellRenderer() {
							return hSpan([`成员 ${vm.cptProjectMembers.length} 人`]);
						},
						cellRenderer: ({ rowData }) => {
							return hDiv({ staticClass: "flex middle", key: rowData.uid }, [
								h("xItem", {
									staticClass: "mr4 ml4",
									style: "--xItem-wrapper-width:32px;",
									configs: {
										value: rowData.uid || "",
										itemType: "xspaceItemAvatar",
										disabled: true
									}
								}),
								h(
									"span",
									{
										staticClass: "pointer",
										async onClick() {
											return _.$openModal({
												title: i18n("个人中心"),
												url: "@/views/User/UserProfile.Dialog.vue",
												parent: vm,
												userId: rowData.uid,
												canModifyAvatar: rowData.uid === vm.APP.user._id,
												onOk() {
													vm.APP.updateGroupProjectList();
												}
											});
										}
									},
									[rowData.username]
								),
								hDiv({ class: "flex1" })
							]);
						}
					},
					{
						prop: `b`,
						label: `b`,
						width: 256,
						headerCellRenderer() {
							return hxBtn({
								vIf: vm.cptAuth,
								configs: {
									label: "添加成员",
									preset: "blue",
									icon: "el-icon-plus",
									onClick: vm.addMember
								}
							});
						},
						cellRenderer({ rowData, rowIndex }) {
							if (vm.cptAuth) {
								return hDiv({ staticClass: "flex middle width100" }, [
									h("xItem", {
										configs: vm.item_role,
										value: rowData.role,
										payload: { rowIndex, rowData },
										class: "mr8"
									}),
									hxBtn({
										configs: {
											label: "移除",
											preset: "danger",
											onClick() {
												vm.removeMember(rowData);
											}
										}
									})
								]);
							} else {
								const ROLE_MAP = {
									owner: i18n("组长"),
									dev: i18n("开发者"),
									guest: i18n("访客")
								};
								// 非管理员可以看到权限 但无法修改

								return hDiv({ staticClass: "flex middle width100" }, [
									ROLE_MAP[rowData.role]
								]);
							}
						}
					}
				]
			};
		},
		methods: {
			async addMember() {
				const vm = this;
				_.$openModal({
					title: i18n("添加成员"),
					url: "@/views/Api/Group/Section/MemberList/GroupSectionMemberList.AddMember.vue",
					parent: vm,
					async onOk({ member_uids, role, dialogVm }) {
						const { data } = await _api.xspace.project_add_member({
							id: vm.APP.cptProject?._id,
							member_uids,
							role
						});
						const { add_members, exist_members } = data;
						const addLength = add_members.length;
						const existLength = exist_members.length;
						_.$msg(`新增 ${addLength} 人， ${existLength} 人已存在`);
						vm.APP.updateGroupProjectList();
						dialogVm.closeModal();
					}
				});
			},
			async removeMember(row) {
				const vm = this;
				const { data } = await _api.xspace.project_del_member({
					id: vm.APP.cptProject._id,
					member_uid: row.uid
				});
				_.$msg(`移除成功`);
				vm.APP.updateGroupProjectList();
			},

			async changeMemberRole({ role, uid, index }) {
				const group_id = this.APP.cptCurrentGroup._id;
				_.$loading(true);
				try {
					if (role) {
						await _api.xspace.project_change_member_role({
							id: group_id,
							member_uid: uid,
							role
						});
						this.APP.groupMemberList[index].role = role;
					}
				} catch (error) {
					_.$msgError(error);
				} finally {
					_.$loading(false);
				}
			},
			cptAvatarUrl(id) {
				return _common_utils.avatar_url(id);
			}
		},
		computed: {
			cptProjectMembers() {
				return this.APP.cptProject?.members || [];
			},
			isShow() {
				return true;
			},
			cptAuth() {
				if (this.APP.user?.role === Vue._xspace_var.ADMIN) {
					return true;
				}
				return _.some(this.APP?.cptProject?.members, member => {
					return member.uid === this.APP?.user?._id;
				});
			}
		},
		watch: {
			"APP.cptGroupId": {
				immediate: true,
				async handler(group_id) {
					if (group_id) {
						try {
							this.APP.updateGroupProjectList();
						} catch (error) {
							_.$msgError(error);
						}
					}
				}
			}
		}
	});
}
</script>
<style lang="less"></style>
