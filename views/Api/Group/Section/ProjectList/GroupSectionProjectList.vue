<template>
	<div class="GroupSectionProjectList flex1 flex vertical">
		<xGap t="8" />
		<xTablebar :configs="configsTable">
			<template #left>
				<xBtnArray :configs="oprBtnArray" />
			</template>
			<template #right>
				<xAdvancedSearch
					mountTo="#AdvancedSearch"
					:collapse="isAdvancedSearchCollapse"
					@change="isAdvancedSearchCollapse = !isAdvancedSearchCollapse">
					<xBlock class="mt">
						<xForm col="3">
							<xItem :configs="formQuery.group" />
							<xItem :configs="formQuery.name" />
							<div class="flex end width100" span="full">
								<xBtn :configs="btnRest" class="ml4" />
								<xBtn :configs="btnQuery" class="ml4" />
							</div>
						</xForm>
					</xBlock>
					<template #collapse>
						<div class="flex middle mr8">
							<xItem :configs="formQuery.group" />
							<xItem :configs="formQuery.name" />
							<xGap r />
							<xBtn :configs="btnQuery" />
						</div>
					</template>
				</xAdvancedSearch>
			</template>
		</xTablebar>
		<div id="AdvancedSearch"></div>
		<div class="x-page-content-middle mt8">
			<xTableVir
				:columns="configsTable.columns"
				:data="cptConfigsTableDataList"
				:slotsRow="customRowRender"
				:rowHeight="rowHeight" />
		</div>
		<!-- <GroupSectionProjectListPrivate /> -->
		<!-- <GroupSectionProjectListNormal /> -->
		<!-- <xspacePlaceholderView view="GroupSectionProjectList" /> -->
	</div>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["APP", "GroupSection"],
		components: {
			GroupSectionProjectListPrivate: () =>
				_.$importVue(
					"@/views/Api/Group/Section/ProjectList/Private/GroupSectionProjectListPrivate.vue"
				),
			GroupSectionProjectListNormal: () =>
				_.$importVue(
					"@/views/Api/Group/Section/ProjectList/Normal/GroupSectionProjectListNormal.vue"
				)
		},
		mounted() {
			this.configsTable.onQuery();
		},
		data(vm) {
			return {
				rowHeight: 100,
				isAdvancedSearchCollapse: true,
				formQuery: defItems({
					name: {
						value:
							(_.$lStorage &&
								_.$lStorage.GroupSectionProjectListFormQuery &&
								_.$lStorage.GroupSectionProjectListFormQuery.name) ||
							"",
						resetValue: "",
						placeholder: i18n("分组、项目的名称或者ID"),
						clearable: true
					},
					group: {
						value:
							(_.$lStorage &&
								_.$lStorage.GroupSectionProjectListFormQuery &&
								_.$lStorage.GroupSectionProjectListFormQuery.group) ||
							[],
						resetValue: [],
						itemType: "xItemSelect",
						multiple: true,
						options: [],
						placeholder: i18n("分组"),
						once() {
							vm.$watch(
								() => [vm.APP.groupList],
								([groupList = []]) => {
									this.options = _.map(groupList, row => {
										return {
											label: row.group_name,
											value: row._id
										};
									});
								},
								{ immediate: true }
							);
						}
					}
				}),
				oprBtnArray: [
					{
						label: i18n("添加项目"),
						onClick() {
							return _.$openModal({
								title: i18n("添加项目"),
								url: "@/views/Api/Group/Section/ProjectList/GroupSectionProjectListAddProject.vue",
								parent: vm,
								onOk() {
									vm.configsTable.onQuery();
								}
							});
						}
					}
				],

				configsTable: defTable({
					isHideQuery: true,
					async onQuery() {
						_.$loading(true);
						try {
							let {
								data: { list, total }
							} = await _api.xspace.project_page({ page: 0, size: -1 });

							const group_at_least_one_project = new Set();

							list = _.reduce(
								list,
								(_list, row) => {
									const groupItem = _.find(vm.APP.groupList, {
										_id: row.group_id
									});
									if (groupItem) {
										group_at_least_one_project.add(groupItem._id);
										/* 全局的grouplist 拼接行数据 */
										_list.push({
											...row,
											group_desc: (groupItem && groupItem.group_desc) || "",
											group_name: groupItem?.group_name || ""
										});
									}
									return _list;
								},
								[]
							);
							list = _.reduce(
								vm.APP.groupList,
								(_list, groupItem) => {
									if (!group_at_least_one_project.has(groupItem._id)) {
										group_at_least_one_project.add(groupItem._id);

										_list.push({
											group_id: groupItem._id,
											group_desc: groupItem.group_desc || "",
											group_name: groupItem.group_name || ""
										});
									}
									return _list;
								},
								list
							);
							_.$setTableData(vm.configsTable, {
								list,
								total
							});
						} catch (error) {
							console.error(error);
						} finally {
							_.$loading(false);
						}
					},
					data: {
						set: new Set(),
						list: []
					},
					columns: [
						{
							prop: "group_id",
							label: "所属分组",
							cellRenderer({ rowData, rowIndex }) {
								return hDiv({ class: "flex vertical width100" }, [
									hDiv({ class: "flex middle width100" }, [
										hLink({
											label: `${rowData.group_name}`,
											// label: `${rowData.group_name}-${rowIndex}`,
											href: _.$aHashLink("/api/group", {
												group_id: rowData.group_id
											})
										}),
										hxIcon({
											key: rowIndex,
											icon: "tips",
											vIf: !!rowData.group_desc,
											directives: [
												hTipsHover({
													content: h("xMd", {
														md: rowData.group_desc,
														style: "max-width:600px"
													})
												})
											]
										}),
										h("xGap", { attrs: { r: true } }),
										hxIcon({
											icon: "edit",
											class: "pointer",
											async onClick() {
												return _.$openModal({
													title: i18n("编辑分组"),
													url: "@/views/Api/Group/Group.Upsert.vue",
													groupInfo: rowData,
													async onOk() {
														await vm.APP.updateGroupList();
														await vm.configsTable.onQuery();
													}
												});
											}
										})
									]),
									hDiv({ class: "data-list-id-number" }, [
										`${rowData._id || "--"}`
									])
								]);
							}
						},
						{
							prop: "name",
							label: "项目",
							cellRenderer({ rowData }) {
								if (!rowData._id) {
									return null;
								}
								return hDiv({ class: "flex vertical width100" }, [
									hDiv({ class: "flex middle" }, [
										hxIcon({
											key: rowData._id,
											class: "mr4",
											style: "width: 32px; height: 32px;",
											img: _common_utils.avatar_url(rowData._id, "project")
										}),
										hLink({
											class: "flex1",
											label: `${rowData.name}`,
											href: _.$aHashLink("/api/project", {
												project_id: rowData._id,
												group_id: rowData.group_id
											})
										})
									]),
									hDiv({ class: "data-list-id-number" }, [`${rowData._id}`])
								]);
							}
						},
						{
							prop: "desc",
							label: "描述",
							cellRenderer({ rowData }) {
								return h("xMd", {
									class: "height100 overflow-auto",
									md: rowData.desc
								});
							}
						}
					]
				})
			};
		},
		computed: {
			cptFormQuery() {
				return _.$pickFormValues(this.formQuery);
			},
			cptConfigsTableDataList() {
				const rowArray = _.filter(this.configsTable.data.list, row => {
					const { name, group } = this.cptFormQuery;
					_.$lStorage.GroupSectionProjectListFormQuery = { name, group };
					if (name) {
						const regexp = val => new RegExp(val, "ig");
						return _.some([row._id, row.name, row.group_id, row.group_name], val =>
							regexp(name).test(val || "")
						);
					} else if (!_.isEmpty(group)) {
						return _.includes(group, row.group_id);
					} else {
						return true;
					}
				});
				return xTableVirMergeData({
					rowArray,
					GroupPropArray: ["group_id"]
				});
			},
			btnRest() {
				const vm = this;
				return {
					label: "重置",
					onClick() {
						_.$resetFormValues(`#AdvancedSearch`);
					}
				};
			},
			btnQuery() {
				const vm = this;
				return {
					label: "查询",
					preset: "primary",
					onClick() {
						vm.configsTable.onQuery();
					}
				};
			},
			isShow() {
				return this.$route.query.group_view_tab_name === Vue._xspace_var.TAB_KEY_PROJECT_LIST;
			}
		},
		methods: {
			customRowRender({ cells, columns, depth, isScrolling, rowData, rowIndex, style }) {
				const { rowHeight } = this;
				return xTableVirCells({
					columns,
					cells,
					rowData,
					rowHeight,
					GroupPropArray: ["group_id"],
					setStyle({ rowSpan }) {
						return {
							backgroundColor: "#f8f8f8",
							border: "var(--el-table-border)",
							height: `${rowSpan * rowHeight}px`,
							alignSelf: "flex-start",
							zIndex: rowSpan
						};
					}
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

<style lang="less">
.GroupSectionProjectList {
	width: 100%;

	.GroupSectionProjectList-header {
		background: #eee;
		height: 64px;
		line-height: 40px;
		border-radius: var(--border-radius);
		text-align: right;
		padding: 0 10px;
		font-weight: bold;
		margin-bottom: 15px;
		display: flex;
		align-items: center;
		color: var(--color-title);
		font-weight: 500;
	}
}
</style>
