<style lang="less">
.NoteAside {
	background-color: white;
	.wiki-tree-wrapper {
		.aside-tree-item {
			width: 1px;
		}
		.is-current {
			.el-tree-node__content {
				color: var(--el-color-primary);
				background: #ecf5ff;
				border-color: #b3d8ff;
			}
		}

		.el-tree-node__content {
			.xIcon._add {
				display: none;
			}
			&:hover {
				.xIcon._add {
					display: block;
				}
			}
		}
	}
}
</style>

<template>
	<aside class="NoteAside x-sider_wrapper" :style="stateStyle">
		<div class="x-sider_wrapper_tree flex vertical">
			<div class="group-operate flex start middle mb10 left-tree box-shadow">
				<xItem :configs="configsSearch" class="flex1" />
				<xGap l="10" />
				<xIcon icon="currentLocation" class="icon-opreation_click pointer" />
				<!-- 添加分组 -->
				<xIcon
					icon="_add"
					class="icon-opreation_click pointer ml10"
					@click="addNewWiki()" />
			</div>
			<div class="flex1-overflow-auto wiki-tree-wrapper" ref="refTreeScroll">
				<xTree
					ref="refTree"
					:contentRender="treeContentRender"
					:data="inject_note.tree_data"
					:expandedKeys.sync="inject_note.expandedKeys"
					:props="treeProps"
					:dragAndDrop="handleDragAndDrop" />
			</div>
		</div>
		<div class="resize_bar" icon="scroll" v-xmove="resizeOptions" />
	</aside>
</template>
<script lang="ts">
export default async function () {
	const { useXmove } = await _.$importVue("@/utils/hooks.vue");

	const getTreeOrder = (treeData, orderArray = []) => {
		treeData = _.cloneDeep(treeData);
		let item;
		while ((item = treeData.shift())) {
			orderArray.push(item._id);
			if (_.$isArrayFill(item.children)) {
				treeData.unshift(...item.children);
			}
		}
		return orderArray;
	};

	const getParents = (id, allNodeObject, parents = []) => {
		const self = allNodeObject[id];
		if (self) {
			parents.push(self);
			if (self.p_id) {
				return getParents(self.p_id, allNodeObject, parents);
			}
		}
		return parents;
	};

	/* 分组信息 */
	return defineComponent({
		inject: ["APP", "inject_note"],
		mounted() {
			this.initExpandedKeys();
		},
		setup(props) {
			const { stateStyle, resizeOptions } = (() => {
				if (this.APP.isMobile) {
					return { stateStyle: {}, resizeOptions: {} };
				} else {
					return useXmove(props);
				}
			})();

			return {
				stateStyle,
				resizeOptions,
				treeProps: {
					value: "_id",
					label: "title",
					children: "children"
				}
			};
		},
		computed: {},
		watch: {
			"inject_note.currentWiki": {
				handler(wiki) {
					if (wiki?._id) {
						this.$refs.refTree?.setCurrentKey?.(wiki._id);
					}
				},
				immediate: true
			}
		},
		data() {
			const vm = this;

			return {
				configsSearch: defItem({
					isSearch: false,
					value: "",
					placeholder: "搜索文档",
					onEnter() {
						return vm.inject_note.update_wiki_menu_list({
							content: vm.configsSearch.value
						});
					},
					itemSlots: {
						afterController() {
							return hxBtn({
								configs: {
									class: "ml4",
									label: "查询",
									preset: "primary",
									onClick() {
										return vm.configsSearch.onEnter();
									}
								}
							});
						}
					},
					clearable: true
				}),
				groupListForShow: []
			};
		},
		methods: {
			async initExpandedKeys() {
				let id = await _.$ensure(() => this.inject_note?.currentWiki?._id);
				const ALL_NODE_MAP = new Map();
				_.$traverse(this.inject_note.tree_data, node => {
					ALL_NODE_MAP.set(node._id, node);
				});
				const expandedKeys = [];
				let node;
				while ((node = ALL_NODE_MAP.get(id))) {
					if (node._id) {
						expandedKeys.push(node._id);
					}
					if (node.p_id > -1) {
						if (id === node.p_id) {
							break;
						}
						id = node.p_id;
					}
				}
				this.inject_note.expandedKeys = expandedKeys;
			},
			scrollToLocation() {},
			async handleDragAndDrop({ drag, drop, type: dropType }) {
				const { inject_note } = this;
				const dragItem = { ...drag.data };
				const dropItem = { ...drop.data };
				if (dragItem._id == dropItem._id) {
					return;
				}
				const menuOrderArray = getTreeOrder(inject_note.tree_data);
				const dragIndex = menuOrderArray.indexOf(dragItem._id);
				/* drag不能放在drap的子级 */
				/* 获取drop的所有父级 */
				const parents = getParents(dropItem._id, inject_note.all_wiki);
				/* drag不在drop的祖级上 */
				if (_.find(parents, parent => parent._id === dragItem._id)) {
					_.$msgError("不能把父级拖到子级上");
					return;
				}
				if (dropType === "inner") {
					dragItem.p_id = dropItem._id;
				} else if (dropType === "after") {
					dragItem.p_id = dropItem.p_id;
					menuOrderArray.splice(dragIndex, 1);
					const dropIndex = menuOrderArray.indexOf(dropItem._id);
					menuOrderArray.splice(dropIndex + 1, 0, dragItem._id);
				} else if (dropType === "before") {
					dragItem.p_id = dropItem.p_id;
					menuOrderArray.splice(dragIndex, 1);
					let dropIndex = menuOrderArray.indexOf(dropItem._id);
					if (dropIndex === 0) {
						menuOrderArray.unshift(dragItem._id);
					} else {
						menuOrderArray.splice(dropIndex - 1, 0, dragItem._id);
					}
				}

				try {
					await _api.xspace.wiki_upsert_one(dragItem);
					await _api.xspace.wikiResetMenuOrder({
						order: menuOrderArray,
						belong_type: inject_note.cptBelongType,
						belong_id: (() => {
							// const { group_id, project_id } = cptRouter.value.query;
							// const s_map = {
							// 	group: group_id,
							// 	project: project_id
							// };
							return /* s_map[inject_note.belongType] */ "";
						})()
					});
					await inject_note.update_wiki_menu_list();
					_.$msg(i18n("更新成功"));
				} catch (error) {
					_.$msgError(error.message);
				}
			},
			async addNewWiki(payload) {
				const { node } = payload || {};

				return _.$openModal({
					parent: this,
					title: "新增",
					url: "@/views/Note/Note.dialog.insert.vue",
					parentDocId: node?._id || "",
					belong_type: this.inject_note.cptBelongType,
					belong_id: this.inject_note.cptBelongId
				});
			},
			/* 菜单 */
			treeContentRender({ node, data, injectRootTree }) {
				const vm = this;

				const vNodeTitleLabel = h(
					"div",
					{
						staticClass: "node-name pointer flex1 ellipsis",
						onClick(...args) {
							vm.inject_note.setCurrentWiki(data);
						}
					},
					[data.title]
				);
				const vNodeGap = h("xGap", { attrs: { f: "" } });
				const vNodeIconAddSubwiki = h(
					"xIcon",
					{
						staticClass: "ml",
						icon: "_add",
						onClick(event) {
							event.stopPropagation();
							event.preventDefault();
							vm.addNewWiki({ node: data });
						}
					},
					[data.title]
				);

				return h(
					"div",
					{
						staticClass: "aside-tree-item flex middle flex1",
						attrs: {
							"data-wiki-id": data._id
						}
					},
					[vNodeTitleLabel, vNodeIconAddSubwiki]
				);
			}
		}
	});
}
</script>
