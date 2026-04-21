<template>
	<div id="ViewProject">
		<ProjectTabs key="ProjectTabs" @change-tab="changeTab" />
		<xAutoResizer :onResize="setSize" class="flex1 width1px">
			<ProjectDoc
				:style="wrapperStyle.transition_wrapper"
				key="ProjectDoc"
				v-if="isShow('文档')" />
			<ProjectInterface
				:style="wrapperStyle.transition_wrapper"
				key="ProjectInterface"
				v-if="isShow('接口')" />
			<ProjectSetting
				:style="wrapperStyle.transition_wrapper"
				key="ProjectSetting"
				v-if="isShow('设置')" />
			<ProjectCi
				:style="wrapperStyle.transition_wrapper"
				key="ProjectCi"
				v-if="isShow('CI')" />
			<ProjectHoppscotch
				:style="wrapperStyle.transition_wrapper"
				key="ProjectHoppscotch"
				v-if="isShow('测试')" />
		</xAutoResizer>
	</div>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["APP"],
		components: {
			ProjectTabs: () => _.$importVue("@/views/Api/Project/Tabs/ProjectTabs.vue"),
			ProjectDoc: () => _.$importVue("@/views/Api/Project/Tabs/ProjectDoc.vue"),
			ProjectInterface: () => _.$importVue("@/views/Api/Project/Tabs/ProjectInterface.vue"),
			ProjectSetting: () => _.$importVue("@/views/Api/Project/Tabs/ProjectSetting.vue"),
			ProjectCi: () => _.$importVue("@/views/Api/Project/Tabs/ProjectCi.vue"),
			ProjectHoppscotch: () => _.$importVue("@/views/Api/Project/Tabs/ProjectHoppscotch.vue")
		},
		provide() {
			return {
				inject_project: this
			};
		},
		mounted() {
			/* 获取接口信息 */
			this.get_interface_list();
		},
		setup() {
			const wrapperStyle = reactive({ transition_wrapper: {} });
			return {
				wrapperStyle,
				setSize({ width, height }) {
					wrapperStyle.transition_wrapper = {
						width: `${width}px`,
						height: `${height}px`
					};
				},

				addItem() {
					this.items.push({ id: this.nextId++ });
				},
				removeItem() {
					if (this.items.length > 0) {
						this.items.pop();
					}
				}
			};
		},
		data() {
			return {
				style: {},
				all_category: [],
				all_interface: [],
				all_tags: [],
				items: [],
				nextId: 1
			};
		},
		computed: {
			cptAsideTreeData() {
				return [
					{
						_id: Vue._xspace_var.ALL,
						title: i18n("全部接口"),
						menuType: Vue._xspace_var.ALL,
						list: []
					},
					...this.all_category
				];
			},
			cpt_tab_name: {
				get() {
					if (!this.$route.query?.project_tab_name) {
						this.cpt_tab_name = "接口";
					}
					return this.$route.query.project_tab_name;
				},
				set(project_tab_name) {
					if (project_tab_name) {
						this.$router.push({
							path: this.$route.path,
							query: {
								...this.$route.query,
								project_tab_name: project_tab_name
							}
						});
					}
				}
			}
		},
		methods: {
			changeTab(tab) {
				this.cpt_tab_name = tab.label;
			},
			isShow(tabName) {
				return this.cpt_tab_name === tabName;
			},

			async get_interface_list() {
				const vm = this;
				const { data } = await _api.xspace.apiInterfaceListMenu(this.APP.cptProjectId);
				if (data) {
					const all_category = data.map(category => {
						const children = _.map(category.list, i => {
							return {
								...i,
								menuType: "interface",
								categoryName: category.title,
								categoryId: i.catid
							};
						});
						return {
							...category,
							children,
							isCategory: true,
							categoryName: category.title,
							categoryId: category._id,
							menuType: "category",
							title: category.name,
							/* 下拉选项 */
							value: category._id,
							label: category.name
						};
					});

					vm.all_category = all_category;
					vm.all_interface = _.reduce(
						all_category,
						(dataSource, i) => {
							if (_.$isArrayFill(i.list)) {
								dataSource = dataSource.concat(i.list);
							}
							return dataSource;
						},
						[]
					);
					const _all_tags = _.reduce(
						vm.all_interface,
						(all_tags, i) => {
							return all_tags.concat(i.tag);
						},
						[]
					);
					vm.all_tags = _.uniqBy(_all_tags);
					return vm.all_category;
				}
			}
		}
	});
}
</script>
<style lang="less">
#ViewProject {
	height: 100%;
	display: flex;
	flex-flow: row nowrap;

	/* 淡入淡出动画相关样式 */
	.list-enter-active,
	.list-leave-active {
		transition: all 1s;
		transform: translateX(-100%);
	}
	.list-enter,
	.list-leave-to {
		opacity: 0;
		transform: translateX(100%);
	}
}
</style>
