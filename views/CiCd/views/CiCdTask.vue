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
					<span class="mr4"> 任务列表 </span>
				</div>
			</template>
		</xPageTitle>
		<xPageContent>
			<xTablebar :configs="configsTable">
				<template #left>
					<xBtn :configs="configsAddTaskBtn" />
				</template>
			</xTablebar>
			<div class="mt flex1">
				<xTableVir :columns="configsTable.columns" :data="configsTable.data.list" />
			</div>
		</xPageContent>
	</div>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["APP"],
		setup() {
			const vm = this;
			const { cicd_id, git_repo_id } = this.$route.query;

			const task_run_output = "task_run_output";

			/* 当前页可以接受提示信息 */
			onMounted(async () => {
				await _.$ensure(() => vm.APP.WS);
				vm.APP.WS.on(task_run_output, ({ msg, task_id }) => {
					const index = _.findIndex(configsTable.data.list, row => row._id === task_id);
					if (index > -1) {
						configsTable.data.list[index].msg = msg;
						configsTable.data.list = [...configsTable.data.list];
					}
				});
			});

			onBeforeUnmount(async () => {
				await _.$ensure(() => vm.APP.WS);
				vm.APP.WS.off(task_run_output);
			});

			const configsTable = reactive(
				defTable({
					// isHideQuery: true,
					// isHideFilter: true,
					// isHidePagination: true,
					async onQuery(pagination = {}) {
						try {
							const { errcode, data } = await _api.xspace.apiCicdTaskList({
								cicd_id
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
						{ prop: "_id", label: "任务ID", width: 100 },
						{ prop: "task_name", label: "任务名称", width: 100 },
						{
							prop: "task_token",
							label: "webhook URL",
							cellRenderer({ rowData }) {
								const hookUrl = `${location.origin}/api/cicd/task_run?task_token=${rowData.task_token}&task_id=${rowData._id}`;
								return hDiv(
									{
										class: {
											pointer: true,
											ellipsis: true,
											flex: true,
											middle: true
										},
										// style: { "max-width": "200px" },
										onClick: () => {
											_.$copyToClipboard(hookUrl);
											_.$notify.info("复制成功");
										}
									},
									[
										hxIcon({ icon: "_copy", class: "mr" }),
										hDiv({ class: "flex1 ellipsis" }, hookUrl)
									]
								);
							}
						},
						defTable.colActions({
							width: 210,
							cellRenderer({ rowData }) {
								return hBtnWithMore({
									children: [
										{
											label: "编辑任务",
											async onClick() {
												_.$openModal({
													title: "编辑任务",
													parent: vm,
													url: "@/views/CiCd/views/CiCdTask.task.upsert.dialog.vue",
													cicd_id,
													git_repo_id,
													row: rowData,
													onSuccess() {
														_.$msg("更新成功");
														configsTable.onQuery();
													}
												});

												/* try { const res = await _api.xspace.apiCicdTaskRun({ task_id: rowData._id, cicd_id, task_token: rowData.task_token }); _.$msg("触发成功"); } catch (error) { _.$msgError(error); } */
											}
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

			async function addTask() {
				if (!cicd_id) {
					_.$msgError("当前流水线无效，请重新选择一个流水线");
				} else {
					_.$openModal({
						title: "添加任务",
						parent: vm,
						url: "@/views/CiCd/views/CiCdTask.task.upsert.dialog.vue",
						cicd_id,
						git_repo_id,
						onSuccess() {
							_.$msg("更新成功");
							configsTable.onQuery();
						}
					});

					/* try { const res = await _api.xspace.apiCicdTaskRun({ task_id: rowData._id, cicd_id, task_token: rowData.task_token }); _.$msg("触发成功"); } catch (error) { _.$msgError(error); } */
				}
			}
			return {
				configsTable,
				configsAddTaskBtn: {
					preset: "primary",
					label: "添加任务",
					onClick: addTask
				}
			};
		}
	});
}
</script>
