<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
	const api = await _.$importVue("@/utils/api.vue");

	return {
		props: {
			windowData: Object
		},
		// system: v1 Desktop Workspace 局部状态（来自 ViewXspace）
		inject: {
			system: {
				default: null
			}
		},
		data() {
			return {
				// API Data
				apiData: null,
				// Global Environment State
				environments: ["Development", "Test", "Staging", "Production"],
				activeEnvironment: "Development",
				// State
				selectedFile: null,
				searchQuery: "",
				showPreview: true,
				viewMode: "list", // list | grid
				expandedFolders: new Set(["personal_space", "groups"]),
				selectedNodeId: "personal_space",
				// Loading and Error State
				isLoading: false,
				error: null,
				// Sort State
				sortField: "name",
				sortDirection: "asc",
				// API Execution State
				isRequesting: false,
				responseData: null,
				responseStatus: null,
				responseTime: null,
				// Editors State
				editingContent: null,
				// Context Menu State
				contextMenu: {
					show: false,
					x: 0,
					y: 0,
					node: null
				},
				// Create Project Dialog State
				showCreateProjectDialog: false,
				createProjectForm: {
					name: "",
					description: "",
					visibility: "private"
				},
				createProjectFormConfigs: {
					name: {
						label: "项目名称",
						value: "",
						placeholder: "请输入项目名称",
						rules: [{ required: true, message: "请输入项目名称" }]
					},
					description: {
						label: "项目描述",
						type: "textarea",
						value: "",
						placeholder: "请输入项目描述"
					},
					visibility: {
						label: "项目可见性",
						type: "select",
						value: "private",
						options: [
							{ label: "私有", value: "private" },
							{ label: "内部", value: "internal" },
							{ label: "公开", value: "public" }
						]
					}
				},
				creatingProject: false
			};
		},
		computed: {
			activeNode() {
				if (this.windowData) {
					// 避免在计算属性中修改状态，移到mounted或watch中处理
					return this.windowData;
				}
				// 默认返回个人空间
				return this.apiData?.find(item => item.id === "personal_space");
			},
			filteredAndSortedFiles() {
				if (!this.isFolderType(this.activeNode?.type)) return [];

				let files = this.activeNode?.children || [];

				if (this.searchQuery) {
					const lowerQuery = this.searchQuery.toLowerCase();
					files = files.filter(f => f.name.toLowerCase().includes(lowerQuery));
				}

				return [...files].sort((a, b) => {
					let comparison = 0;

					if (
						(this.sortField === "name" || this.sortField === "type") &&
						this.isFolderType(a.type) !== this.isFolderType(b.type)
					) {
						if (this.isFolderType(a.type)) return -1;
						if (this.isFolderType(b.type)) return 1;
					}

					switch (this.sortField) {
						case "name":
							comparison = a.name.localeCompare(b.name);
							break;
						case "updatedAt":
							comparison =
								new Date(a.updatedAt).getTime() - new Date(b.updatedAt).getTime();
							break;
						case "type":
							comparison = a.type.localeCompare(b.type);
							break;
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
			// 初始化时处理windowData
			if (this.windowData) {
				this.expandAncestors(this.windowData.id);
			}
		},
		watch: {
			windowData: {
				handler(newVal) {
					if (newVal) {
						this.expandAncestors(newVal.id);
					}
				},
				immediate: false
			}
		},
		methods: {
			// API Data Loading
			async loadApiData() {
				this.isLoading = true;
				this.error = null;
				try {
					// 加载群组数据
					const groups = await api.groupMine();

					// 构建API数据结构
					this.apiData = [
						{
							id: "personal_space",
							name: "个人空间",
							type: "personal",
							role: "owner",
							updatedAt: new Date().toISOString(),
							path: "/个人空间",
							visibility: "private",
							children: [
								{
									id: "ps_my_projects",
									name: "我的项目",
									type: "folder",
									updatedAt: new Date().toISOString(),
									path: "/个人空间/我的项目",
									children: []
								},
								{
									id: "ps_followed",
									name: "星标项目",
									type: "folder",
									updatedAt: new Date().toISOString(),
									path: "/个人空间/星标项目",
									children: []
								}
							]
						},
						{
							id: "groups",
							name: "群组",
							type: "folder",
							updatedAt: new Date().toISOString(),
							path: "/群组",
							children:
								groups.data?.map(group => ({
									id: `group_${group.id}`,
									name: group.name,
									type: "group",
									role: group.role,
									updatedAt: new Date().toISOString(),
									path: `/群组/${group.name}`,
									visibility: group.visibility,
									description: group.desc,
									createdBy: group.created_by,
									createdAt: group.created_at,
									stats: {
										projectCount: group.project_count || 0,
										docCount: group.doc_count || 0,
										memberCount: group.member_count || 0,
										activityDays: group.activity_days || 0
									},
									children: [
										{
											id: `group_${group.id}_projects`,
											name: "Projects",
											type: "folder",
											updatedAt: new Date().toISOString(),
											path: `/群组/${group.name}/Projects`,
											children:
												group.projects?.map(project => ({
													id: project.id,
													name: project.name,
													type: "project",
													role: project.role,
													updatedAt: project.updated_at,
													path: `/群组/${group.name}/Projects/${project.name}`,
													visibility: project.visibility,
													followed: project.follow,
													content: {
														endpoint: `api/${project.id}/endpoints`,
														method: "GET",
														description: `API 端点 for ${project.name}`
													}
												})) || []
										},
										{
											id: `group_${group.id}_members`,
											name: "Group Members",
											type: "member_list",
											updatedAt: new Date().toISOString(),
											path: `/群组/${group.name}/Group Members`,
											content: [] // 后续通过API加载
										},
										{
											id: `group_${group.id}_docs`,
											name: "Group Docs",
											type: "doc_folder",
											updatedAt: new Date().toISOString(),
											path: `/群组/${group.name}/Group Docs`,
											children: [] // 后续通过API加载
										},
										{
											id: `group_${group.id}_settings`,
											name: "Group Settings",
											type: "setting",
											updatedAt: new Date().toISOString(),
											path: `/群组/${group.name}/Group Settings`,
											content: {
												name: group.name,
												description: group.desc,
												visibility: group.visibility,
												defaultRole: group.default_role || "guest",
												allowExternalShare:
													group.allow_external_share || false
											}
										},
										{
											id: `group_${group.id}_log`,
											name: "Activity Log",
											type: "log",
											updatedAt: new Date().toISOString(),
											path: `/群组/${group.name}/Activity Log`,
											content: [] // 后续通过API加载
										}
									]
								})) || []
						}
					];

					// 加载个人项目
					try {
						const personalProjects = await api.project_page({
							group_id: null,
							page: 1,
							size: 1000,
							name: ""
						});
						this.apiData[0].children[0].children =
							personalProjects.data?.list?.map(project => ({
								id: project.id,
								name: project.name,
								type: "project",
								role: project.role,
								updatedAt: project.updated_at,
								path: `/个人空间/我的项目/${project.name}`,
								visibility: project.visibility,
								followed: project.follow,
								content: {
									endpoint: `api/${project.id}/endpoints`,
									method: "GET",
									description: `API 端点 for ${project.name}`
								}
							})) || [];
					} catch (error) {
						console.error("Failed to load personal projects:", error);
					}

					// 加载星标项目 - 暂时使用空数组，因为API中没有直接获取星标项目的方法
					// 后续可以通过遍历所有项目并筛选followed为true的项目来实现
				} catch (error) {
					console.error("Failed to load API data:", error);
					this.error = "加载数据失败，请刷新页面重试";
				} finally {
					this.isLoading = false;
				}
			},
			// LocalStorage Utils
			loadExpandedFolders() {
				try {
					const saved = localStorage.getItem("apiManager_expandedFolders");
					return saved
						? new Set(JSON.parse(saved))
						: new Set(["personal_space", "groups"]);
				} catch {
					return new Set(["personal_space", "groups"]);
				}
			},
			saveExpandedFolders() {
				try {
					localStorage.setItem(
						"apiManager_expandedFolders",
						JSON.stringify([...this.expandedFolders])
					);
				} catch (e) {
					console.error("Failed to save expanded folders:", e);
				}
			},
			loadSelectedNodeId() {
				try {
					return localStorage.getItem("apiManager_selectedNodeId") || "personal_space";
				} catch {
					return "personal_space";
				}
			},
			saveSelectedNodeId(nodeId) {
				try {
					localStorage.setItem("apiManager_selectedNodeId", nodeId);
				} catch (e) {
					console.error("Failed to save selected node:", e);
				}
			},
			// Utils
			formatDate(dateString) {
				try {
					const date = new Date(dateString);
					return date.toLocaleString();
				} catch (e) {
					return dateString;
				}
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
			findParentNode(root, targetId) {
				// 处理根级别数组
				if (Array.isArray(root)) {
					for (const item of root) {
						if (item.id === targetId) return null; // 根级项目没有父节点
						const found = this.findParentNode(item, targetId);
						if (found) return found;
					}
					return null;
				}

				// 处理普通节点
				if (!root.children) return null;
				for (const child of root.children) {
					if (child.id === targetId) return root;
					const found = this.findParentNode(child, targetId);
					if (found) return found;
				}
				return null;
			},
			// Auto-expand ancestors of activeNode
			expandAncestors(nodeId) {
				let currentId = nodeId;
				while (currentId !== "personal_space" && currentId !== "groups") {
					const parent = this.findParentNode(this.apiData, currentId);
					if (parent) {
						this.expandedFolders.add(parent.id);
						currentId = parent.id;
					} else {
						break;
					}
				}
			},
			// Methods
			handleOpenNode(node) {
				const appId = node.type === "api" ? "api_endpoint" : node.type;
				if (this.system?.openApp) {
					this.system.openApp(appId, true, node);
				}
			},
			handleNavigateUp() {
				if (this.activeNode?.id === "personal_space" || this.activeNode?.id === "groups")
					return;
				const parent = this.findParentNode(this.apiData, this.activeNode?.id);
				if (parent) {
					this.handleOpenNode(parent);
				}
			},
			handleSort(field) {
				if (this.sortField === field) {
					this.sortDirection = this.sortDirection === "asc" ? "desc" : "asc";
				} else {
					this.sortField = field;
					this.sortDirection = "asc";
				}
			},
			toggleFolder(folderId, e) {
				e.stopPropagation();
				if (this.expandedFolders.has(folderId)) {
					this.expandedFolders.delete(folderId);
				} else {
					this.expandedFolders.add(folderId);
				}
				this.saveExpandedFolders();
			},
			// Context Menu Methods
			showContextMenu(node, e) {
				e.preventDefault();
				e.stopPropagation();
				this.contextMenu = {
					show: true,
					x: e.clientX,
					y: e.clientY,
					node
				};
			},
			hideContextMenu() {
				this.contextMenu.show = false;
			},
			handleContextMenuAction(action) {
				const { node } = this.contextMenu;
				if (!node) return;

				switch (action) {
					case "rename":
						this.renameNode(node);
						break;
					case "move":
						this.moveNode(node);
						break;
					case "delete":
						this.deleteNode(node);
						break;
					case "copyLink":
						this.copyNodeLink(node);
						break;
					case "openNewTab":
						this.openNodeInNewTab(node);
						break;
				}
				this.hideContextMenu();
			},
			renameNode(node) {
				const newName = prompt("Enter new name:", node.name);
				if (newName && newName.trim()) {
					node.name = newName.trim();
					_.$msg("Renamed successfully");
				}
			},
			moveNode(node) {
				_.$msg("Move functionality not implemented yet");
			},
			deleteNode(node) {
				if (confirm(`Are you sure you want to delete "${node.name}"?`)) {
					const parent = this.findParentNode(this.apiData, node.id);
					if (parent && parent.children) {
						parent.children = parent.children.filter(child => child.id !== node.id);
						_.$msg("Deleted successfully");
					}
				}
			},
			copyNodeLink(node) {
				const link = `${window.location.origin}${window.location.pathname}?nodeId=${node.id}`;
				navigator.clipboard.writeText(link).then(() => {
					_.$msg("Link copied to clipboard");
				});
			},
			openNodeInNewTab(node) {
				const link = `${window.location.origin}${window.location.pathname}?nodeId=${node.id}`;
				window.open(link, "_blank");
			},
			getIcon(type) {
				switch (type) {
					case "personal":
						return "user";
					case "group":
						return "folder";
					case "project":
						return "folder";
					case "api_folder":
						return "folder";
					case "doc_folder":
						return "folder";
					case "folder":
						return "folder";
					case "api":
						return "code";
					case "doc":
						return "file-text";
					case "code":
						return "code";
					case "member_list":
						return "users";
					case "setting":
						return "settings";
					case "cicd":
						return "activity";
					case "log":
						return "history";
					default:
						return "file";
				}
			},
			getIconColor(type, node) {
				switch (type) {
					case "personal":
						return "api-manager__icon api-manager__icon--personal";
					case "group":
						return "api-manager__icon api-manager__icon--group";
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
					case "api_folder":
						return "api-manager__icon api-manager__icon--api-folder";
					case "doc_folder":
						return "api-manager__icon api-manager__icon--doc-folder";
					case "folder":
						return "api-manager__icon api-manager__icon--folder";
					case "api":
						return "api-manager__icon api-manager__icon--api";
					case "doc":
						return "api-manager__icon api-manager__icon--doc";
					case "code":
						return "api-manager__icon api-manager__icon--code";
					case "member_list":
						return "api-manager__icon api-manager__icon--member-list";
					case "setting":
						return "api-manager__icon api-manager__icon--setting";
					case "cicd":
						return "api-manager__icon api-manager__icon--cicd";
					case "log":
						return "api-manager__icon api-manager__icon--log";
					default:
						return "api-manager__icon api-manager__icon--default";
				}
			},
			// Editors Methods
			startEditing() {
				this.editingContent = JSON.parse(JSON.stringify(this.activeNode?.content || {}));
			},
			saveEditing() {
				if (this.activeNode) {
					this.activeNode.content = this.editingContent;
				}
				this.editingContent = null;
			},
			cancelEditing() {
				this.editingContent = null;
			},
			getMethodColor(method) {
				switch (method?.toUpperCase()) {
					case "GET":
						return "api-manager__method-badge api-manager__method-badge--get";
					case "POST":
						return "api-manager__method-badge api-manager__method-badge--post";
					case "PUT":
						return "api-manager__method-badge api-manager__method-badge--put";
					case "DELETE":
						return "api-manager__method-badge api-manager__method-badge--delete";
					default:
						return "api-manager__method-badge api-manager__method-badge--default";
				}
			},
			handlePlainTextInput(event) {
				this.editingContent = event.target.value;
			},
			// API Execution
			sendRequest() {
				this.isRequesting = true;
				this.responseData = null;
				this.responseStatus = null;
				this.responseTime = null;

				const startTime = performance.now();

				// Simulate network request
				setTimeout(
					() => {
						this.isRequesting = false;
						this.responseStatus = 200;
						this.responseTime = Math.round(performance.now() - startTime);

						// Generate mock response based on endpoint
						const endpoint = this.activeNode.content?.endpoint || "";
						let mockResponse = {};

						if (
							endpoint.includes("users") &&
							this.activeNode.content?.method === "GET"
						) {
							mockResponse = {
								status: "success",
								environment: this.activeEnvironment,
								data: [
									{ id: 1, name: "Alice Smith", email: "alice@example.com" },
									{ id: 2, name: "Bob Jones", email: "bob@example.com" }
								],
								meta: { total: 2, page: 1 }
							};
						} else if (
							endpoint.includes("users") &&
							this.activeNode.content?.method === "POST"
						) {
							mockResponse = {
								status: "success",
								environment: this.activeEnvironment,
								message: "User created successfully",
								data: { id: 3, name: "New User", email: "new@example.com" }
							};
						} else {
							mockResponse = {
								status: "success",
								environment: this.activeEnvironment,
								message: "Request executed successfully",
								timestamp: new Date().toISOString()
							};
						}

						this.responseData = JSON.stringify(mockResponse, null, 2);
					},
					800 + Math.random() * 500
				);
			},
			// Create Project Methods
			openCreateProjectDialog() {
				this.createProjectForm = {
					name: "",
					description: "",
					visibility: "private"
				};
				// 同步表单配置值
				this.createProjectFormConfigs.name.value = "";
				this.createProjectFormConfigs.description.value = "";
				this.createProjectFormConfigs.visibility.value = "private";
				this.showCreateProjectDialog = true;
			},
			closeCreateProjectDialog() {
				this.showCreateProjectDialog = false;
			},
			async createProject() {
				if (!this.createProjectForm.name.trim()) {
					_.$msg.error("项目名称不能为空");
					return;
				}

				this.creatingProject = true;
				try {
					const response = await api.project_add({
						name: this.createProjectForm.name,
						desc: this.createProjectForm.description,
						visibility: this.createProjectForm.visibility
					});

					if (response.errcode === 0) {
						_.$msg.success("项目创建成功");
						this.showCreateProjectDialog = false;
						// 重新加载数据
						this.loadApiData();
					} else {
						_.$msg.error(response.errmsg || "项目创建失败");
					}
				} catch (error) {
					console.error("Failed to create project:", error);
					_.$msg.error("项目创建失败，请重试");
				} finally {
					this.creatingProject = false;
				}
			},
			async toggleStarProject(project) {
				try {
					const response = await api.project_follow({
						id: project.id,
						type: project.followed ? "unfollow" : "follow"
					});

					if (response.errcode === 0) {
						_.$msg.success(project.followed ? "取消星标成功" : "星标成功");
						// 更新本地状态
						project.followed = !project.followed;
						// 重新加载数据以更新星标项目列表
						this.loadApiData();
					} else {
						_.$msg.error(response.errmsg || "操作失败");
					}
				} catch (error) {
					console.error("Failed to toggle star:", error);
					_.$msg.error("操作失败，请重试");
				}
			}
		}
	};
}
</script>

<template>
	<div class="api-manager" @click="hideContextMenu">
		<!-- Top Bar -->
		<div class="api-manager__topbar">
			<!-- Toolbar Row -->
			<div class="api-manager__toolbar">
				<div class="api-manager__nav">
					<button
						class="api-manager__toolbar-btn api-manager__toolbar-btn--disabled"
						type="button">
						<xIcon icon="left" :size="20" />
					</button>
					<button
						class="api-manager__toolbar-btn api-manager__toolbar-btn--disabled"
						type="button">
						<xIcon icon="right" :size="20" />
					</button>
					<button
						@click="handleNavigateUp"
						:disabled="!canNavigateUp"
						class="api-manager__toolbar-btn"
						:class="{ 'api-manager__toolbar-btn--disabled': !canNavigateUp }"
						title="Up">
						<xIcon icon="top" :size="20" />
					</button>
				</div>

				<div class="api-manager__search">
					<div class="api-manager__search-icon" aria-hidden="true">
						<xIcon icon="search" :size="16" />
					</div>
					<input
						type="text"
						placeholder="Search resources..."
						v-model="searchQuery"
						class="api-manager__search-input" />
				</div>

				<div class="api-manager__toolbar-right">
					<!-- Environment Switcher -->
					<div class="api-manager__select-shell api-manager__select-shell--toolbar">
						<xIcon icon="tools" :size="16" class="api-manager__select-icon" />
						<select v-model="activeEnvironment" class="api-manager__select">
							<option v-for="env in environments" :key="env" :value="env"
								>{{ env }}
							</option>
						</select>
					</div>

					<div class="api-manager__divider" aria-hidden="true"></div>

					<div class="api-manager__view-toggle">
						<button
							@click="viewMode = 'list'"
							class="api-manager__view-toggle-btn"
							:class="{ 'api-manager__view-toggle-btn--active': viewMode === 'list' }"
							title="List View">
							<xIcon icon="list" :size="18" />
						</button>
						<button
							@click="viewMode = 'grid'"
							class="api-manager__view-toggle-btn"
							:class="{ 'api-manager__view-toggle-btn--active': viewMode === 'grid' }"
							title="Grid View">
							<xIcon icon="grid" :size="18" />
						</button>
					</div>

					<button
						@click="showPreview = !showPreview"
						class="api-manager__toggle-btn"
						:class="{ 'api-manager__toggle-btn--active': showPreview }"
						title="Toggle Preview Pane">
						<xIcon icon="view" :size="20" />
					</button>
				</div>
			</div>
		</div>

		<div class="api-manager__body">
			<!-- Sidebar (Tree) -->
			<div class="api-manager__sidebar">
				<div class="api-manager__sidebar-inner">
					<h2 class="api-manager__sidebar-title">API Explorer</h2>
					<!-- Recursive Tree Component inline simulation -->
					<div class="api-manager__tree">
						<!-- Loading State -->
						<div v-if="isLoading" class="api-manager__loading">
							<xIcon type="loader-2" :size="24" class="api-manager__loading-icon" />
							<span>加载中...</span>
						</div>

						<!-- Error State -->
						<div v-else-if="error" class="api-manager__error">
							<xIcon type="alert-circle" :size="24" class="api-manager__error-icon" />
							<span>{{ error }}</span>
							<button @click="loadApiData" class="api-manager__retry-btn"
								>重试</button
							>
						</div>

						<!-- Root Level - Personal Space and Groups -->
						<div v-else v-for="rootItem in apiData" :key="rootItem.id">
							<div
									class="api-manager__tree-item"
									:class="{
										'api-manager__tree-item--active': activeNode?.id === rootItem.id
									}"
									style="padding-left: 8px"
									@click.stop="handleOpenNode(rootItem)"
									@contextmenu="showContextMenu(rootItem, $event)">
								<span
									class="api-manager__tree-toggle"
									@click.stop="e => toggleFolder(rootItem.id, e)">
									<template
										v-if="
											isFolderType(rootItem.type) && rootItem.children?.length
										">
										<xIcon
											type="chevron-down"
											v-if="expandedFolders.has(rootItem.id)"
											:size="14" />
										<xIcon type="chevron-right" v-else :size="14" />
									</template>
								</span>
								<span class="api-manager__tree-icon">
									<xIcon
										:type="getIcon(rootItem.type)"
										:size="16"
										:class="getIconColor(rootItem.type)" />
								</span>
								<span class="api-manager__tree-label">{{ rootItem.name }}</span>
							</div>

							<!-- Level 1 -->
							<div v-if="expandedFolders.has(rootItem.id) && rootItem.children">
								<div :key="child.id" v-for="child in rootItem.children">
									<div
										class="api-manager__tree-item"
										:class="{
											'api-manager__tree-item--active':
												activeNode?.id === child.id
										}"
										style="padding-left: 20px"
										@click.stop="handleOpenNode(child)"
										@contextmenu="showContextMenu(child, $event)">
										<span
											class="api-manager__tree-toggle"
											@click.stop="e => toggleFolder(child.id, e)">
											<template
												v-if="
													isFolderType(child.type) &&
													child.children?.length
												">
												<xIcon
													type="chevron-down"
													v-if="expandedFolders.has(child.id)"
													:size="14" />
												<xIcon type="chevron-right" v-else :size="14" />
											</template>
										</span>
										<span class="api-manager__tree-icon">
											<xIcon
												:type="getIcon(child.type)"
												:size="16"
												:class="getIconColor(child.type, child)" />
										</span>
										<span class="api-manager__tree-label">
											{{ child.name }}
											<!-- Personal Project Badge -->
											<span
												v-if="
													child.type === 'project' &&
													child.path.includes('/个人空间/我的项目/')
												"
												class="api-manager__project-badge api-manager__project-badge--personal"
												title="个人项目"
												>个人</span
											>
											<!-- Starred Project Badge -->
											<span
												v-if="
													child.type === 'project' &&
													child.path.includes('/个人空间/星标项目/')
												"
												class="api-manager__project-badge api-manager__project-badge--starred"
												title="星标项目"
												>星标</span
											>
											<!-- Group Project Badge -->
											<span
												v-if="
													child.type === 'project' &&
													child.path.includes('/群组/')
												"
												class="api-manager__project-badge api-manager__project-badge--group"
												title="群组项目"
												>群组</span
											>
										</span>
										<!-- Add Project Button for Personal Space My Projects -->
										<span
											v-if="child.id === 'ps_my_projects'"
											class="api-manager__tree-actions">
											<button
												@click.stop="openCreateProjectDialog()"
												class="api-manager__tree-action-btn"
												title="创建项目"
												><xIcon icon="plus" :size="14"
											/></button>
										</span>
										<!-- Star Button for Projects -->
										<span
											v-if="child.type === 'project'"
											class="api-manager__tree-actions">
											<button
												@click.stop="toggleStarProject(child)"
												class="api-manager__tree-action-btn"
												:title="child.followed ? '取消星标' : '星标'">
												<xIcon
													:type="child.followed ? 'star' : 'star-o'"
													:size="14"
													:class="
														child.followed
															? 'api-manager__star-icon--active'
															: ''
													" />
											</button>
										</span>
									</div>

									<!-- Level 2 -->
									<div v-if="expandedFolders.has(child.id) && child.children">
										<div :key="subchild.id" v-for="subchild in child.children">
											<div
												class="api-manager__tree-item"
												:class="{
													'api-manager__tree-item--active':
														activeNode.id === subchild.id
												}"
												style="padding-left: 32px"
												@click.stop="handleOpenNode(subchild)"
												@contextmenu="showContextMenu(subchild, $event)">
												<span
													class="api-manager__tree-toggle"
													@click.stop="e => toggleFolder(subchild.id, e)">
													<template
														v-if="
															isFolderType(subchild.type) &&
															subchild.children?.length
														">
														<xIcon
															type="chevron-down"
															v-if="expandedFolders.has(subchild.id)"
															:size="14" />
														<xIcon
															type="chevron-right"
															v-else
															:size="14" />
													</template>
												</span>
												<span class="api-manager__tree-icon">
													<xIcon
														:type="getIcon(subchild.type)"
														:size="16"
														:class="getIconColor(subchild.type)" />
												</span>
												<span class="api-manager__tree-label">{{
													subchild.name
												}}</span>
											</div>

											<!-- Level 3 -->
											<div
												v-if="
													expandedFolders.has(subchild.id) &&
													subchild.children
												">
												<div
													:key="subsubchild.id"
													v-for="subsubchild in subchild.children">
													<div
														class="api-manager__tree-item"
														:class="{
															'api-manager__tree-item--active':
																activeNode.id === subsubchild.id
														}"
														style="padding-left: 44px"
														@click.stop="handleOpenNode(subsubchild)"
														@contextmenu="
															showContextMenu(subsubchild, $event)
														">
														<span
															class="api-manager__tree-toggle"
															@click.stop="
																e => toggleFolder(subsubchild.id, e)
															">
															<template
																v-if="
																	isFolderType(
																		subsubchild.type
																	) &&
																	subsubchild.children?.length
																">
																<xIcon
																	type="chevron-down"
																	v-if="
																		expandedFolders.has(
																			subsubchild.id
																		)
																	"
																	:size="14" />
																<xIcon
																	type="chevron-right"
																	v-else
																	:size="14" />
															</template>
														</span>
														<span class="api-manager__tree-icon">
															<xIcon
																:type="getIcon(subsubchild.type)"
																:size="16"
																:class="
																	getIconColor(subsubchild.type)
																" />
														</span>
														<span class="api-manager__tree-label">{{
															subsubchild.name
														}}</span>
													</div>

													<!-- Level 4 -->
													<div
														v-if="
															expandedFolders.has(subsubchild.id) &&
															subsubchild.children
														">
														<template>
															<div
																:key="leaf.id"
																v-for="leaf in subsubchild.children">
																<div
																	class="api-manager__tree-item"
																	:class="{
																		'api-manager__tree-item--active':
																			activeNode.id ===
																			leaf.id
																	}"
																	style="padding-left: 56px"
																	@click.stop="
																		handleOpenNode(leaf)
																	"
																	@contextmenu="
																		showContextMenu(
																			leaf,
																			$event
																		)
																	">
																	<span
																		class="api-manager__tree-toggle api-manager__tree-toggle--spacer"
																		aria-hidden="true">
																	</span>
																	<span
																		class="api-manager__tree-icon">
																		<xIcon
																			:type="
																				getIcon(leaf.type)
																			"
																			:size="16"
																			:class="
																				getIconColor(
																					leaf.type
																				)
																			" />
																	</span>
																	<span
																		class="api-manager__tree-label"
																		>{{ leaf.name }}</span
																	>
																</div>
															</div>
														</template>
													</div>
													<!-- 修复：闭合 Level 4 -->
												</div>
												<!-- 修复：闭合 Level 3 -->
											</div>
											<!-- 修复：闭合 Level 2 -->
										</div>
										<!-- 修复：闭合 Level 1 子项循环 -->
									</div>
									<!-- 修复：闭合 Level 1 展开容器 -->
								</div>
								<!-- 修复：闭合根节点循环 -->
							</div>
						</div>
					</div>
				</div>

				<!-- Create Project Dialog 修复：移到循环外部 -->
				<xDialog
					v-if="showCreateProjectDialog"
					title="创建个人项目"
					@close="closeCreateProjectDialog">
					<xForm col="1">
						<xItem
							:configs="createProjectFormConfigs.name"
							@input="
								val => {
									createProjectForm.name = val;
									createProjectFormConfigs.name.value = val;
								}
							" />
						<xItem
							:configs="createProjectFormConfigs.description"
							@input="
								val => {
									createProjectForm.description = val;
									createProjectFormConfigs.description.value = val;
								}
							" />
						<xItem
							:configs="createProjectFormConfigs.visibility"
							@input="
								val => {
									createProjectForm.visibility = val;
									createProjectFormConfigs.visibility.value = val;
								}
							" />
					</xForm>
					<template #footer>
						<xBtn @click="closeCreateProjectDialog">取消</xBtn>
						<xBtn
							:configs="{
								label: '创建',
								preset: 'blue',
								disabled: creatingProject,
								onClick: createProject
							}" />
					</template>
				</xDialog>

				<!-- Main Area -->
				<div class="api-manager__main">
					<!-- Breadcrumb -->
					<div class="api-manager__breadcrumb">
							<div
								v-for="(part, index) in (activeNode?.path || '').split('/').filter(Boolean)"
								:key="index">
								<span
									class="api-manager__breadcrumb-part"
									:class="{
										'api-manager__breadcrumb-part--current':
											index ===
											(activeNode?.path || '').split('/').filter(Boolean).length - 1
									}">
									{{ part }}
								</span>
								<span
									v-if="index < (activeNode?.path || '').split('/').filter(Boolean).length - 1"
									class="api-manager__breadcrumb-sep"
									>/</span
								>
							</div>
						</div>

					<!-- Folder View -->
					<template v-if="isFolderType(activeNode.type)">
						<!-- Table Header (only for list view) -->
						<div v-if="viewMode === 'list'" class="api-manager__table-header">
							<div
								class="api-manager__table-header-cell api-manager__table-header-cell--name"
								@click="handleSort('name')">
								Name
								<xIcon
									type="arrow-up"
									v-if="sortField === 'name' && sortDirection === 'asc'"
									:size="12"
									class="api-manager__sort-icon" />
								<xIcon
									type="arrow-down"
									v-if="sortField === 'name' && sortDirection === 'desc'"
									:size="12"
									class="api-manager__sort-icon" />
							</div>
							<div
								class="api-manager__table-header-cell api-manager__table-header-cell--date"
								@click="handleSort('updatedAt')">
								Date Modified
								<xIcon
									type="arrow-up"
									v-if="sortField === 'updatedAt' && sortDirection === 'asc'"
									:size="12"
									class="api-manager__sort-icon" />
								<xIcon
									type="arrow-down"
									v-if="sortField === 'updatedAt' && sortDirection === 'desc'"
									:size="12"
									class="api-manager__sort-icon" />
							</div>
							<div
								class="api-manager__table-header-cell api-manager__table-header-cell--type"
								@click="handleSort('type')">
								Type
								<xIcon
									type="arrow-up"
									v-if="sortField === 'type' && sortDirection === 'asc'"
									:size="12"
									class="api-manager__sort-icon" />
								<xIcon
									type="arrow-down"
									v-if="sortField === 'type' && sortDirection === 'desc'"
									:size="12"
									class="api-manager__sort-icon" />
							</div>
						</div>

						<!-- File List (List View) -->
						<div
							v-if="viewMode === 'list'"
							class="api-manager__file-list"
							@click="selectedFile = null">
							<div
								v-if="filteredAndSortedFiles.length === 0"
								class="api-manager__empty-state">
								<xIcon icon="folder" :size="48" class="api-manager__empty-icon" />
								<p>This folder is empty</p>
							</div>
							<div v-else class="api-manager__file-list-inner">
								<div
									v-for="file in filteredAndSortedFiles"
									:key="file.id"
									@click.stop="selectedFile = file"
									@dblclick.stop="handleOpenNode(file)"
									class="api-manager__file-row"
									:class="{
										'api-manager__file-row--selected':
											selectedFile?.id === file.id
									}">
									<div
										class="api-manager__file-cell api-manager__file-cell--name">
										<xIcon
											:type="getIcon(file.type)"
											:size="20"
											:class="getIconColor(file.type)" />
										<span class="api-manager__file-name">{{ file.name }}</span>
									</div>
									<div
										class="api-manager__file-cell api-manager__file-cell--date">
										{{ formatDate(file.updatedAt) }}
									</div>
									<div
										class="api-manager__file-cell api-manager__file-cell--type">
										{{ file.type.replace("_", " ") }}
									</div>
								</div>
							</div>
						</div>

						<!-- Grid View -->
						<div v-if="viewMode === 'grid'" class="api-manager__file-grid">
							<div
								v-if="filteredAndSortedFiles.length === 0"
								class="api-manager__empty-state">
								<xIcon icon="folder" :size="48" class="api-manager__empty-icon" />
								<p>This folder is empty</p>
							</div>
							<div v-else class="api-manager__file-grid-inner">
								<div
									v-for="file in filteredAndSortedFiles"
									:key="file.id"
									@click.stop="selectedFile = file"
									@dblclick.stop="handleOpenNode(file)"
									class="api-manager__file-card"
									:class="{
										'api-manager__file-card--selected':
											selectedFile?.id === file.id
									}">
									<div class="api-manager__file-card-icon">
										<xIcon
											:type="getIcon(file.type)"
											:size="32"
											:class="getIconColor(file.type)" />
									</div>
									<div class="api-manager__file-card-content">
										<h4 class="api-manager__file-card-name">{{ file.name }}</h4>
										<p class="api-manager__file-card-meta">
											{{ file.type.replace("_", " ") }}
										</p>
										<p class="api-manager__file-card-date">
											{{ formatDate(file.updatedAt) }}
										</p>
									</div>
								</div>
							</div>
						</div>
					</template>

					<!-- File Editor View -->
					<template v-else>
						<div class="api-manager__editor-wrap">
							<div class="api-manager__editor-card api-manager__editor-card--wide">
								<!-- Editor Header -->
								<div class="api-manager__editor-header">
									<div class="api-manager__editor-header-left">
										<xIcon
											:type="getIcon(activeNode.type)"
											:size="24"
											:class="getIconColor(activeNode.type)" />
										<div class="api-manager__editor-header-copy">
											<h2 class="api-manager__editor-title">{{
												activeNode.name
											}}</h2>
											<p class="api-manager__editor-subtitle">{{
												activeNode.type.replace("_", " ")
											}}</p>
										</div>
									</div>
									<div class="api-manager__editor-header-actions">
										<template v-if="editingContent">
											<button
												@click="cancelEditing"
												class="api-manager__action-btn api-manager__action-btn--ghost"
												>Cancel
											</button>
											<button
												@click="saveEditing"
												class="api-manager__action-btn api-manager__action-btn--primary api-manager__action-btn--with-icon">
												<xIcon icon="save" :size="16" />
												Save
											</button>
										</template>
										<template v-else>
											<button
												@click="startEditing"
												class="api-manager__action-btn api-manager__action-btn--secondary api-manager__action-btn--with-icon">
												<xIcon icon="edit" :size="16" />
												Edit
											</button>
										</template>
									</div>
								</div>

								<!-- Editor Content -->
								<div class="api-manager__editor-body">
									<!-- Member List Editor -->
									<template v-if="activeNode.type === 'member_list'">
										<div class="api-manager__section">
											<div class="api-manager__section-header">
												<h3 class="api-manager__section-title">Members</h3>
												<button
													v-if="editingContent"
													class="api-manager__link-btn api-manager__link-btn--with-icon">
													<xIcon icon="plus" :size="16" />
													Add Member
												</button>
											</div>
											<div class="api-manager__table-shell">
												<table class="api-manager__table">
													<thead class="api-manager__table-head">
														<tr>
															<th class="api-manager__th">Name</th>
															<th class="api-manager__th">Role</th>
															<th class="api-manager__th">Email</th>
															<th
																v-if="editingContent"
																class="api-manager__th api-manager__th--right"
																>Actions
															</th>
														</tr>
													</thead>
													<tbody>
														<tr
															v-for="(
																member, idx
															) in editingContent ||
															activeNode.content"
															:key="idx"
															class="api-manager__tr">
															<td
																class="api-manager__td api-manager__td--strong">
																<input
																	v-if="editingContent"
																	v-model="member.name"
																	class="api-manager__field-input api-manager__field-input--inline" />
																<span v-else>{{
																	member.name
																}}</span>
															</td>
															<td class="api-manager__td">
																<select
																	v-if="editingContent"
																	v-model="member.role"
																	class="api-manager__field-input api-manager__field-input--inline">
																	<option>Admin</option>
																	<option>Developer</option>
																	<option>Viewer</option>
																	<option>Owner</option>
																</select>
																<span
																	v-else
																	class="api-manager__inline-badge"
																	>{{ member.role }}</span
																>
															</td>
															<td
																class="api-manager__td api-manager__td--muted">
																<input
																	v-if="editingContent"
																	v-model="member.email"
																	class="api-manager__field-input api-manager__field-input--inline" />
																<span v-else>{{
																	member.email || "--"
																}}</span>
															</td>
															<td
																v-if="editingContent"
																class="api-manager__td api-manager__td--right">
																<button
																	class="api-manager__icon-action">
																	<xIcon
																		icon="delete"
																		:size="16" />
																</button>
															</td>
														</tr>
													</tbody>
												</table>
											</div>
										</div>
									</template>

									<!-- Settings Editor -->
									<template v-else-if="activeNode.type === 'setting'">
										<div class="api-manager__settings">
											<div
												v-for="(val, key) in editingContent ||
												activeNode.content"
												:key="key"
												class="api-manager__setting-row">
												<label class="api-manager__setting-label">{{
													String(key)
														.replace(/([A-Z])/g, " $1")
														.trim()
												}}</label>
												<div class="api-manager__setting-control">
													<input
														v-if="editingContent"
														v-model="editingContent[key]"
														class="api-manager__field-input api-manager__field-input--block" />
													<div v-else class="api-manager__field-value"
														>{{ val }}
													</div>
												</div>
											</div>
										</div>
									</template>

									<!-- API Editor -->
									<template v-else-if="activeNode.type === 'api'">
										<div class="api-manager__api-editor">
											<div class="api-manager__api-row">
												<select
													v-if="editingContent"
													v-model="editingContent.method"
													class="api-manager__field-input api-manager__method-select">
													<option>GET </option>
													<option>POST </option>
													<option>PUT </option>
													<option>DELETE</option>
												</select>
												<span
													v-else
													:class="
														getMethodColor(activeNode.content?.method)
													">
													{{ activeNode.content?.method }}
												</span>

												<input
													v-if="editingContent"
													v-model="editingContent.endpoint"
													class="api-manager__field-input api-manager__field-input--code api-manager__endpoint-input" />
												<span v-else class="api-manager__endpoint-text">{{
													activeNode.content?.endpoint
												}}</span>

												<!-- Send Button -->
												<button
													v-if="!editingContent"
													@click="sendRequest"
													:disabled="isRequesting"
													class="api-manager__action-btn api-manager__action-btn--primary api-manager__action-btn--with-icon">
													<xIcon
														type="loader-2"
														v-if="isRequesting"
														:size="16"
														class="api-manager__spinner" />
													<xIcon icon="video-play" v-else :size="16" />
													Send
												</button>
											</div>

											<div class="api-manager__field">
												<label class="api-manager__field-label"
													>Description</label
												>
												<textarea
													v-if="editingContent"
													v-model="editingContent.description"
													rows="2"
													class="api-manager__field-input api-manager__field-input--textarea"></textarea>
												<p v-else class="api-manager__field-text">{{
													activeNode.content?.description
												}}</p>
											</div>

											<div
												v-if="
													editingContent?.params ||
													activeNode.content?.params
												">
												<label
													class="api-manager__field-label api-manager__field-label--spaced"
													>Parameters</label
												>
												<div class="api-manager__code-block">
													<pre>{{
														JSON.stringify(
															editingContent?.params ||
																activeNode.content?.params,
															null,
															2
														)
													}}</pre>
												</div>
											</div>

											<div
												v-if="
													editingContent?.body || activeNode.content?.body
												">
												<label
													class="api-manager__field-label api-manager__field-label--spaced"
													>Request Body</label
												>
												<div class="api-manager__code-block">
													<pre>{{
														editingContent?.body ||
														activeNode.content?.body
													}}</pre>
												</div>
											</div>

											<!-- Response Section -->
											<div
												v-if="!editingContent"
												class="api-manager__response">
												<div class="api-manager__response-header">
													<h3 class="api-manager__response-title">
														Response
														<span
															v-if="responseStatus"
															class="api-manager__status-badge">
															{{ responseStatus }} OK
														</span>
													</h3>
													<span
														v-if="responseTime"
														class="api-manager__response-meta"
														>{{ responseTime }} ms</span
													>
												</div>

												<div
													v-if="isRequesting"
													class="api-manager__state-panel api-manager__state-panel--loading">
													<xIcon
														type="loader-2"
														class="api-manager__spinner api-manager__spinner--lg" />
													<p class="api-manager__state-text"
														>Sending request to
														{{ activeEnvironment }}...</p
													>
												</div>
												<div
													v-else-if="responseData"
													class="api-manager__state-panel api-manager__state-panel--code">
													<pre>{{ responseData }}</pre>
												</div>
												<div
													v-else
													class="api-manager__state-panel api-manager__state-panel--dashed">
													Click "Send" to execute the request
												</div>
											</div>
										</div>
									</template>

									<!-- Doc Editor -->
									<template
										v-else-if="
											activeNode.type === 'doc' || activeNode.type === 'code'
										">
										<div class="api-manager__doc-editor">
											<textarea
												v-if="editingContent"
												v-model="editingContent"
												class="api-manager__field-input api-manager__field-input--code api-manager__doc-textarea"></textarea>
											<div v-else class="api-manager__doc-preview">
												<pre
													class="api-manager__code-block api-manager__code-block--wrap"
													>{{ activeNode.content }}</pre
												>
											</div>
										</div>
									</template>

									<!-- Default JSON Editor -->
									<template v-else>
										<textarea
											v-if="editingContent"
											:value="
												typeof editingContent === 'string'
													? editingContent
													: JSON.stringify(editingContent, null, 2)
											"
											@input="handlePlainTextInput"
											class="api-manager__field-input api-manager__field-input--code api-manager__json-textarea"></textarea>
										<pre
											v-else
											class="api-manager__code-block api-manager__code-block--scroll"
											>{{
												typeof activeNode.content === "string"
													? activeNode.content
													: JSON.stringify(activeNode.content, null, 2)
											}}</pre
										>
									</template>
								</div>
							</div>
						</div>
					</template>
				</div>

				<!-- Context Menu -->
				<div
					v-if="contextMenu.show"
					class="api-manager__context-menu"
					:style="{ left: contextMenu.x + 'px', top: contextMenu.y + 'px' }"
					@click.stop>
					<div
						class="api-manager__context-menu-item"
						@click="handleContextMenuAction('rename')">
						<xIcon icon="edit" :size="14" />
						<span>Rename</span>
					</div>
					<div
						class="api-manager__context-menu-item"
						@click="handleContextMenuAction('move')">
						<xIcon icon="move" :size="14" />
						<span>Move</span>
					</div>
					<div
						class="api-manager__context-menu-item api-manager__context-menu-item--danger"
						@click="handleContextMenuAction('delete')">
						<xIcon icon="delete" :size="14" />
						<span>Delete</span>
					</div>
					<div class="api-manager__context-menu-divider"></div>
					<div
						class="api-manager__context-menu-item"
						@click="handleContextMenuAction('copyLink')">
						<xIcon icon="copy" :size="14" />
						<span>Copy Link</span>
					</div>
					<div
						class="api-manager__context-menu-item"
						@click="handleContextMenuAction('openNewTab')">
						<xIcon icon="external-link" :size="14" />
						<span>Open in New Tab</span>
					</div>
				</div>

				<!-- Preview Pane (Only for Folders) -->
				<div
					v-if="showPreview && isFolderType(activeNode.type)"
					class="api-manager__preview">
					<div v-if="!selectedFile" class="api-manager__preview-empty">
						<xIcon
							icon="info-filled"
							:size="48"
							class="api-manager__preview-empty-icon" />
						<p>Select a resource to preview</p>
					</div>
					<div v-else class="api-manager__preview-inner">
						<!-- Preview Content -->
						<div class="api-manager__preview-hero">
							<xIcon
								:type="getIcon(selectedFile.type)"
								:size="60"
								:class="getIconColor(selectedFile.type)" />
						</div>

						<!-- Metadata -->
						<h3 class="api-manager__preview-title">
							{{ selectedFile.name }}
						</h3>
						<p class="api-manager__preview-subtitle">{{
							selectedFile.type.replace("_", " ")
						}}</p>

						<div class="api-manager__preview-meta">
							<div class="api-manager__preview-section">
								<h4 class="api-manager__preview-section-title"> Information </h4>
								<dl class="api-manager__preview-dl">
									<div class="api-manager__preview-dl-row">
										<dt class="api-manager__preview-dt">Modified</dt>
										<dd class="api-manager__preview-dd">
											{{ formatDate(selectedFile.updatedAt) }}
										</dd>
									</div>
									<div class="api-manager__preview-dl-row">
										<dt class="api-manager__preview-dt">Path</dt>
										<dd
											class="api-manager__preview-dd api-manager__preview-dd--break">
											{{ selectedFile.path }}
										</dd>
									</div>
								</dl>
							</div>

							<div v-if="selectedFile.content" class="api-manager__preview-section">
								<h4 class="api-manager__preview-section-title"> Quick Preview </h4>
								<div
									class="api-manager__code-block api-manager__code-block--preview">
									<pre class="api-manager__preview-code">{{
										typeof selectedFile.content === "string"
											? selectedFile.content
											: JSON.stringify(selectedFile.content, null, 2)
									}}</pre>
								</div>
							</div>

							<button
								v-if="!isFolderType(selectedFile.type)"
								@click="handleOpenNode(selectedFile)"
								class="api-manager__action-btn api-manager__action-btn--primary api-manager__preview-action">
								Open Editor
							</button>
						</div>
					</div>
				</div>
			</div>
		</div>
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
	font-family:
		-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans",
		"PingFang SC", "Hiragino Sans GB", "Microsoft Yahei", sans-serif;

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
	}

	&__sidebar {
		width: 256px;
		flex-shrink: 0;
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
		transition:
			background-color 0.15s ease,
			color 0.15s ease;

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

	&__main {
		flex: 1;
		display: flex;
		flex-direction: column;
		overflow: hidden;
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
		transition:
			background-color 0.15s ease,
			border-color 0.15s ease;

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

	&__th,
	&__td {
		padding: 12px 16px;
		text-align: left;
	}

	&__th--right,
	&__td--right {
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
		font-family:
			ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
			monospace;
	}

	&__endpoint-text {
		flex: 1;
		min-width: 240px;
		font-size: 16px;
		font-weight: 600;
		color: var(--color-on-surface);
		font-family:
			ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
			monospace;
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
		from {
			transform: rotate(0deg);
		}
		to {
			transform: rotate(360deg);
		}
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
		font-family:
			ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
			monospace;

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
		font-family:
			ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
			monospace;
	}

	&__state-panel {
		border-radius: 14px;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container-lowest);
		padding: 24px;
	}

	&__state-panel--loading,
	&__state-panel--dashed {
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
		font-family:
			ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
			monospace;
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
		font-family:
			ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
			monospace;
		font-size: 14px;
	}

	&__json-textarea {
		width: 100%;
		height: 16rem;
		padding: 12px;
		font-family:
			ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
			monospace;
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
		width: 120px;
		height: 120px;
		margin: 0 auto 24px;
		border-radius: 18px;
		background: var(--color-surface-variant);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		display: flex;
		align-items: center;
		justify-content: center;
	}

	&__preview-title {
		margin: 0 0 4px;
		font-size: 18px;
		font-weight: 700;
		color: var(--color-on-surface);
		text-align: center;
		word-break: break-word;
	}

	&__preview-subtitle {
		margin: 0 0 24px;
		font-size: 14px;
		font-weight: 600;
		color: var(--color-on-surface-variant);
		text-transform: capitalize;
		text-align: center;
	}

	&__preview-meta {
		display: flex;
		flex-direction: column;
		gap: 16px;
	}

	&__preview-section {
		padding-top: 16px;
		border-top: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	}

	&__preview-section-title {
		margin: 0 0 12px;
		font-size: 12px;
		font-weight: 800;
		text-transform: uppercase;
		letter-spacing: 0.08em;
		color: var(--color-on-surface-variant);
	}

	&__preview-dl {
		margin: 0;
		display: flex;
		flex-direction: column;
		gap: 10px;
		font-size: 14px;
	}

	&__preview-dl-row {
		display: grid;
		grid-template-columns: 1fr 2fr;
		gap: 8px;
		align-items: start;
	}

	&__preview-dt {
		color: var(--color-on-surface-variant);
	}

	&__preview-dd {
		margin: 0;
		font-weight: 600;
		color: var(--color-on-surface);
	}

	&__preview-dd--break {
		word-break: break-all;
	}

	&__preview-code {
		margin: 0;
		font-size: 12px;
		white-space: pre-wrap;
		color: var(--color-on-surface);
		font-family:
			ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New",
			monospace;
	}

	&__preview-action {
		display: block;
		width: 100%;
		margin-top: 16px;
	}

	&__toolbar-btn,
	&__toggle-btn {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		min-width: 32px;
		min-height: 32px;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		border-radius: 10px;
		background: var(--color-surface-container-lowest);
		color: var(--color-on-surface-variant);
		cursor: pointer;
		transition:
			background-color 0.2s ease,
			border-color 0.2s ease,
			color 0.2s ease;

		&:hover {
			background: var(--color-surface-container);
		}
	}

	&__toolbar-btn--disabled {
		opacity: 0.45;
		cursor: not-allowed;
		pointer-events: none;
	}

	&__toggle-btn--active {
		background: var(--color-primary-container);
		border-color: color-mix(in srgb, var(--color-primary) 22%, transparent);
		color: var(--color-on-primary-container);
	}

	&__view-toggle {
		display: flex;
		background: var(--color-surface-container);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		border-radius: 10px;
		overflow: hidden;
	}

	&__view-toggle-btn {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		min-width: 32px;
		min-height: 32px;
		border: 0;
		background: transparent;
		color: var(--color-on-surface-variant);
		cursor: pointer;
		transition:
			background-color 0.2s ease,
			color 0.2s ease;

		&:hover {
			background: var(--color-surface-variant);
		}

		&--active {
			background: var(--color-surface-container-lowest);
			color: var(--color-primary);
		}
	}

	&__search-input,
	&__field-input,
	&__select-shell {
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		border-radius: 10px;
		background: var(--color-surface-container-lowest);
		color: var(--color-on-surface);
		transition:
			border-color 0.2s ease,
			box-shadow 0.2s ease,
			background-color 0.2s ease;
	}

	&__search-input,
	&__field-input {
		outline: none;

		&:focus {
			border-color: var(--el-color-primary);
			box-shadow: 0 0 0 2px color-mix(in srgb, var(--el-color-primary) 16%, transparent);
		}
	}

	&__field-input--inline {
		padding: 0 0 4px;
		border-width: 0 0 1px;
		border-radius: 0;
		background: transparent;
	}

	&__field-input--code {
		line-height: 1.6;
	}

	&__select-shell {
		background: var(--color-surface-container);
	}

	&__select {
		width: 100%;
		border: 0;
		background: transparent;
		color: var(--color-on-surface);
		cursor: pointer;
		outline: none;
	}

	&__editor-card {
		background: var(--color-surface-container-lowest);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		border-radius: 16px;
		box-shadow: var(--el-box-shadow);
	}

	&__action-btn {
		min-height: 36px;
		padding: 0 16px;
		border-radius: 10px;
		border: 1px solid transparent;
		cursor: pointer;
		font-size: 14px;
		font-weight: 600;
		transition:
			background-color 0.2s ease,
			border-color 0.2s ease,
			color 0.2s ease,
			box-shadow 0.2s ease;

		&:disabled {
			opacity: 0.7;
			cursor: not-allowed;
		}
	}

	&__action-btn--primary {
		background: var(--el-color-primary);
		color: var(--color-on-primary);
		box-shadow: var(--el-box-shadow-light);

		&:hover {
			background: var(--el-color-primary-hover);
		}
	}

	&__action-btn--secondary {
		background: var(--color-surface-container-lowest);
		border-color: color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		color: var(--color-on-surface);

		&:hover {
			background: var(--color-surface-variant);
		}
	}

	&__action-btn--ghost {
		background: transparent;
		color: var(--color-on-surface-variant);

		&:hover {
			background: var(--color-surface-variant);
			color: var(--color-on-surface);
		}
	}

	&__preview-action {
		justify-content: center;
	}

	&__inline-badge,
	&__status-badge,
	&__method-badge {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		border-radius: 999px;
		font-weight: 700;
	}

	&__inline-badge {
		padding: 4px 10px;
		background: var(--color-surface-variant);
		color: var(--color-on-surface-variant);
		font-size: 12px;
	}

	&__status-badge,
	&__method-badge {
		padding: 4px 10px;
		font-size: 12px;
	}

	&__method-badge--get,
	&__icon--personal,
	&__icon--group,
	&__icon--member-list {
		color: var(--el-color-primary);
	}

	// Project type specific icons
	&__icon--project-personal {
		color: #409eff;
	}

	&__icon--project-starred {
		color: #e6a23c;
	}

	&__icon--project-group {
		color: #67c23a;
	}

	// Project badges
	&__project-badge {
		display: inline-block;
		padding: 0 8px;
		margin-left: 8px;
		font-size: 12px;
		line-height: 18px;
		border-radius: 10px;
		font-weight: 500;
	}

	&__project-badge--personal {
		background-color: #ecf5ff;
		color: #409eff;
	}

	&__project-badge--starred {
		background-color: #fdf6ec;
		color: #e6a23c;
	}

	&__project-badge--group {
		background-color: #f0f9eb;
		color: #67c23a;
	}

	// Star icon active state
	&__star-icon--active {
		color: #e6a23c;
	}

	// Tree actions
	&__tree-actions {
		display: inline-flex;
		align-items: center;
		margin-left: auto;
		opacity: 0;
		transition: opacity 0.2s;
	}

	&__tree-item:hover &__tree-actions {
		opacity: 1;
	}

	&__tree-action-btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 20px;
		height: 20px;
		border: none;
		background: transparent;
		border-radius: 4px;
		cursor: pointer;
		transition: background-color 0.2s;
		margin-left: 4px;
	}

	&__tree-action-btn:hover {
		background-color: rgba(0, 0, 0, 0.05);
	}

	// Loading and error states
	&__loading {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 40px 0;
		color: #909399;
	}

	&__loading-icon {
		margin-right: 10px;
		animation: spin 1s linear infinite;
	}

	@keyframes spin {
		from {
			transform: rotate(0deg);
		}
		to {
			transform: rotate(360deg);
		}
	}

	&__error {
		display: flex;
		align-items: center;
		justify-content: center;
		padding: 40px 0;
		color: #f56c6c;
		flex-direction: column;
	}

	&__error-icon {
		margin-right: 10px;
		margin-bottom: 10px;
	}

	&__retry-btn {
		margin-top: 10px;
		padding: 4px 12px;
		border: 1px solid #dcdfe6;
		border-radius: 4px;
		background: #ffffff;
		color: #409eff;
		cursor: pointer;
		transition: all 0.2s;
	}

	&__retry-btn:hover {
		background: #ecf5ff;
		border-color: #c6e2ff;
	}

	&__method-badge--get {
		background: var(--el-color-primary-light-9);
	}

	&__method-badge--post,
	&__icon--api-folder,
	&__icon--api {
		color: var(--el-color-success-dark-2);
	}

	&__method-badge--post {
		background: var(--el-color-success-light-9);
	}

	&__method-badge--put,
	&__icon--doc-folder,
	&__icon--doc {
		color: var(--el-color-warning-dark-2);
	}

	&__method-badge--put {
		background: var(--el-color-warning-light-9);
	}

	&__method-badge--delete {
		background: var(--el-color-danger-light-9);
		color: var(--el-color-danger);
	}

	&__method-badge--default,
	&__icon--folder,
	&__icon--setting,
	&__icon--default {
		background: var(--el-fill-color-light);
		color: var(--color-on-surface-variant);
	}

	&__icon--project {
		color: var(--el-color-primary-dark-2);
	}

	&__icon--log {
		color: var(--el-color-info);
	}

	&__icon--code {
		color: var(--el-color-warning-dark-2);
	}

	&__icon--cicd {
		color: var(--el-color-info-dark-2);
	}

	&__status-badge {
		background: var(--el-color-success-light-9);
		color: var(--el-color-success-dark-2);
	}

	&__state-panel,
	&__code-block,
	&__field-value {
		background: var(--color-surface-container-lowest);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		border-radius: 12px;
	}

	&__state-panel {
		color: var(--color-on-surface-variant);
	}

	&__state-panel--code {
		box-shadow: inset 0 1px 3px rgba(15, 23, 42, 0.08);
	}

	&__state-panel--dashed {
		border-style: dashed;
	}

	&__empty-state {
		color: var(--color-on-surface-variant);
	}

	&__icon-action {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		width: 28px;
		height: 28px;
		border: 0;
		border-radius: 8px;
		background: transparent;
		color: var(--el-color-danger);
		cursor: pointer;

		&:hover {
			background: var(--el-color-danger-light-9);
		}
	}

	&__context-menu {
		position: fixed;
		z-index: 1000;
		min-width: 160px;
		background: var(--color-surface-container-lowest);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		border-radius: 12px;
		box-shadow: var(--el-box-shadow);
		overflow: hidden;
	}

	&__context-menu-item {
		display: flex;
		align-items: center;
		gap: 8px;
		padding: 8px 12px;
		font-size: 14px;
		color: var(--color-on-surface);
		cursor: pointer;
		transition: background-color 0.15s ease;

		&:hover {
			background: var(--color-surface-variant);
		}

		&--danger {
			color: var(--el-color-danger);

			&:hover {
				background: var(--el-color-danger-light-9);
			}
		}
	}

	&__context-menu-divider {
		height: 1px;
		margin: 4px 0;
		background: color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
	}

	&__file-grid {
		flex: 1;
		overflow-y: auto;
	}

	&__file-grid-inner {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
		gap: 16px;
		padding: 16px;
	}

	&__file-card {
		display: flex;
		flex-direction: column;
		padding: 16px;
		border-radius: 14px;
		background: var(--color-surface-container-lowest);
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 30%, transparent);
		cursor: pointer;
		transition:
			background-color 0.15s ease,
			border-color 0.15s ease;

		&:hover {
			background: color-mix(in srgb, var(--color-surface-variant) 30%, transparent);
			border-color: color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		}

		&--selected {
			background: color-mix(in srgb, var(--color-primary-container) 30%, transparent);
			border-color: color-mix(in srgb, var(--color-primary) 22%, transparent);
		}
	}

	&__file-card-icon {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 48px;
		height: 48px;
		border-radius: 12px;
		background: var(--color-surface-variant);
		margin-bottom: 12px;
		align-self: flex-start;
	}

	&__file-card-content {
		flex: 1;
		display: flex;
		flex-direction: column;
		gap: 4px;
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
		margin-top: auto;
	}
}
</style>
