<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
	const api = await _.$importVue("@/utils/api.vue");
	const utils = await _.$importVue("@/utils/api-manager.vue");

	return {
		components: {
			ApiManagerToolbar: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerToolbar.vue"),
			ApiManagerSidebar: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerSidebar.vue"),
			ApiManagerBreadcrumb: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerBreadcrumb.vue"),
			ApiManagerFileList: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerFileList.vue"),
			ApiManagerFileGrid: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerFileGrid.vue"),
			ApiManagerEditor: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerEditor.vue"),
			ApiManagerPreview: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerPreview.vue"),
			ApiManagerContextMenu: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerContextMenu.vue"),
			ApiManagerCreateProjectDialog: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerCreateProjectDialog.vue")
		},
		props: {
			windowData: Object
		},
		inject: {
			system: {
				default: null
			}
		},
		data() {
			return {
				apiData: null,
				environments: ["Development", "Test", "Staging", "Production"],
				activeEnvironment: "Development",
				selectedFile: null,
				searchQuery: "",
				showPreview: true,
				viewMode: "list",
				expandedFolders: utils.loadExpandedFolders(),
				selectedNodeId: utils.loadSelectedNodeId(),
				isLoading: false,
				error: null,
				sortField: "name",
				sortDirection: "asc",
				treeSearchQuery: "",
				isRequesting: false,
				responseData: null,
				responseStatus: null,
				responseTime: null,
				editingContent: null,
				contextMenu: { show: false, x: 0, y: 0, node: null },
				showCreateProjectDialog: false,
				creatingProject: false
			};
		},
		computed: {
			activeNode() {
				if (this.windowData) return this.windowData;
				return this.apiData?.find(item => item.id === this.selectedNodeId) || 
					{ id: "personal_space", name: "个人空间", type: "personal", path: "/个人空间", children: [] };
			},
			filteredAndSortedFiles() {
				if (!utils.isFolderType(this.activeNode?.type)) return [];
				let files = this.activeNode?.children || [];
				if (this.searchQuery) {
					const lowerQuery = this.searchQuery.toLowerCase();
					files = files.filter(f => f.name.toLowerCase().includes(lowerQuery));
				}
				return [...files].sort((a, b) => {
					let comparison = 0;
					if ((this.sortField === "name" || this.sortField === "type") &&
						utils.isFolderType(a.type) !== utils.isFolderType(b.type)) {
						return utils.isFolderType(a.type) ? -1 : 1;
					}
					switch (this.sortField) {
						case "name": comparison = a.name.localeCompare(b.name); break;
						case "updatedAt": comparison = new Date(a.updatedAt).getTime() - new Date(b.updatedAt).getTime(); break;
						case "type": comparison = a.type.localeCompare(b.type); break;
					}
					return this.sortDirection === "asc" ? comparison : -comparison;
				});
			},
			canNavigateUp() {
				return this.activeNode?.id !== "personal_space" && this.activeNode?.id !== "groups";
			}
		},
		mounted() {
			this.loadApiData();
			if (this.windowData) this.expandAncestors(this.windowData.id);
		},
		watch: {
			windowData: {
				handler(newVal) {
					if (newVal) this.expandAncestors(newVal.id);
				},
				immediate: false
			}
		},
		methods: {
			async loadApiData() {
				this.isLoading = true;
				this.error = null;
				try {
					const groups = await api.groupMine();
					this.apiData = [
						{
							id: "personal_space", name: "个人空间", type: "personal", role: "owner",
							updatedAt: new Date().toISOString(), path: "/个人空间", visibility: "private",
							children: [
								{ id: "ps_my_projects", name: "我的项目", type: "folder", updatedAt: new Date().toISOString(), path: "/个人空间/我的项目", children: [] },
								{ id: "ps_followed", name: "星标项目", type: "folder", updatedAt: new Date().toISOString(), path: "/个人空间/星标项目", children: [] }
							]
						},
						{
							id: "groups", name: "群组", type: "folder", updatedAt: new Date().toISOString(), path: "/群组",
							children: (groups.data || []).filter(group => group._id).map(group => ({
								id: `group_${group._id}`, name: group.group_name, type: "group", role: group.role,
								updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}`,
								visibility: group.type === "private" ? "private" : "public", description: group.group_desc,
								createdBy: group.uid, createdAt: group.add_time,
								stats: {
									projectCount: group.project_count || 0,
									docCount: group.doc_count || 0,
									memberCount: group.member_count || 0,
									activityDays: group.activity_days || 0
								},
								children: [
									{ id: `group_${group._id}_projects`, name: "Projects", type: "folder", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Projects`,
										children: (group.projects || []).map(project => ({
											id: project._id || project.id, name: project.name, type: "project", role: project.role,
											updatedAt: project.up_time || project.updated_at, path: `/群组/${group.group_name}/Projects/${project.name}`,
											visibility: project.project_type || project.visibility, followed: project.follow,
											content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API 端点 for ${project.name}` }
										}))
									},
									{ id: `group_${group._id}_members`, name: "Group Members", type: "member_list", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Group Members`, content: [] },
									{ id: `group_${group._id}_docs`, name: "Group Docs", type: "doc_folder", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Group Docs`, children: [] },
									{ id: `group_${group._id}_settings`, name: "Group Settings", type: "setting", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Group Settings`,
										content: { name: group.group_name, description: group.group_desc, visibility: group.type === "private" ? "private" : "public",
											defaultRole: group.default_role || "guest", allowExternalShare: group.allow_external_share || false }
									},
									{ id: `group_${group._id}_log`, name: "Activity Log", type: "log", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Activity Log`, content: [] }
								]
							}))
						}
					];
					const personalProjects = await api.project_page({ page: 1, size: 1000, name: "", isPersonal: true });
					const allPersonalProjects = personalProjects.data?.list || [];
					this.apiData[0].children[0].children = allPersonalProjects
						.filter(project => !project.follow)
						.map(project => ({
							id: project._id || project.id, name: project.name, type: "project", role: project.role,
							updatedAt: project.up_time || project.updated_at, path: `/个人空间/我的项目/${project.name}`,
							visibility: project.project_type || project.visibility, followed: project.follow,
							content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API 端点 for ${project.name}` }
						}));
					this.apiData[0].children[1].children = allPersonalProjects
						.filter(project => project.follow)
						.map(project => ({
							id: project._id || project.id, name: project.name, type: "project", role: project.role,
							updatedAt: project.up_time || project.updated_at, path: `/个人空间/星标项目/${project.name}`,
							visibility: project.project_type || project.visibility, followed: project.follow,
							content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API 端点 for ${project.name}` }
						}));
				} catch (error) {
					console.error("Failed to load API data:", error);
					this.error = "加载数据失败，请刷新页面重试";
				} finally {
					this.isLoading = false;
				}
			},
			toggleFolder(folderId) {
				const index = this.expandedFolders.indexOf(folderId);
				if (index > -1) {
					this.expandedFolders.splice(index, 1);
				} else {
					this.expandedFolders.push(folderId);
				}
				utils.saveExpandedFolders(this.expandedFolders);
			},
			expandAncestors(nodeId) {
				let currentId = nodeId;
				while (currentId !== "personal_space" && currentId !== "groups") {
					const parent = utils.findParentNode(this.apiData, currentId);
					if (parent && this.expandedFolders.indexOf(parent.id) === -1) {
						this.expandedFolders.push(parent.id);
						currentId = parent.id;
					} else break;
				}
			},
			handleOpenNode(node) {
				this.selectedNodeId = node.id;
				utils.saveSelectedNodeId(node.id);
				this.selectedFile = null;
				if (utils.isFolderType(node.type) && this.expandedFolders.indexOf(node.id) === -1) {
					this.expandedFolders.push(node.id);
					utils.saveExpandedFolders(this.expandedFolders);
				}
				const appId = node.type === "api" ? "api_endpoint" : node.type;
				if (this.system?.openApp) this.system.openApp(appId, true, node);
			},
			handleNavigateUp() {
				if (!this.canNavigateUp) return;
				const parent = utils.findParentNode(this.apiData, this.activeNode?.id);
				if (parent) this.handleOpenNode(parent);
			},
			handleSort(field) {
				this.sortDirection = this.sortField === field ? (this.sortDirection === "asc" ? "desc" : "asc") : "asc";
				this.sortField = field;
			},
			showContextMenu(node, e) {
				this.contextMenu = { show: true, x: e.clientX, y: e.clientY, node };
			},
			hideContextMenu() {
				this.contextMenu.show = false;
			},
			handleContextMenuAction(action, node) {
				switch (action) {
					case "rename": {
						const newName = prompt("Enter new name:", node.name);
						if (newName && newName.trim()) { node.name = newName.trim(); _.msg("Renamed successfully"); }
						break;
					}
					case "move": _.msg("Move functionality not implemented yet"); break;
					case "delete": {
						if (confirm(`Are you sure you want to delete "${node.name}"?`)) {
							const parent = utils.findParentNode(this.apiData, node.id);
							if (parent && parent.children) {
								parent.children = parent.children.filter(child => child.id !== node.id);
								_.msg("Deleted successfully");
							}
						}
						break;
					}
					case "copyLink": {
						const link = `${window.location.origin}${window.location.pathname}?nodeId=${node.id}`;
						navigator.clipboard.writeText(link).then(() => _.msg("Link copied to clipboard"));
						break;
					}
					case "openNewTab": {
						const link = `${window.location.origin}${window.location.pathname}?nodeId=${node.id}`;
						window.open(link, "_blank");
						break;
					}
				}
				this.hideContextMenu();
			},
			startEditing() {
				this.editingContent = JSON.parse(JSON.stringify(this.activeNode?.content || {}));
			},
			saveEditing() {
				if (this.activeNode) this.activeNode.content = this.editingContent;
				this.editingContent = null;
			},
			cancelEditing() {
				this.editingContent = null;
			},
			sendRequest() {
				this.isRequesting = true;
				this.responseData = null;
				const startTime = performance.now();
				setTimeout(() => {
					this.isRequesting = false;
					this.responseStatus = 200;
					this.responseTime = Math.round(performance.now() - startTime);
					const endpoint = this.activeNode.content?.endpoint || "";
					let mockResponse = {};
					if (endpoint.includes("users") && this.activeNode.content?.method === "GET") {
						mockResponse = { status: "success", environment: this.activeEnvironment,
							data: [{ id: 1, name: "Alice Smith", email: "alice@example.com" },
								{ id: 2, name: "Bob Jones", email: "bob@example.com" }], meta: { total: 2, page: 1 } };
					} else if (endpoint.includes("users") && this.activeNode.content?.method === "POST") {
						mockResponse = { status: "success", environment: this.activeEnvironment,
							message: "User created successfully", data: { id: 3, name: "New User", email: "new@example.com" } };
					} else {
						mockResponse = { status: "success", environment: this.activeEnvironment,
							message: "Request executed successfully", timestamp: new Date().toISOString() };
					}
					this.responseData = JSON.stringify(mockResponse, null, 2);
				}, 800 + Math.random() * 500);
			},
			openCreateProjectDialog() {
				this.showCreateProjectDialog = true;
			},
			closeCreateProjectDialog() {
				this.showCreateProjectDialog = false;
			},
			async createProject(formData) {
				if (!formData.name.trim()) { _.msg.error("项目名称不能为空"); return; }
				this.creatingProject = true;
				try {
					const response = await api.project_add({ name: formData.name, desc: formData.description, visibility: formData.visibility });
					if (response.errcode === 0) {
						_.msg.success("项目创建成功");
						this.showCreateProjectDialog = false;
						this.loadApiData();
					} else {
						_.msg.error(response.errmsg || "项目创建失败");
					}
				} catch (error) {
					console.error("Failed to create project:", error);
					_.msg.error("项目创建失败，请重试");
				} finally {
					this.creatingProject = false;
				}
			},
			async toggleStarProject(project) {
					try {
						let response;
						if (project.followed) {
							response = await api.projectDelFollow(project.id);
						} else {
							response = await api.projectAddFollow({ projectid: project.id, projectname: project.name });
						}
						if (response.errcode === 0) {
							_.msg.success(project.followed ? "取消星标成功" : "星标成功");
							project.followed = !project.followed;
							this.loadApiData();
						} else {
							_.msg.error(response.errmsg || "操作失败");
						}
					} catch (error) {
						console.error("Failed to toggle star:", error);
						_.msg.error("操作失败，请重试");
					}
				},
			isFolderType(type) {
				return utils.isFolderType(type);
			}
		}
	};
}
</script>

<template>
	<div class="api-manager" @click="hideContextMenu">
		<ApiManagerToolbar
			:search-query="searchQuery"
			:active-environment="activeEnvironment"
			:view-mode="viewMode"
			:show-preview="showPreview"
			:can-navigate-up="canNavigateUp"
			:environments="environments"
			@update:search-query="searchQuery = $event"
			@update:active-environment="activeEnvironment = $event"
			@update:view-mode="viewMode = $event"
			@update:show-preview="showPreview = $event"
			@navigate-up="handleNavigateUp"
		/>

		<div class="api-manager__body">
			<ApiManagerSidebar
				:api-data="apiData"
				:expanded-folders="expandedFolders"
				:active-node-id="selectedNodeId"
				:is-loading="isLoading"
				:error="error"
				:tree-search-query.sync="treeSearchQuery"
				@toggle-folder="toggleFolder"
				@open-node="handleOpenNode"
				@context-menu="showContextMenu"
				@create-project="openCreateProjectDialog"
				@toggle-star="toggleStarProject"
				@retry="loadApiData"
			/>

			<div class="api-manager__content">
				<div class="api-manager__main">
					<ApiManagerBreadcrumb :path="activeNode?.path" @navigate-to="() => {}" />

					<template v-if="isFolderType(activeNode?.type)">
						<ApiManagerFileList
							v-if="viewMode === 'list'"
							:files="filteredAndSortedFiles"
							:selected-file-id="selectedFile?.id"
							:sort-field="sortField"
							:sort-direction="sortDirection"
							@select-file="selectedFile = $event"
							@open-file="handleOpenNode"
							@sort="handleSort"
						/>
						<ApiManagerFileGrid
							v-else
							:files="filteredAndSortedFiles"
							:selected-file-id="selectedFile?.id"
							@select-file="selectedFile = $event"
							@open-file="handleOpenNode"
						/>
					</template>

					<ApiManagerEditor
						v-else
						:node="activeNode"
						:editing-content="editingContent"
						:is-requesting="isRequesting"
						:response-data="responseData"
						:response-status="responseStatus"
						:response-time="responseTime"
						:active-environment="activeEnvironment"
						@start-edit="startEditing"
						@save-edit="saveEditing"
						@cancel-edit="cancelEditing"
						@send-request="sendRequest"
					/>
				</div>

				<ApiManagerPreview
					v-if="showPreview && isFolderType(activeNode?.type)"
					:selected-file="selectedFile"
					:is-folder-type="selectedFile ? isFolderType(selectedFile.type) : false"
					@open-node="handleOpenNode"
				/>
			</div>
		</div>

		<ApiManagerContextMenu
			:show="contextMenu.show"
			:x="contextMenu.x"
			:y="contextMenu.y"
			:node="contextMenu.node"
			@action="handleContextMenuAction"
			@hide="hideContextMenu"
		/>

		<ApiManagerCreateProjectDialog
			:show="showCreateProjectDialog"
			:creating-project="creatingProject"
			@close="closeCreateProjectDialog"
			@create="createProject"
		/>
	</div>
</template>

<style lang="less">
.api-manager {
	height: 100%;
	display: flex;
	flex-direction: column;
	overflow: hidden;
	background: var(--color-surface);
	color: var(--color-on-surface);
	font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", "PingFang SC", "Hiragino Sans GB", "Microsoft Yahei", sans-serif;

	&__topbar {
		display: flex;
		flex-direction: column;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container-lowest);
	}

	&__toolbar {
		display: flex;
		align-items: center;
		gap: 16px;
		padding: 8px 16px;
	}

	&__nav {
		display: flex;
		align-items: center;
		gap: 4px;
	}

	&__search {
		flex: 1;
		max-width: 42rem;
		position: relative;
	}

	&__search-icon {
		position: absolute;
		top: 0;
		bottom: 0;
		left: 12px;
		display: flex;
		align-items: center;
		pointer-events: none;
		color: var(--color-on-surface-variant);
	}

	&__search-input {
		width: 100%;
		padding: 6px 12px 6px 40px;
	}

	&__toolbar-right {
		display: flex;
		align-items: center;
		gap: 12px;
		margin-left: auto;
	}

	&__select-shell--toolbar {
		display: flex;
		align-items: center;
		padding: 6px 12px;
	}

	&__select-icon {
		margin-right: 8px;
		color: var(--color-on-surface-variant);
	}

	&__divider {
		width: 1px;
		height: 24px;
		background: color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	}

	&__body {
		flex: 1;
		display: flex;
		overflow: hidden;
		background: var(--color-surface-container-lowest);
		min-height: 0;
	}

	&__sidebar {
		width: 256px;
		flex-shrink: 0;
		min-height: 0;
		border-right: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container);
		overflow-y: auto;
	}

	&__sidebar-inner {
		padding: 8px;
	}

	&__sidebar-title {
		padding: 0 8px;
		margin: 0 0 8px;
		font-size: 12px;
		font-weight: 700;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: var(--color-on-surface-variant);
	}

	&__tree {
		user-select: none;
	}

	&__tree-item {
		display: flex;
		align-items: center;
		gap: 0;
		padding: 4px 8px;
		border-radius: 10px;
		cursor: pointer;
		font-size: 14px;
		color: var(--color-on-surface);
		transition: background-color 0.15s ease, color 0.15s ease;

		&:hover {
			background: var(--color-surface-variant);
		}
	}

	&__tree-item--active {
		background: var(--color-primary-container);
		color: var(--color-on-primary-container);

		&:hover {
			background: var(--color-primary-container);
		}
	}

	&__tree-toggle {
		width: 16px;
		height: 16px;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		margin-right: 4px;
		color: var(--color-on-surface-variant);
	}

	&__tree-item:hover &__tree-toggle {
		color: var(--color-on-surface);
	}

	&__tree-toggle--spacer {
		visibility: hidden;
		pointer-events: none;
	}

	&__tree-icon {
		margin-right: 8px;
		display: inline-flex;
		align-items: center;
		justify-content: center;
	}

	&__tree-label {
		flex: 1;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	&__content {
		flex: 1;
		display: flex;
		overflow: hidden;
		min-height: 0;
	}

	&__main {
		flex: 1;
		display: flex;
		flex-direction: column;
		overflow: hidden;
		min-height: 0;
		min-width: 0;
		background: var(--color-surface-container-lowest);
	}

	&__breadcrumb {
		display: flex;
		align-items: center;
		padding: 8px 16px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container);
		font-size: 14px;
		color: var(--color-on-surface-variant);
	}

	&__breadcrumb-part {
		cursor: pointer;
		transition: color 0.15s ease;

		&:hover {
			color: var(--color-primary);
		}
	}

	&__breadcrumb-part--current {
		cursor: default;
		font-weight: 600;
		color: var(--color-on-surface);

		&:hover {
			color: var(--color-on-surface);
		}
	}

	&__breadcrumb-sep {
		margin: 0 8px;
		color: color-mix(in srgb, var(--color-on-surface-variant) 55%, transparent);
	}

	&__table-header {
		display: grid;
		grid-template-columns: 7fr 3fr 2fr;
		gap: 16px;
		padding: 8px 16px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container);
		font-size: 12px;
		font-weight: 700;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: var(--color-on-surface-variant);
		user-select: none;
	}

	&__table-header-cell {
		display: inline-flex;
		align-items: center;
		cursor: pointer;
		transition: color 0.15s ease;

		&:hover {
			color: var(--color-on-surface);
		}
	}

	&__sort-icon {
		margin-left: 4px;
	}

	&__file-list {
		flex: 1;
		overflow-y: auto;
	}

	&__file-list-inner {
		padding: 4px 0;
	}

	&__file-row {
		display: grid;
		grid-template-columns: 7fr 3fr 2fr;
		gap: 16px;
		padding: 8px 16px;
		font-size: 14px;
		align-items: center;
		cursor: pointer;
		user-select: none;
		border-bottom: 1px solid transparent;
		transition: background-color 0.15s ease, border-color 0.15s ease;

		&:hover {
			background: color-mix(in srgb, var(--color-surface-variant) 50%, transparent);
		}
	}

	&__file-row--selected {
		background: color-mix(in srgb, var(--color-primary-container) 50%, transparent);
		border-color: color-mix(in srgb, var(--color-primary) 12%, transparent);
	}

	&__file-cell {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		color: var(--color-on-surface-variant);
	}

	&__file-cell--name {
		display: flex;
		align-items: center;
		gap: 12px;
		color: var(--color-on-surface);
	}

	&__file-name {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		font-weight: 600;
	}

	&__empty-state {
		height: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 8px;
		color: var(--color-on-surface-variant);
	}

	&__empty-icon {
		margin-bottom: 8px;
		color: var(--color-outline-variant);
	}

	&__editor-wrap {
		flex: 1;
		overflow-y: auto;
		padding: 24px;
		background: var(--color-surface);
	}

	&__editor-card--wide {
		max-width: 60rem;
		margin: 0 auto;
		overflow: hidden;
	}

	&__editor-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 16px 24px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: color-mix(in srgb, var(--color-surface-container) 30%, transparent);
	}

	&__editor-header-left {
		display: flex;
		align-items: center;
		gap: 12px;
	}

	&__editor-header-copy {
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	&__editor-title {
		margin: 0;
		font-size: 18px;
		font-weight: 800;
		color: var(--color-on-surface);
	}

	&__editor-subtitle {
		margin: 0;
		font-size: 12px;
		font-weight: 600;
		color: var(--color-on-surface-variant);
		text-transform: capitalize;
	}

	&__editor-header-actions {
		display: flex;
		gap: 8px;
	}

	&__editor-body {
		padding: 24px;
	}

	&__action-btn--with-icon {
		display: inline-flex;
		align-items: center;
		gap: 8px;
	}

	&__section {
		display: flex;
		flex-direction: column;
		gap: 16px;
	}

	&__section-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	&__section-title {
		margin: 0;
		font-size: 14px;
		font-weight: 700;
		color: var(--color-on-surface);
	}

	&__link-btn {
		border: 0;
		background: transparent;
		padding: 0;
		color: var(--color-primary);
		cursor: pointer;
		font-size: 14px;
		font-weight: 600;
	}

	&__link-btn--with-icon {
		display: inline-flex;
		align-items: center;
		gap: 6px;
	}

	&__link-btn:hover {
		text-decoration: underline;
	}

	&__table-shell {
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 14px;
		overflow: hidden;
	}

	&__table {
		width: 100%;
		border-collapse: collapse;
		font-size: 14px;
		color: var(--color-on-surface);
	}

	&__table-head {
		background: var(--color-surface-container);
		color: var(--color-on-surface-variant);
		font-size: 12px;
		text-transform: uppercase;
		letter-spacing: 0.06em;
	}

	&__th, &__td {
		padding: 12px 16px;
		text-align: left;
	}

	&__th--right, &__td--right {
		text-align: right;
	}

	&__td--strong {
		font-weight: 600;
		color: var(--color-on-surface);
	}

	&__td--muted {
		color: var(--color-on-surface-variant);
	}

	&__tr {
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);

		&:last-child {
			border-bottom: 0;
		}

		&:hover {
			background: color-mix(in srgb, var(--color-surface-variant) 30%, transparent);
		}
	}

	&__settings {
		max-width: 42rem;
		display: flex;
		flex-direction: column;
		gap: 16px;
	}

	&__setting-row {
		display: grid;
		grid-template-columns: 1fr 2fr;
		gap: 16px;
		align-items: center;
	}

	&__setting-label {
		font-size: 14px;
		font-weight: 600;
		color: var(--color-on-surface-variant);
		text-transform: capitalize;
	}

	&__field-input--block {
		width: 100%;
		padding: 8px 12px;
	}

	&__field-value {
		padding: 8px 12px;
		border-radius: 10px;
		background: var(--color-surface-container-lowest);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		font-size: 14px;
		color: var(--color-on-surface);
	}

	&__api-editor {
		display: flex;
		flex-direction: column;
		gap: 20px;
	}

	&__api-row {
		display: flex;
		align-items: center;
		gap: 16px;
		flex-wrap: wrap;
	}

	&__method-select {
		padding: 8px 12px;
		font-weight: 800;
	}

	&__endpoint-input {
		flex: 1;
		min-width: 240px;
		padding: 8px 12px;
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
	}

	&__endpoint-text {
		flex: 1;
		min-width: 240px;
		font-size: 16px;
		font-weight: 600;
		color: var(--color-on-surface);
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	&__spinner {
		animation: api-manager-spin 1s linear infinite;
	}

	&__spinner--lg {
		font-size: 32px;
	}

	@keyframes api-manager-spin {
		from { transform: rotate(0deg); }
		to { transform: rotate(360deg); }
	}

	&__field {
		display: flex;
		flex-direction: column;
		gap: 6px;
	}

	&__field-label {
		font-size: 14px;
		font-weight: 600;
		color: var(--color-on-surface-variant);
	}

	&__field-label--spaced {
		margin-bottom: 8px;
	}

	&__field-input--textarea {
		width: 100%;
		padding: 8px 12px;
		resize: vertical;
	}

	&__field-text {
		font-size: 14px;
		color: var(--color-on-surface);
		margin: 0;
	}

	&__code-block {
		padding: 12px;
		border-radius: 14px;
		background: var(--color-surface-container);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 45%, transparent);
		overflow-x: auto;
		font-size: 14px;
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;

		pre {
			margin: 0;
			white-space: pre;
		}
	}

	&__code-block--wrap pre {
		white-space: pre-wrap;
	}

	&__code-block--preview {
		max-height: 160px;
		overflow-y: auto;
	}

	&__response {
		margin-top: 24px;
		padding-top: 24px;
		border-top: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	}

	&__response-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		margin-bottom: 16px;
	}

	&__response-title {
		margin: 0;
		font-size: 16px;
		font-weight: 800;
		color: var(--color-on-surface);
		display: flex;
		align-items: center;
		gap: 8px;
	}

	&__response-meta {
		font-size: 12px;
		color: var(--color-on-surface-variant);
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
	}

	&__state-panel {
		border-radius: 14px;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container-lowest);
		padding: 24px;
	}

	&__state-panel--loading, &__state-panel--dashed {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 12px;
		min-height: 140px;
		color: var(--color-on-surface-variant);
		text-align: center;
	}

	&__state-panel--dashed {
		border-style: dashed;
	}

	&__state-panel--code {
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
		font-size: 14px;
		overflow-x: auto;

		pre {
			margin: 0;
			white-space: pre;
		}
	}

	&__state-text {
		margin: 0;
		font-size: 14px;
	}

	&__doc-editor {
		min-height: 300px;
	}

	&__doc-textarea {
		width: 100%;
		min-height: 300px;
		padding: 12px;
		resize: vertical;
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
		font-size: 14px;
	}

	&__json-textarea {
		width: 100%;
		height: 16rem;
		padding: 12px;
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
		font-size: 14px;
	}

	&__preview {
		width: 320px;
		flex-shrink: 0;
		border-left: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container);
		display: flex;
		flex-direction: column;
		overflow-y: auto;
		min-height: 0;
	}

	&__preview-empty {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 24px;
		text-align: center;
		color: var(--color-on-surface-variant);
		gap: 12px;
	}

	&__preview-empty-icon {
		margin-bottom: 4px;
		color: var(--color-outline-variant);
	}

	&__preview-inner {
		padding: 24px;
	}

	&__preview-hero {
		display: flex;
		justify-content: center;
		margin-bottom: 16px;
	}

	&__preview-title {
		margin: 0 0 4px;
		font-size: 18px;
		font-weight: 800;
		color: var(--color-on-surface);
	}

	&__preview-subtitle {
		margin: 0 0 16px;
		font-size: 12px;
		font-weight: 600;
		color: var(--color-on-surface-variant);
		text-transform: capitalize;
	}

	&__preview-meta {
		display: flex;
		flex-direction: column;
		gap: 16px;
	}

	&__preview-section {
		display: flex;
		flex-direction: column;
		gap: 8px;
	}

	&__preview-section-title {
		margin: 0;
		font-size: 12px;
		font-weight: 700;
		letter-spacing: 0.08em;
		text-transform: uppercase;
		color: var(--color-on-surface-variant);
	}

	&__preview-dl {
		margin: 0;
		display: flex;
		flex-direction: column;
		gap: 8px;
	}

	&__preview-dl-row {
		display: flex;
		gap: 8px;
	}

	&__preview-dt {
		font-size: 12px;
		font-weight: 600;
		color: var(--color-on-surface-variant);
		min-width: 60px;
		flex-shrink: 0;
	}

	&__preview-dd {
		font-size: 14px;
		color: var(--color-on-surface);
	}

	&__preview-dd--break {
		word-break: break-all;
	}

	&__preview-code {
		margin: 0;
		white-space: pre-wrap;
		word-break: break-all;
	}

	&__preview-action {
		margin-top: 8px;
		width: 100%;
	}

	&__context-menu {
		position: fixed;
		background: var(--color-surface-container);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 14px;
		padding: 4px;
		min-width: 180px;
		box-shadow: 0 4px 24px rgba(0, 0, 0, 0.15);
		z-index: 1000;
	}

	&__context-menu-item {
		display: flex;
		align-items: center;
		gap: 8px;
		padding: 8px 12px;
		cursor: pointer;
		border-radius: 8px;
		transition: background-color 0.15s ease;

		&:hover {
			background: var(--color-surface-variant);
		}
	}

	&__context-menu-item--danger {
		color: var(--color-error);

		&:hover {
			background: color-mix(in srgb, var(--color-error) 10%, transparent);
		}
	}

	&__context-menu-divider {
		height: 1px;
		background: color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		margin: 4px 0;
	}

	&__file-grid {
		flex: 1;
		overflow-y: auto;
		padding: 16px;
	}

	&__file-grid-inner {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
		gap: 12px;
	}

	&__file-card {
		display: flex;
		flex-direction: column;
		gap: 8px;
		padding: 12px;
		background: var(--color-surface-container);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 30%, transparent);
		border-radius: 14px;
		cursor: pointer;
		transition: all 0.15s ease;

		&:hover {
			background: color-mix(in srgb, var(--color-surface-variant) 30%, transparent);
			border-color: color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		}
	}

	&__file-card--selected {
		background: color-mix(in srgb, var(--color-primary-container) 30%, transparent);
		border-color: color-mix(in srgb, var(--color-primary) 20%, transparent);
	}

	&__file-card-icon {
		display: flex;
		justify-content: center;
	}

	&__file-card-content {
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	&__file-card-name {
		margin: 0;
		font-size: 14px;
		font-weight: 600;
		color: var(--color-on-surface);
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	&__file-card-meta {
		margin: 0;
		font-size: 12px;
		color: var(--color-on-surface-variant);
	}

	&__file-card-date {
		margin: 0;
		font-size: 12px;
		color: var(--color-on-surface-variant);
	}

	&__tree-actions {
		display: flex;
		gap: 4px;
		margin-left: auto;
	}

	&__tree-action-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 24px;
		height: 24px;
		border: 0;
		background: transparent;
		border-radius: 6px;
		cursor: pointer;
		transition: background-color 0.15s ease;

		&:hover {
			background: var(--color-surface-variant);
		}
	}

	&__star-icon--active {
		color: var(--color-warning);
	}

	&__project-badge {
		font-size: 10px;
		font-weight: 700;
		padding: 2px 6px;
		border-radius: 4px;
		margin-left: 4px;
	}

	&__project-badge--personal {
		background: color-mix(in srgb, var(--color-primary) 15%, transparent);
		color: var(--color-primary);
	}

	&__project-badge--starred {
		background: color-mix(in srgb, var(--color-warning) 15%, transparent);
		color: var(--color-warning);
	}

	&__project-badge--group {
		background: color-mix(in srgb, var(--color-secondary) 15%, transparent);
		color: var(--color-secondary);
	}

	&__loading {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 24px;
		gap: 8px;
		color: var(--color-on-surface-variant);
	}

	&__loading-icon {
		animation: api-manager-spin 1s linear infinite;
	}

	&__error {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 24px;
		gap: 8px;
		color: var(--color-on-surface-variant);
	}

	&__error-icon {
		color: var(--color-error);
	}

	&__retry-btn {
		margin-top: 8px;
		padding: 8px 16px;
		background: var(--color-primary);
		color: var(--color-on-primary);
		border: 0;
		border-radius: 8px;
		cursor: pointer;
		font-size: 14px;
		font-weight: 600;
		transition: background-color 0.15s ease;

		&:hover {
			background: var(--color-primary-container);
			color: var(--color-on-primary-container);
		}
	}

	&__toolbar-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 32px;
		height: 32px;
		border: 0;
		background: transparent;
		border-radius: 8px;
		cursor: pointer;
		transition: background-color 0.15s ease;

		&:hover {
			background: var(--color-surface-variant);
		}

		&--disabled {
			opacity: 0.4;
			cursor: not-allowed;

			&:hover {
				background: transparent;
			}
		}
	}

	&__toggle-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 32px;
		height: 32px;
		border: 0;
		background: transparent;
		border-radius: 8px;
		cursor: pointer;
		transition: all 0.15s ease;

		&:hover {
			background: var(--color-surface-variant);
		}

		&--active {
			background: var(--color-primary-container);
			color: var(--color-on-primary-container);
		}
	}

	&__view-toggle {
		display: flex;
		background: var(--color-surface-variant);
		border-radius: 8px;
		overflow: hidden;
	}

	&__view-toggle-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 32px;
		height: 32px;
		border: 0;
		background: transparent;
		cursor: pointer;
		transition: all 0.15s ease;

		&:hover {
			background: var(--color-surface);
		}

		&--active {
			background: var(--color-surface);
			color: var(--color-primary);
		}
	}

	&__select {
		border: 0;
		background: transparent;
		cursor: pointer;
		font-size: 14px;
		color: var(--color-on-surface);
	}

	&__action-btn {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding: 8px 16px;
		border: 0;
		border-radius: 10px;
		cursor: pointer;
		font-size: 14px;
		font-weight: 600;
		transition: all 0.15s ease;

		&--primary {
			background: var(--color-primary);
			color: var(--color-on-primary);

			&:hover {
				background: var(--color-primary-container);
				color: var(--color-on-primary-container);
			}

			&:disabled {
				opacity: 0.5;
				cursor: not-allowed;
			}
		}

		&--secondary {
			background: var(--color-surface-variant);
			color: var(--color-on-surface);

			&:hover {
				background: var(--color-surface-container);
			}
		}

		&--ghost {
			background: transparent;
			color: var(--color-on-surface);

			&:hover {
				background: var(--color-surface-variant);
			}
		}
	}

	&__icon-action {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 28px;
		height: 28px;
		border: 0;
		background: transparent;
		border-radius: 6px;
		cursor: pointer;
		color: var(--color-on-surface-variant);
		transition: all 0.15s ease;

		&:hover {
			background: var(--color-surface-variant);
			color: var(--color-error);
		}
	}

	&__inline-badge {
		display: inline-block;
		padding: 2px 8px;
		background: var(--color-surface-variant);
		border-radius: 4px;
		font-size: 12px;
		font-weight: 600;
		color: var(--color-on-surface-variant);
	}

	&__field-input {
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 10px;
		background: var(--color-surface-container-lowest);
		font-size: 14px;
		color: var(--color-on-surface);

		&--inline {
			padding: 4px 8px;
		}

		&--code {
			font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
		}
	}

	&__method-badge {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding: 4px 12px;
		border-radius: 6px;
		font-size: 14px;
		font-weight: 800;

		&--get {
			background: color-mix(in srgb, var(--color-success) 15%, transparent);
			color: var(--color-success);
		}

		&--post {
			background: color-mix(in srgb, var(--color-primary) 15%, transparent);
			color: var(--color-primary);
		}

		&--put {
			background: color-mix(in srgb, var(--color-warning) 15%, transparent);
			color: var(--color-warning);
		}

		&--delete {
			background: color-mix(in srgb, var(--color-error) 15%, transparent);
			color: var(--color-error);
		}

		&--default {
			background: var(--color-surface-variant);
			color: var(--color-on-surface-variant);
		}
	}

	&__status-badge {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding: 2px 8px;
		background: color-mix(in srgb, var(--color-success) 15%, transparent);
		color: var(--color-success);
		border-radius: 4px;
		font-size: 12px;
		font-weight: 700;
	}

	&__icon {
		&--personal { color: var(--color-primary); }
		&--group { color: var(--color-secondary); }
		&--project-personal { color: var(--color-primary); }
		&--project-starred { color: var(--color-warning); }
		&--project-group { color: var(--color-secondary); }
		&--project { color: var(--color-on-surface-variant); }
		&--api-folder { color: var(--color-primary); }
		&--doc-folder { color: var(--color-secondary); }
		&--folder { color: var(--color-on-surface-variant); }
		&--api { color: var(--color-primary); }
		&--doc { color: var(--color-secondary); }
		&--code { color: var(--color-warning); }
		&--member-list { color: var(--color-secondary); }
		&--setting { color: var(--color-on-surface-variant); }
		&--cicd { color: var(--color-success); }
		&--log { color: var(--color-on-surface-variant); }
		&--default { color: var(--color-on-surface-variant); }
	}
}
</style>