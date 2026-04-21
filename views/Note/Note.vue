<template>
	<section id="ViewNote" :class="cptNoteClass">
		<AppHeader v-if="cptIsShowAppHeaderComponent" />
		<div v-if="APP.isMobile" class="flex1-overflow-auto flex vertical">
			<xAdvancedSearch
				mountTo="#MobileMenu"
				v-model="isCollapse"
				:label="false"
				:mountProps="cptMountProps"
				:style="cptToggleStyle">
				<xGap t />
				<NoteAside class="width100 flex1 height100" />
			</xAdvancedSearch>
			<div id="MobileMenu"></div>
			<NoteSection style="width: 100%; height: 1px" v-show="isCollapse" />
		</div>
		<div v-else class="flex1-overflow-auto flex">
			<NoteAside v-show="!isShowEditor" />
			<NoteSection />
		</div>
	</section>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["APP"],
		components: {
			AppHeader: () => _.$importVue("@/views/Api/Header/ApiHeader.vue"),
			NoteAside: () => _.$importVue("@/views/Note/NoteAside.vue"),
			NoteSection: () => _.$importVue("@/views/Note/NoteSection.vue")
		},
		async mounted() {
			(() => {
				const TITLE_MAP = {
					private: "个人可见",
					all: "所有人可见"
				};
				const title = TITLE_MAP[this.cptBelongType];
				title && (document.title = `文档-${title}`);
			})();

			await this.update_wiki_menu_list();
			await this.updateCurrentWiki();
		},
		provide() {
			const inject_note = this;
			return {
				inject_note
			};
		},
		data() {
			const vm = this;
			vm.updateCurrentWiki = _.debounce(async function updateCurrentWiki(callback = false) {
				_.$loading(true);
				$(".flash-when").addClass("loading");
				try {
					if (!_.$isInput(vm.$route.query.wiki)) {
						return;
					}
					const res = await _api.xspace.wikiDetail({ _id: vm.$route.query.wiki });
					if (!res.errcode) {
						vm.currentWiki = res.data;
						callback && callback();
					}
				} catch (error) {
					console.error(error);
					_.$msgError(error);
				} finally {
					_.$loading(false);
					setTimeout(() => {
						$(".flash-when").removeClass("loading");
					}, 300);
				}
			}, 300);

			return {
				isCollapse: true,
				tree_data: [],
				currentWiki: {},
				expandedKeys: [],
				isShowEditor: false
			};
		},
		methods: {
			setCurrentWiki(data) {
				this.$router.push({
					path: this.$route.path,
					query: {
						...this.$route.query,
						wiki: data._id
					}
				});
			},
			async update_wiki_menu_list(search_params = {}) {
				_.$loading(true);
				try {
					let payload = {
						belong_type: this.cptBelongType,
						belong_id: this.cptBelongId,
						search_params
					};
					const { data } = await _api.xspace.wiki_menu(payload);
					const { list, orderArray } = data;
					this.tree_data = this.buildTree(list, orderArray);

					(() => {
						const wikiId = this.$route.query.wiki;

						if (wikiId) {
							const wiki = this.all_wiki[wikiId];
							if (wiki) {
								this.APP.routerUpsertQuery({ wiki: wikiId });
								return;
							}
						}

						if (this.tree_data[0]) {
							this.APP.routerUpsertQuery({ wiki: this.tree_data[0]._id });
							return;
						}

						this.APP.routerUpsertQuery({ wiki: "" });
					})();
				} catch (error) {
					console.error(error);
				} finally {
					_.$loading(false);
				}
			},
			buildTree(dataArray, orderArray) {
				console.time("buildTree");

				// 将数据数组转换为以_id为键的对象，便于查找
				this.all_wiki = _.reduce(
					dataArray,
					(target, item) => {
						target[item._id] = item;
						return target;
					},
					{}
				);

				// 构建树结构，为每个节点添加子节点
				const rootNodes = [];
				_.each(this.all_wiki, item => {
					if (!item) return;
					const parent = this.all_wiki[item.p_id];
					if (parent) {
						if (!_.isArray(parent.children)) {
							parent.children = [];
						}
						parent.children.push(item);
					} else {
						// 找不到父节点，视为根节点
						rootNodes.push(item);
					}
				});

				// 使用找到的根节点，而不是仅依赖p_id === 0
				let tree = rootNodes;

				// 检查orderArray是否有效且适用
				if (this.isValidOrderArray(tree, orderArray)) {
					tree = this.sortTreeByOrder(tree, orderArray);
				}

				console.timeEnd("buildTree");
				return tree;
			},

			// 验证orderArray有效性的方法
			isValidOrderArray(tree, orderArray) {
				// 检查是否为有效的非空数组
				if (!Array.isArray(orderArray) || orderArray.length === 0) {
					return false;
				}

				// 获取树中所有根节点的ID
				const treeIds = tree.map(item => item._id);

				// 检查长度是否匹配
				if (orderArray.length !== treeIds.length) {
					return false;
				}

				// 检查orderArray中的ID是否与树中根节点ID完全匹配
				const orderSet = new Set(orderArray);
				return treeIds.every(id => orderSet.has(id));
			},
			sortTreeByOrder(tree_data, orderArray = []) {
				tree_data = _.cloneDeep(tree_data);

				tree_data.sort((nowItem, nextItem) => {
					const nowIndex = orderArray.indexOf(nowItem._id);
					const nextIndex = orderArray.indexOf(nextItem._id);
					return nowIndex - nextIndex;
				});

				return _.map(tree_data, item => {
					if (_.$isArrayFill(item.children)) {
						item.children = this.sortTreeByOrder(item.children, orderArray);
					}
					return item;
				});
			}
		},
		computed: {
			cptToggleStyle() {
				return {
					position: "absolute",
					zIndex: 1,
					margin: "8px"
				};
			},
			cptMountProps() {
				return {
					class: "flex1 flex vertical",
					style: {
						height: "1px"
					}
				};
			},
			cptNoteClass() {
				return { "is-show-header": this.cptIsShowAppHeaderComponent };
			},
			cptIsShowAppHeaderComponent() {
				return ["private", "all"].includes(this.cptBelongType);
			},
			cptBelongType() {
				const { privateId, project_id, group_id } = this.$route.query;
				/* 有优先级和权重，顺序不可变 */
				if (privateId) return "private";
				if (project_id) return "project";
				if (group_id) return "group";
				return "all";
			},
			cptBelongId() {
				const { privateId, project_id, group_id } = this.$route.query;
				const variable_map = {
					private: privateId,
					project: project_id,
					group: group_id,
					all: "BELONG_ALL"
				};
				return variable_map[this.cptBelongType];
			},
			cptCurrentWiki() {
				return {
					...this.currentWiki,
					md: this.currentWiki.markdown || ""
				};
			}
		},
		watch: {
			cptCurrentWiki(wiki) {
				if (wiki.title) {
					document.title = wiki.title;
				}
			},
			"$route.query.wiki"() {
				this.updateCurrentWiki();
			}
		}
	});
}
</script>

<style lang="less">
#ViewNote {
	height: 100%;
	width: 100%;
	display: flex;
	flex-flow: row nowrap;

	&.is-show-header {
		flex-flow: column nowrap;
	}
}
</style>
