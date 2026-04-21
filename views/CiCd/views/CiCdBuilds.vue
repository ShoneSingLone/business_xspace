<style lang="less">
#CiCdBuilds {
	--xPageTitle-padding: var(--ui-one);
}

.CiCdBuilds {
	.log-container {
		height: 100%;
		overflow-y: auto;
		font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
		background-color: #f9f9f9;
		border-radius: 8px;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
	}

	.log-item {
		position: relative;
		margin: 12px 0;
		padding: 12px 15px 12px 35px;
		background: linear-gradient(135deg, #ffffff 0%, #f5f7fa 100%);
		border-left: 3px solid #4a90e2;
		border-radius: 0 4px 4px 0;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
		transition: all 0.3s ease;
	}

	.log-item:hover {
		transform: translateX(2px);
		box-shadow: 0 3px 8px rgba(0, 0, 0, 0.12);
	}

	.log-item::before {
		content: "";
		position: absolute;
		left: -8px;
		top: 50%;
		transform: translateY(-50%);
		width: 13px;
		height: 13px;
		border-radius: 50%;
		background-color: #4a90e2;
		box-shadow:
			0 0 0 2px #fff,
			0 0 0 4px #4a90e2;
	}

	.log-item:nth-child(odd)::before {
		background-color: #50e3c2;
		box-shadow:
			0 0 0 2px #fff,
			0 0 0 4px #50e3c2;
	}

	.log-item:nth-child(odd) {
		border-left-color: #50e3c2;
	}

	.log-item:last-child::after {
		content: "";
		position: absolute;
		left: -6.5px;
		top: calc(100% - 5px);
		height: calc(100% + 20px);
		width: 2px;
		background-color: #e0e0e0;
		z-index: 0;
	}

	.log-timestamp {
		font-size: 0.85em;
		color: #888;
		margin-right: 8px;
	}

	.log-message {
		color: #333;
		line-height: 1.4;
	}
}
</style>
<template>
	<div id="CiCdBuilds" class="x-page-view flex1">
		<xDrawer title="作业日志" :visible.sync="state.isShowDrawer" class="CiCdBuilds">
			<div class="log-container x-padding">
				<div class="log-item" v-for="([time, msg], index) in state.logArray" :key="index">
					<span class="log-timestamp">{{ time }}</span>
					<span class="log-message">{{ msg }}</span>
				</div>
			</div>
		</xDrawer>

		<xPageTitle>
			<template #title>
				<div class="flex middle">
					<span class="mr4"> 作业列表 </span>
				</div>
			</template>
		</xPageTitle>
		<xPageContent>
			<xTablebar :configs="configsTable" />
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
			const state = reactive({
				logArray: [],
				isShowDrawer: false
			});
			const task_run_output = "task_run_output";

			/* 当前页可以接受提示信息 */
			onMounted(async () => {
				await _.$ensure(() => vm.APP.WS);
				vm.APP.WS.on(task_run_output, ({ msg, task_id }) => {
					state.logArray.push([_.$dateFormat(), msg]);
					state.isShowDrawer = true;
					queryJobs();
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
					async onQuery() {
						try {
							const {
								data: { list }
							} = await _api.xspace.apiCicdJobList({
								cicd_id
							});
							_.$setTableData(configsTable, { list });
						} catch (error) {
							_.$msgError(error);
						}
					},
					data: {
						set: new Set(),
						list: []
					},
					columns: [
						{ prop: "_id", label: "作业ID", width: 100 },
						{ prop: "task_status", label: "作业状态", width: 100 },
						{
							prop: "resource",
							label: "产出物",
							width: 100,
							cellRenderer({ cellData: resource, rowData }) {
								if (!resource?.name) {
									return resource;
								}
								const file_url = Vue._common_utils.appendToken(
									`${window._AJAX_URL_PREFIX || ""}/api/resource/get?id=${
										resource._id
									}`
								);

								return hDiv([
									h(
										"a",
										{
											class: "flex1 ellipsis flex middle",
											attrs: {
												href: file_url,
												download: resource.name.replace(
													`${rowData.commit_hash}_`,
													""
												),
												target: "_blank"
											}
										},
										[hxIcon({ icon: "download", class: "mr" }), "📦"]
									)
								]);
							}
						},
						{ prop: "task_ref", label: "触发分支" },
						{ prop: "commit_hash", label: "commit hash", width: 200 },
						{ prop: "message", label: "commit message" },
						{
							prop: "last_time",
							label: "提交时间",
							width: 150,
							cellRenderer({ cellData }) {
								return _.$dateFormat(cellData);
							}
						},
						defTable.colActions({
							width: 100,
							cellRenderer({ rowData }) {
								return hBtnWithMore({
									children: [
										{
											label: "日志",
											isHide: !rowData.task_log,
											async onClick() {
												_.$openModal({
													title: "作业日志",
													parent: vm,
													url: "@/views/CiCd/views/CiCdBuilds.jobs_log.dialog.vue",
													logString: rowData.task_log
												});
											}
										},
										{
											icon: "refresh",
											// isHide: rowData.task_status === "running",
											async onClick() {
												try {
													const {
														data: {
															list: [task]
														}
													} = await _api.xspace.apiCicdTaskList({
														task_id: rowData.task_id
													});
													const hookUrl = `${location.origin}/api/cicd/task_run?task_token=${task.task_token}&task_id=${task._id}`;
													await _.$ajax.post(hookUrl, {
														data: _.merge(rowData, {
															ref: rowData.task_ref,
															after: rowData.commit_hash
														})
													});
													configsTable.onQuery();
													_.$msgSuccess("任务已启动");
												} catch (error) {
													_.$msgError(error);
												}
											}
										}
									]
								});
							}
						})
					]
				})
			);

			const queryJobs = _.debounce(() => {
				configsTable.onQuery();
			}, 1000 * 3);

			onMounted(configsTable.onQuery);

			return {
				configsTable,
				state
			};
		}
	});
}
</script>
