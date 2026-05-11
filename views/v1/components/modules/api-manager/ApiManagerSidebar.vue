<script lang="ts">
export default async function () {
	return {
		props: {
			apiData: {
				type: Array,
				default: () => []
			},
			expandedFolders: {
				type: Array,
				default: () => ["personal_space", "groups"]
			},
			activeNodeId: {
				type: String,
				default: "personal_space"
			},
			isLoading: {
				type: Boolean,
				default: false
			},
			error: {
				type: String,
				default: null
			}
		},
		methods: {
			toggleFolder(folderId) {
				this.$emit("toggle-folder", folderId);
			},
			openNode(node) {
				this.$emit("open-node", node);
			},
			showContextMenu(node, event) {
				console.log("🖱️ ApiManagerSidebar: showContextMenu called, node:", node?.name, "event type:", event.type);
				console.log("🖱️ Event clientX:", event.clientX, "clientY:", event.clientY);
				event.preventDefault();
				event.stopPropagation();
				this.$emit("context-menu", node, event);
			},
			createProject() {
				this.$emit("create-project");
			},
			toggleStarProject(project) {
				this.$emit("toggle-star", project);
			},
			retryLoad() {
				this.$emit("retry");
			},
			isFolderType(type) {
				return [
					"personal",
					"group",
					"project",
					"api_folder",
					"doc_folder",
					"folder"
				].includes(type);
			},
			isProjectFolder(node) {
				return node.id === "ps_my_projects" || node.id === "ps_followed" || node.id?.endsWith("_projects");
			},
			getIcon(type) {
				switch (type) {
					case "personal": return "user";
					case "group": return "folder";
					case "project": return "folder";
					case "api_folder": return "folder";
					case "doc_folder": return "folder";
					case "folder": return "folder";
					case "api": return "code";
					case "doc": return "file-text";
					case "code": return "code";
					case "member_list": return "_member_list";
					case "setting": return "_configs";
					case "cicd": return "activity";
					case "log": return "_article";
					default: return "file";
				}
			},
			getIconColor(type, node) {
				switch (type) {
					case "personal": return "api-manager__icon api-manager__icon--personal";
					case "group": return "api-manager__icon api-manager__icon--group";
					case "project":
						if (node && node.path) {
							if (node.path.includes("/个人空间/我的项目/")) {
								return "api-manager__icon api-manager__icon--project-personal";
							} else if (node.path.includes("/个人空间/星标项目/")) {
								return "api-manager__icon api-manager__icon--project-starred";
							} else if (node.path.includes("/群组/")) {
								return "api-manager__icon api-manager__icon--project-group";
							}
						}
						return "api-manager__icon api-manager__icon--project";
					case "api_folder": return "api-manager__icon api-manager__icon--api-folder";
					case "doc_folder": return "api-manager__icon api-manager__icon--doc-folder";
					case "folder": return "api-manager__icon api-manager__icon--folder";
					case "api": return "api-manager__icon api-manager__icon--api";
					case "doc": return "api-manager__icon api-manager__icon--doc";
					case "code": return "api-manager__icon api-manager__icon--code";
					case "member_list": return "api-manager__icon api-manager__icon--member-list";
					case "setting": return "api-manager__icon api-manager__icon--setting";
					case "cicd": return "api-manager__icon api-manager__icon--cicd";
					case "log": return "api-manager__icon api-manager__icon--log";
					default: return "api-manager__icon api-manager__icon--default";
				}
			},
			getTreeItemClass(node) {
				return {
					'api-manager__tree-item': true,
					'api-manager__tree-item--active': this.activeNodeId === node.id
				};
			},
			getStarIconClass(node) {
				return node.followed ? 'api-manager__star-icon--active' : '';
			}
		}
	};
}
</script>

<template>
	<div class="api-manager__sidebar">
		<div class="api-manager__sidebar-inner">
			<div class="api-manager__tree">
				<div v-if="isLoading" class="api-manager__loading">
					<xIcon icon="_loader-2" :size="24" class="api-manager__loading-icon" />
					<span>加载中...</span>
				</div>

				<div v-else-if="error" class="api-manager__error">
					<xIcon icon="alert-circle" :size="24" class="api-manager__error-icon" />
					<span>{{ error }}</span>
					<button @click="retryLoad" class="api-manager__retry-btn">重试</button>
				</div>

				<template v-else>
					<div v-for="rootItem in apiData" :key="rootItem.id">
						<div 
							:class="getTreeItemClass(rootItem)"
							style="padding-left: 8px" 
							@click.stop="openNode(rootItem)"
							@contextmenu="showContextMenu(rootItem, $event)">
							<span class="api-manager__tree-toggle">
								<div v-if="isFolderType(rootItem.type) && rootItem.children?.length" @click.stop="toggleFolder(rootItem.id)" style="cursor: pointer; padding: 4px;">
									<xIcon icon="arrow-down" v-if="expandedFolders.indexOf(rootItem.id) > -1" :size="14" />
									<xIcon icon="arrow-right" v-else :size="14" />
								</div>
							</span>
							<span class="api-manager__tree-icon">
								<xIcon :icon="getIcon(rootItem.type)" :size="16" :class="getIconColor(rootItem.type)" />
							</span>
							<span class="api-manager__tree-label">{{ rootItem.name }}</span>
						</div>

						<div v-if="expandedFolders.indexOf(rootItem.id) > -1 && rootItem.children">
							<div :key="child.id" v-for="child in rootItem.children">
								<div 
									:class="getTreeItemClass(child)"
									style="padding-left: 20px" 
									@click.stop="openNode(child)"
									@contextmenu="showContextMenu(child, $event)">
									<span class="api-manager__tree-toggle">
										<template v-if="isFolderType(child.type) && child.children?.length && !isProjectFolder(child)">
											<div @click.stop="toggleFolder(child.id)" style="cursor: pointer; padding: 4px;">
												<xIcon icon="arrow-down" v-if="expandedFolders.indexOf(child.id) > -1" :size="14" />
												<xIcon icon="arrow-right" v-else :size="14" />
											</div>
										</template>
									</span>
									<span class="api-manager__tree-icon">
										<xIcon :icon="getIcon(child.type)" :size="16" :class="getIconColor(child.type, child)" />
									</span>
									<span class="api-manager__tree-label">
										{{ child.name }}
										<span v-if="child.type === 'project' && child.path?.includes('/个人空间/我的项目/')" 
											class="api-manager__project-badge api-manager__project-badge--personal" 
											title="个人项目">个人</span>
										<span v-if="child.type === 'project' && child.path?.includes('/个人空间/星标项目/')" 
											class="api-manager__project-badge api-manager__project-badge--starred" 
											title="星标项目">星标</span>
										<span v-if="child.type === 'project' && child.path?.includes('/群组/')" 
											class="api-manager__project-badge api-manager__project-badge--group" 
											title="群组项目">群组</span>
									</span>
									<span v-if="child.id === 'ps_my_projects'" class="api-manager__tree-actions">
										<button @click.stop="createProject" class="api-manager__tree-action-btn" title="创建项目">
											<xIcon icon="plus" :size="14" />
										</button>
									</span>
									<span v-if="child.type === 'project'" class="api-manager__tree-actions">
										<button @click.stop="toggleStarProject(child)" class="api-manager__tree-action-btn"
											:title="child.followed ? '取消星标' : '星标'">
											<xIcon :icon="child.followed ? 'star-filled' : 'star'" :size="14" 
												:class="getStarIconClass(child)" />
										</button>
									</span>
								</div>

								<div v-if="expandedFolders.indexOf(child.id) > -1 && child.children">
									<div :key="subchild.id" v-for="subchild in child.children">
										<div 
										:class="getTreeItemClass(subchild)"
										style="padding-left: 32px" 
										@click.stop="openNode(subchild)"
										@contextmenu="showContextMenu(subchild, $event)">
											<span class="api-manager__tree-toggle">
												<template v-if="isFolderType(subchild.type) && subchild.children?.length && !isProjectFolder(subchild)">
													<div @click.stop="toggleFolder(subchild.id)" style="cursor: pointer; padding: 4px;">
														<xIcon icon="arrow-down" v-if="expandedFolders.indexOf(subchild.id) > -1" :size="14" />
														<xIcon icon="arrow-right" v-else :size="14" />
													</div>
												</template>
											</span>
											<span class="api-manager__tree-icon">
												<xIcon :icon="getIcon(subchild.type)" :size="16" :class="getIconColor(subchild.type)" />
											</span>
											<span class="api-manager__tree-label">{{ subchild.name }}</span>
										</div>

										<div v-if="expandedFolders.indexOf(subchild.id) > -1 && subchild.children">
											<div :key="subsubchild.id" v-for="subsubchild in subchild.children">
												<div 
													:class="getTreeItemClass(subsubchild)"
													style="padding-left: 44px" 
													@click.stop="openNode(subsubchild)"
													@contextmenu="showContextMenu(subsubchild, $event)">
													<span class="api-manager__tree-toggle">
														<template v-if="isFolderType(subsubchild.type) && subsubchild.children?.length">
															<div @click.stop="toggleFolder(subsubchild.id)" style="cursor: pointer; padding: 4px;">
																<xIcon icon="arrow-down" v-if="expandedFolders.indexOf(subsubchild.id) > -1" :size="14" />
																<xIcon icon="arrow-right" v-else :size="14" />
															</div>
														</template>
													</span>
													<span class="api-manager__tree-icon">
														<xIcon :icon="getIcon(subsubchild.type)" :size="16" :class="getIconColor(subsubchild.type)" />
													</span>
													<span class="api-manager__tree-label">{{ subsubchild.name }}</span>
												</div>

												<div v-if="expandedFolders.indexOf(subsubchild.id) > -1 && subsubchild.children">
													<div :key="leaf.id" v-for="leaf in subsubchild.children">
														<div 
															:class="getTreeItemClass(leaf)"
															style="padding-left: 56px" 
															@click.stop="openNode(leaf)"
															@contextmenu="showContextMenu(leaf, $event)">
															<span class="api-manager__tree-toggle api-manager__tree-toggle--spacer" aria-hidden="true"></span>
															<span class="api-manager__tree-icon">
																<xIcon :icon="getIcon(leaf.type)" :size="16" :class="getIconColor(leaf.type)" />
															</span>
															<span class="api-manager__tree-label">{{ leaf.name }}</span>
														</div>
													</div>
												</div>
											</div>
										</div>
									</div>
								</div>
							</div>
						</div>
					</div>
				</template>
			</div>
		</div>
	</div>
</template>

<style lang="less">
.api-manager__sidebar-header {
	position: sticky;
	top: 0;
	z-index: 10;
	background: var(--color-surface-container);
	border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
}

.api-manager__sidebar-title {
	padding: 12px 8px;
	margin: 0;
	font-size: 12px;
	font-weight: 700;
	letter-spacing: 0.08em;
	text-transform: uppercase;
	color: var(--color-on-surface-variant);
}

.api-manager__tree-toggle {
	margin-right: 4px;
	
	&--spacer {
		width: 14px;
		display: inline-block;
	}
}
</style>