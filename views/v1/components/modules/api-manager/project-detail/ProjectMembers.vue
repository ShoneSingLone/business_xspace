<script lang="ts">
export default async function () {
	const api = await _.$importVue("@/utils/api.vue");

	return {
		props: {
			project: {
				type: Object,
				default: () => ({})
			},
			projectData: {
				type: Object,
				default: () => ({})
			}
		},
		data() {
			const vm = this;
			return {
				members: [],
				isLoading: false,
				searchQuery: "",
				configsSearch: defItem({
					isSearch: false,
					value: "",
					placeholder: "搜索成员...",
					onEmitValue() {
						vm.searchQuery = vm.configsSearch.value;
					},
					clearable: true
				}),
				columns: [
					{
						prop: "username",
						label: `成员`,
						headerCellRenderer() {
							return hSpan([`成员 ${vm.filteredMembers.length} 人`]);
						},
						cellRenderer: ({ rowData }) => {
							return hDiv({ staticClass: "flex middle", key: rowData.uid }, [
								h("xItem", {
									staticClass: "mr4 ml4",
									configs: {
										value: rowData.uid || "",
										itemType: "xspaceItemAvatar",
										disabled: true
									}
								}),
								h("span", { staticClass: "pointer" }, rowData.username)
							]);
						}
					},
					{
						prop: "email",
						label: "邮箱"
					},
					{
						prop: "role",
						label: "角色",
						cellRenderer: ({ rowData }) => {
							const roleNames = {
								owner: "所有者",
								admin: "管理员",
								dev: "开发者",
								guest: "访客"
							};
							return hSpan({ staticClass: "project-members__role-badge" }, roleNames[rowData.role] || rowData.role);
						}
					}
				]
			};
		},
		mounted() {
			this.loadMembers();
		},
		computed: {
			filteredMembers() {
				if (!this.searchQuery) return this.members;
				const query = this.searchQuery.toLowerCase();
				return this.members.filter(member => 
					member.username?.toLowerCase().includes(query) ||
					member.email?.toLowerCase().includes(query)
				);
			}
		},
		methods: {
			async loadMembers() {
				if (!this.project?.id) return;
				
				this.isLoading = true;
				try {
					const response = await api.getProjectMemberList({
						id: this.project.id
					});
					if (response.errcode === 0) {
						this.members = response.data || [];
					}
				} catch (error) {
					console.error("Failed to load members:", error);
				} finally {
					this.isLoading = false;
				}
			}
		}
	};
}
</script>

<template>
	<div class="project-members">
		<div class="project-members__header">
			<xItem :configs="configsSearch" class="flex1" />
			<xBtn :configs="{
				label: '添加成员',
				preset: 'blue',
				icon: 'user-plus',
				onClick() {}
			}" />
		</div>

		<xBlock v-if="!isLoading && filteredMembers.length > 0" :bodyClass="{ height100: true, flex: 1 }">
			<xTableVir :columns="columns" :data="filteredMembers" />
		</xBlock>

		<div v-else-if="isLoading" class="project-members__loading">
			<xIcon icon="_loader-2" :size="24" class="project-members__loading-icon" />
			<span>加载中...</span>
		</div>

		<div v-else class="project-members__empty">
			<xIcon icon="users" :size="48" class="project-members__empty-icon" />
			<p>暂无成员</p>
			<p class="project-members__empty-hint">点击上方按钮添加成员</p>
		</div>
	</div>
</template>

<style lang="less">
.project-members {
	height: 100%;
	display: flex;
	flex-direction: column;

	&__header {
		display: flex;
		align-items: center;
		gap: 16px;
		margin-bottom: 16px;
	}

	&__loading,
	&__empty {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		color: var(--color-on-surface-variant);
	}

	&__loading-icon,
	&__empty-icon {
		margin-bottom: 12px;
		color: var(--color-on-surface-variant);
	}

	&__empty-hint {
		font-size: 12px;
		color: var(--color-on-surface-variant);
		margin-top: 4px;
	}

	&__role-badge {
		padding: 2px 8px;
		border-radius: 4px;
		font-size: 12px;
		background: var(--color-secondary-container);
		color: var(--color-on-secondary-container);
	}
}
</style>
