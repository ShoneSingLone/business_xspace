<style lang="less">
#ProjectCi {
	--xPageTitle-padding: var(--ui-one);
}
</style>
<template>
	<div id="ProjectCi" class="x-page-view flex1">
		<xPageTitle>
			<template #title>
				<div class="flex middle">
					<span class="mr4"> CI/CD </span>
					<xPopover placement="top-start" width="600" trigger="hover">
						<xMd :md="mdString" />
						<xIcon icon="tips" slot="reference" />
					</xPopover>
				</div>
			</template>
		</xPageTitle>
		<xPageContent>
			<xTablebar :configs="configsTable">
				<template #left>
					<xBtn :configs="configsAddNewGitrepo" />
				</template>
			</xTablebar>
			<div class="mt" style="height: 500px">
				<xTableVir :columns="configsTable.columns" :data="configsTable.data.list" />
			</div>
		</xPageContent>
	</div>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["APP", "inject_project"],
		setup() {
			const vm = this;

			const configsTable = reactive(
				defTable({
					// isHideQuery: true,
					// isHideFilter: true,
					// isHidePagination: true,
					async onQuery(pagination = {}) {
						try {
							const { errcode, data } = await _api.xspace.apiCicdGitAddressList({
								project_id: vm.APP.cptProjectId
							});
							if (_.$isSame(errcode, 0)) {
								_.$setTableData(configsTable, { list: data.list });
							}
						} catch (error) {}
					},
					data: {
						set: new Set(),
						list: []
					},
					columns: [
						{ prop: "alias", label: "别名" },
						{ prop: "git_address", label: "Git地址" },
						{ prop: "status", label: "仓库状态" },
						defTable.colActions({
							width: 210,
							cellRenderer({ rowData: gitRepoRow }) {
								const isUnset = gitRepoRow.status === "unset";
								const isDone = gitRepoRow.status === "done";
								return hBtnWithMore({
									children: [
										{
											label: "初始化仓库",
											isHide: !isUnset,
											async onClick() {
												return _.$openModal({
													parent: vm,
													title: "CloneRepo",
													url: "@/views/Api/Project/Tabs/ProjectCi/ProjectCi.GitRepo.Clone.dialog.vue",
													row: gitRepoRow,
													isHideHeader: true,
													onSuccess() {
														configsTable.onQuery();
													}
												});
											}
										},
										{
											label: () =>
												h(
													"a",
													{
														attrs: {
															href: _.$aHashLink("/cicd", {
																cicd_id: gitRepoRow.cicd_id,
																git_repo_id: gitRepoRow.git_repo_id
															}),
															target: "_blank"
														}
													},
													hxBtn({ preset: "text" }, "CI/CD 管理")
												),
											isHide: !isDone
										}
									]
								});
							}
						})
					]
				})
			);
			onMounted(() => {
				configsTable.onQuery();
			});
			return {
				test() {
					return _.$sleep(1000 * 500);
				},
				configsTable,
				configsAddNewGitrepo: {
					label: "添加代码仓库",
					async onClick() {
						_.$openModal({
							project_id: vm.APP.cptProjectId,
							title: "添加代码仓库",
							url: "@/views/Api/Project/Tabs/ProjectCi/ProjectCi.GitRepo.Add.dialog.vue",
							onSuccess() {
								configsTable.onQuery();
							}
						});
					}
				},
				mdString: `1.添加代码仓库
- 可以有多个代码仓库，可以是不同域名仓库，但是要保证\`username\`和\`password\`有效
- 用户名和密码只在后端运行脚本时使用，不会再次出现在前端
2.代码仓库状态分为：
- \`done\`(已下载到服务器)
- \`unset\`(未下载到服务器)
- \`initializing\`(正在初始化)
3.\`unset\`需要手动触发Clone操作
                `
			};
		}
	});
}
</script>
