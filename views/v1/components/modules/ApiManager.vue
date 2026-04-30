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
			ApiManagerProjectList: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerProjectList.vue"),
			ApiManagerEditor: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerEditor.vue"),
			ApiManagerPreview: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerPreview.vue"),
			ApiManagerContextMenu: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerContextMenu.vue"),
			ApiManagerCreateProjectDialog: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerCreateProjectDialog.vue")
		},
		provide() {
			return {
				_onProjectFollowChange: (projectId, isFollow) => {
					this.handleFollowChange(projectId, isFollow);
				}
			};
		},
		props: {
			windowData: Object
		},
		inject: {
			system: {
				default: null
			},
			APP: {
				default: null
			}
		},
		data() {
			return {
				apiData: null,
				selectedFile: null,
				searchQuery: "",
				showPreview: false,
				viewMode: "list",
				expandedFolders: utils.loadExpandedFolders(),
				selectedNodeId: utils.loadSelectedNodeId(),
				isLoading: false,
				error: null,
				sortField: "name",
				sortDirection: "asc",
				isRequesting: false,
				responseData: null,
				responseStatus: null,
				responseTime: null,
				editingContent: null,
				contextMenu: { show: false, x: 0, y: 0, node: null },
				showCreateProjectDialog: false,
				creatingProject: false,
				searchResults: [],
				isSearchLoading: false,
				searchTimeout: null
			};
		},
		computed: {
			activeNode() {
				if (this.windowData) return this.windowData;
				const findNode = (nodes, targetId) => {
					for (const node of nodes) {
						if (node.id === targetId) return node;
						if (node.children && node.children.length > 0) {
							const found = findNode(node.children, targetId);
							if (found) return found;
						}
					}
					return null;
				};
				return findNode(this.apiData || [], this.selectedNodeId) ||
					{ id: "personal_space", name: "个人空间", type: "personal", path: "/个人空间", children: [] };
			},
			filteredAndSortedFiles() {
				console.log("activeNode:", this.activeNode);
				console.log("activeNode type:", this.activeNode?.type);
				console.log("activeNode id:", this.activeNode?.id);
				console.log("isFolderType:", utils.isFolderType(this.activeNode?.type));
				if (!utils.isFolderType(this.activeNode?.type)) return [];
				let files = this.activeNode?.children || [];
				console.log("activeNode children count:", files.length);
				console.log("activeNode children:", files);
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
			handleUrlParams() {
				const params = new URLSearchParams(window.location.search);
				const nodeId = params.get('nodeId');
				if (nodeId) {
					this.selectedNodeId = nodeId;
					utils.saveSelectedNodeId(nodeId);
				}
				if (this.windowData) {
					this.expandAncestors(this.windowData.id);
					this.loadProjectFolderDataIfNeeded(this.windowData);
				} else if (nodeId) {
					this.$nextTick(() => {
						this.expandAncestors(nodeId);
						const node = this.findNodeById(this.apiData, nodeId);
						if (node) {
							this.loadProjectFolderDataIfNeeded(node);
						}
					});
				}
			},
			findNodeById(nodes, targetId) {
				for (const node of nodes) {
					if (node.id === targetId) return node;
					if (node.children && node.children.length > 0) {
						const found = this.findNodeById(node.children, targetId);
						if (found) return found;
					}
				}
				return null;
			},
			loadProjectFolderDataIfNeeded(node) {
				if (this.isProjectFolderType(node.type, node.id)) {
					this.loadProjectFolderData(node);
				}
			},
			async loadApiData() {
				this.isLoading = true;
				this.error = null;
				try {
					const groups = await api.groupMine();
					console.log("Groups data loaded:", groups);
					console.log("Groups data array:", groups.data);
					
					const groupList = (groups.data || []).filter(group => group._id && !group.is_personal && group.group_name !== '个人空间');
					console.log("Filtered groups count:", groupList.length);
					console.log("Filtered groups:", groupList);
					
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
							children: groupList.map(group => ({
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
									{ id: `group_${group._id}_projects`, name: "Projects", type: "folder", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Projects`, children: [] },
									{ id: `group_${group._id}_members`, name: "Group Members", type: "member_list", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Group Members`, content: [] },
									{ id: `group_${group._id}_docs`, name: "Group Docs", type: "doc_folder", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Group Docs`, children: [] },
									{
										id: `group_${group._id}_settings`, name: "Group Settings", type: "setting", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Group Settings`,
										content: {
											name: group.group_name, description: group.group_desc, visibility: group.type === "private" ? "private" : "public",
											defaultRole: group.default_role || "guest", allowExternalShare: group.allow_external_share || false
										}
									},
									{ id: `group_${group._id}_log`, name: "Activity Log", type: "log", updatedAt: new Date().toISOString(), path: `/群组/${group.group_name}/Activity Log`, content: [] }
								]
							}))
						}
					];
					
					console.log("API data structure after groups:", JSON.stringify(this.apiData, null, 2));
					
					const personalProjects = await api.getProjectByGroupId("personal");
					const allPersonalProjects = personalProjects.data?.list || [];
					console.log("Personal projects loaded:", allPersonalProjects);
					console.log("Personal projects count:", allPersonalProjects.length);
					
					const myProjects = allPersonalProjects.filter(project => !project.group_id);
					const starredProjects = allPersonalProjects.filter(project => project.follow);
					
					console.log("My projects count:", myProjects.length);
					console.log("Starred projects count:", starredProjects.length);
					
					this.apiData[0].children[0].children = myProjects.map(project => ({
						id: project._id || project.id, name: project.name, type: "project", role: project.role,
						updatedAt: project.up_time || project.updated_at, path: `/个人空间/我的项目/${project.name}`,
						visibility: project.project_type || project.visibility, followed: project.follow,
						content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API endpoints for ${project.name}` }
					}));
					
					this.apiData[0].children[1].children = starredProjects.map(project => ({
						id: project._id || project.id, name: project.name, type: "project", role: project.role,
						updatedAt: project.up_time || project.updated_at, path: `/个人空间/星标项目/${project.name}`,
						visibility: project.project_type || project.visibility, followed: project.follow,
						content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API endpoints for ${project.name}` }
					}));
					
					console.log("Final API data structure:", JSON.stringify(this.apiData, null, 2));
					
				} catch (error) {
					console.error("Failed to load API data:", error);
					this.error = "加载数据失败，请刷新页面重试";
				} finally {
					this.isLoading = false;
					this.handleUrlParams();
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
				this.loadProjectFolderData(node);
				const appId = node.type === "api" ? "api_endpoint" : node.type;
				if (this.system?.openApp) this.system.openApp(appId, true, node);
			},
			async loadProjectFolderData(node) {
				if (this.isPersonalSpace(node.type, node.id)) {
					console.log("Loading personal space data for node:", node.id, node.type);
					await this.loadPersonalSpaceProjects();
					return;
				}
				
				if (!this.isProjectFolderType(node.type, node.id)) {
					console.log("Not a project folder type:", node.type, node.id);
					return;
				}
				console.log("Loading project folder data for node:", node.id, node.type);
				
				if (node.id === "ps_my_projects") {
					if (this.apiData[0]?.children[0]?.children?.length === 0) {
						await this.loadPersonalSpaceProjects();
					}
					return;
				}
				
				if (node.id === "ps_followed") {
					if (this.apiData[0]?.children[1]?.children?.length === 0) {
						await this.loadPersonalSpaceProjects();
					}
					return;
				}
				
				const groupIdMatch = node.id.match(/group_([^_]+)_projects/);
				console.log("groupIdMatch:", groupIdMatch);
				if (groupIdMatch) {
					const groupId = groupIdMatch[1];
					console.log("Extracted groupId:", groupId);
					if (!groupId) {
						console.error("Invalid groupId extracted:", groupIdMatch[1]);
						return;
					}
					try {
						console.log("Calling api.getProjectByGroupId with groupId:", groupId);
						const projectData = await api.getProjectByGroupId(groupId);
						console.log("API response:", projectData);
						if (!projectData) {
							console.error("API returned null/undefined");
							return;
						}
						const projects = projectData.data?.list || [];
						console.log(`Loaded ${projects.length} projects for group ${groupId}`);
						const updateNode = (nodes) => {
							for (const n of nodes) {
								if (n.id === node.id) {
									n.children = projects.map(project => ({
										id: project._id || project.id, name: project.name, type: "project", role: project.role,
										updatedAt: project.up_time || project.updated_at, path: `${node.path}/${project.name}`,
										visibility: project.project_type || project.visibility, followed: project.follow,
										content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API endpoints for ${project.name}` }
									}));
									return true;
								}
								if (n.children && n.children.length > 0 && updateNode(n.children)) {
									return true;
								}
							}
							return false;
						};
						updateNode(this.apiData);
						console.log("Node updated successfully");
					} catch (error) {
						console.error(`Failed to load projects for group ${groupId}:`, error);
						console.error("Error details:", error.message, error.stack);
					}
				} else {
					console.error("Could not extract groupId from node.id:", node.id);
				}
			},
			async loadPersonalProjects(type) {
				try {
					console.log(`Loading ${type} projects...`);
					console.log("Calling api.getProjectByGroupId with params:", "personal");
					const personalProjects = await api.getProjectByGroupId("personal");
					console.log("API response:", personalProjects);
					console.log("Response data:", personalProjects.data);
					
					const allPersonalProjects = personalProjects.data?.list || [];
					console.log(`Personal projects loaded: ${allPersonalProjects.length}`);
					console.log("Project sample:", allPersonalProjects[0]);
					
					let projects;
					let nodeId;
					if (type === "myProjects") {
						projects = allPersonalProjects.filter(p => !p.group_id);
						nodeId = "ps_my_projects";
					} else {
						projects = allPersonalProjects.filter(p => p.follow);
						nodeId = "ps_followed";
					}
					
					console.log(`${type} count:`, projects.length);
					
					const updateNode = (nodes) => {
						for (const n of nodes) {
							if (n.id === nodeId) {
								n.children = projects.map(project => ({
									id: project._id || project.id, name: project.name, type: "project", role: project.role,
									updatedAt: project.up_time || project.updated_at, path: `/个人空间/${type === 'myProjects' ? '我的项目' : '星标项目'}/${project.name}`,
									visibility: project.project_type || project.visibility, followed: project.follow,
									content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API endpoints for ${project.name}` }
								}));
								return true;
							}
							if (n.children && n.children.length > 0 && updateNode(n.children)) {
								return true;
							}
						}
						return false;
					};
					updateNode(this.apiData);
					console.log(`${type} projects updated successfully`);
				} catch (error) {
					console.error(`Failed to load ${type} projects:`, error);
					console.error("Error details:", error.message, error.stack);
				}
			},
			async loadPersonalSpaceProjects() {
				try {
					console.log("Loading personal space projects...");
					const personalProjects = await api.getProjectByGroupId("personal");
					console.log("Personal space API response:", personalProjects);
					console.log("Personal space response data:", personalProjects.data);
					
					const allPersonalProjects = personalProjects.data?.list || [];
					console.log("All personal projects count:", allPersonalProjects.length);
					
					const myProjects = allPersonalProjects.filter(project => !project.group_id);
					const starredProjects = allPersonalProjects.filter(project => project.follow);
					
					console.log("My projects count:", myProjects.length);
					console.log("Starred projects count:", starredProjects.length);
					
					if (this.apiData[0] && this.apiData[0].children) {
						this.apiData[0].children[0].children = myProjects.map(project => ({
							id: project._id || project.id, name: project.name, type: "project", role: project.role,
							updatedAt: project.up_time || project.updated_at, path: `/个人空间/我的项目/${project.name}`,
							visibility: project.project_type || project.visibility, followed: project.follow,
							content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API endpoints for ${project.name}` }
						}));
						
						this.apiData[0].children[1].children = starredProjects.map(project => ({
							id: project._id || project.id, name: project.name, type: "project", role: project.role,
							updatedAt: project.up_time || project.updated_at, path: `/个人空间/星标项目/${project.name}`,
							visibility: project.project_type || project.visibility, followed: project.follow,
							content: { endpoint: `api/${project._id || project.id}/endpoints`, method: "GET", description: `API endpoints for ${project.name}` }
						}));
					}
					
					console.log("Personal space projects updated successfully");
				} catch (error) {
					console.error("Failed to load personal space projects:", error);
					console.error("Error details:", error.message, error.stack);
				}
			},
			openNewWindow(node) {
				const vm = this;
				_.$ModalManager.open({
					title: node.name,
					url: "@/views/v1/components/modules/api-manager/ApiManagerWindow.dialog.vue",
					parent: this,
					width: 800,
					height: 600,
					node: node,
					onClose() {
						console.log('Window closed');
					}
				});
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
						const params = new URLSearchParams();
						params.set('nodeId', node.id);
						if (node.path) params.set('path', node.path);
						if (node.type) params.set('type', node.type);
						const link = `${window.location.origin}/v1/api-manager?${params.toString()}`;
						window.open(link, "_blank");
						break;
					}
					case "openNewWindow": {
						this.openNewWindow(node);
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
						mockResponse = {
							status: "success", environment: this.activeEnvironment,
							data: [{ id: 1, name: "Alice Smith", email: "alice@example.com" },
							{ id: 2, name: "Bob Jones", email: "bob@example.com" }], meta: { total: 2, page: 1 }
						};
					} else if (endpoint.includes("users") && this.activeNode.content?.method === "POST") {
						mockResponse = {
							status: "success", environment: this.activeEnvironment,
							message: "User created successfully", data: { id: 3, name: "New User", email: "new@example.com" }
						};
					} else {
						mockResponse = {
							status: "success", environment: this.activeEnvironment,
							message: "Request executed successfully", timestamp: new Date().toISOString()
						};
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
				if (!formData.name.trim()) { _.$msgError("项目名称不能为空"); return; }
				this.creatingProject = true;
				try {
					const response = await api.project_add({ name: formData.name, desc: formData.description, visibility: formData.visibility });
					if (response.errcode === 0) {
						_.$msgSuccess("项目创建成功");
						this.showCreateProjectDialog = false;
						this.loadApiData();
					} else {
						_.$msgError(response.errmsg || "项目创建失败");
					}
				} catch (error) {
					console.error("Failed to create project:", error);
					_.$msgError("项目创建失败，请重试");
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
						_.$msgSuccess(project.followed ? "取消星标成功" : "星标成功");
						project.followed = !project.followed;
						this.loadApiData();
					} else {
						_.$msgError(response.errmsg || "操作失败");
					}
				} catch (error) {
					console.error("Failed to toggle star:", error);
					_.$msgError("操作失败，请重试");
				}
			},
			isFolderType(type) {
				return utils.isFolderType(type);
			},
			isProjectFolderType(type, nodeId) {
				console.log("isProjectFolderType - type:", type, "nodeId:", nodeId);
				const result = type === "folder" && (
					["ps_my_projects", "ps_followed"].includes(nodeId) || nodeId?.endsWith("_projects")
				);
				console.log("isProjectFolderType result:", result);
				return result;
			},
			isPersonalSpace(type, nodeId) {
				return type === "personal" && nodeId === "personal_space";
			},
			performSearch(query) {
				if (!query.trim()) {
					this.searchResults = [];
					return;
				}

				if (this.searchTimeout) {
					clearTimeout(this.searchTimeout);
				}

				this.isSearchLoading = true;

				this.searchTimeout = setTimeout(() => {
					const results = this.searchInTree(this.apiData, query.toLowerCase());
					this.searchResults = results;
					this.isSearchLoading = false;
				}, 300);
			},
			searchInTree(nodes, query) {
				if (!nodes || !nodes.length) return [];

				let results = [];

				for (const node of nodes) {
					if (node.name?.toLowerCase().includes(query)) {
						results.push({ ...node });
					}

					if (node.children && node.children.length > 0) {
						results = results.concat(this.searchInTree(node.children, query));
					}
				}

				return results.slice(0, 50);
			},
			handleSelectSearchResult(result) {
				this.handleOpenNode(result);
			},
			handleFollowChange(projectId, isFollow) {
				console.log(`Follow status changed for project ${projectId}: ${isFollow}`);
				this.loadPersonalSpaceProjects();
			}
		}
	};
}
</script>

<template>
	<div class="api-manager" @click="hideContextMenu">
		<ApiManagerToolbar
			:search-query="searchQuery"
			:can-navigate-up="canNavigateUp"
			:search-results="searchResults"
			:is-search-loading="isSearchLoading"
			@update:search-query="searchQuery = $event; performSearch($event)"
			@select-search-result="handleSelectSearchResult"
			@navigate-up="handleNavigateUp" />

		<div class="api-manager__body">
			<ApiManagerSidebar
				:api-data="apiData"
				:expanded-folders="expandedFolders"
				:active-node-id="selectedNodeId"
				:is-loading="isLoading"
				:error="error"
				@toggle-folder="toggleFolder"
				@open-node="handleOpenNode"
				@context-menu="showContextMenu"
				@create-project="openCreateProjectDialog"
				@toggle-star="toggleStarProject"
				@retry="loadApiData" />

			<div class="api-manager__content">
				<div class="api-manager__content-header">
					<ApiManagerBreadcrumb :path="activeNode?.path" @navigate-to="() => { }" />
					<div class="api-manager__content-actions">
						<div class="api-manager__view-toggle">
							<button 
								@click="viewMode = 'list'" 
								:class="{ 'api-manager__view-toggle-btn': true, 'api-manager__view-toggle-btn--active': viewMode === 'list' }"
								title="List View">
								<xIcon icon="list" :size="18" />
							</button>
							<button 
								@click="viewMode = 'grid'" 
								:class="{ 'api-manager__view-toggle-btn': true, 'api-manager__view-toggle-btn--active': viewMode === 'grid' }"
								title="Grid View">
								<xIcon icon="grid" :size="18" />
							</button>
						</div>
						<button 
							@click="showPreview = !showPreview" 
							:class="{ 'api-manager__toggle-btn': true, 'api-manager__toggle-btn--active': showPreview }"
							title="Toggle Preview Pane">
							<xIcon icon="view" :size="20" />
						</button>
					</div>
				</div>
				<div class="api-manager__main-wrapper">
					<div class="api-manager__main">

						<template v-if="isFolderType(activeNode?.type)">
							<ApiManagerProjectList
								v-if="isProjectFolderType(activeNode?.type, activeNode?.id)"
								:projects="filteredAndSortedFiles"
								:selected-project-id="selectedFile?.id"
								:sort-field="sortField"
								:sort-direction="sortDirection"
								:view-mode="viewMode"
								:search-query="searchQuery"
								@select-project="selectedFile = $event"
								@open-project="handleOpenNode"
								@sort="handleSort"
								@toggle-star="toggleStarProject"
								@context-menu="showContextMenu" />
							<template v-else>
								<ApiManagerFileList
									v-if="viewMode === 'list'"
									:files="filteredAndSortedFiles"
									:selected-file-id="selectedFile?.id"
									:sort-field="sortField"
									:sort-direction="sortDirection"
									@select-file="selectedFile = $event"
									@open-file="handleOpenNode"
									@sort="handleSort" />
								<ApiManagerFileGrid
									v-else
									:files="filteredAndSortedFiles"
									:selected-file-id="selectedFile?.id"
									@select-file="selectedFile = $event"
									@open-file="handleOpenNode" />
							</template>
						</template>

						<ApiManagerEditor v-else :node="activeNode" :editing-content="editingContent"
							:is-requesting="isRequesting"
							:response-data="responseData" :response-status="responseStatus" :response-time="responseTime"
							:active-environment="activeEnvironment" @start-edit="startEditing" @save-edit="saveEditing"
							@cancel-edit="cancelEditing" @send-request="sendRequest" />
					</div>

					<ApiManagerPreview v-if="showPreview && isFolderType(activeNode?.type)" :selected-file="selectedFile"
						:is-folder-type="selectedFile ? isFolderType(selectedFile.type) : false"
						@open-node="handleOpenNode" />
				</div>
			</div>
		</div>

		<ApiManagerContextMenu :show="contextMenu.show" :x="contextMenu.x" :y="contextMenu.y" :node="contextMenu.node"
			@action="handleContextMenuAction" @hide="hideContextMenu" />

		<ApiManagerCreateProjectDialog :show="showCreateProjectDialog" :creating-project="creatingProject"
			@close="closeCreateProjectDialog" @create="createProject" />
	</div>
</template>

<style lang="less">
.api-manager {
	height: 100%;
	width: 100%;
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
		padding: 6px 32px 6px 40px;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 8px;
		font-size: 14px;
		outline: none;
		background: var(--color-surface-container-low);
		color: var(--color-on-surface);

		&:focus {
			border-color: var(--color-primary);
			box-shadow: 0 0 0 2px rgba(49, 130, 206, 0.2);
		}

		&::placeholder {
			color: var(--color-on-surface-variant);
		}
	}

	&__search-clear {
		position: absolute;
		top: 0;
		bottom: 0;
		right: 8px;
		display: flex;
		align-items: center;
		justify-content: center;
		width: 24px;
		height: 24px;
		margin: auto;
		padding: 0;
		border: none;
		background: transparent;
		color: var(--color-on-surface-variant);
		cursor: pointer;
		border-radius: 6px;

		&:hover {
			color: var(--color-on-surface);
			background: var(--color-surface-variant);
		}
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
		display: flex;
		flex-direction: column;
	}

	&__sidebar-inner {
		flex: 1;
		overflow-y: auto;
		padding: 8px;
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
		&--personal {
			color: var(--color-primary);
		}

		&--group {
			color: var(--color-secondary);
		}

		&--project-personal {
			color: var(--color-primary);
		}

		&--project-starred {
			color: var(--color-warning);
		}

		&--project-group {
			color: var(--color-secondary);
		}

		&--project {
			color: var(--color-on-surface-variant);
		}

		&--api-folder {
			color: var(--color-primary);
		}

		&--doc-folder {
			color: var(--color-secondary);
		}

		&--folder {
			color: var(--color-on-surface-variant);
		}

		&--api {
			color: var(--color-primary);
		}

		&--doc {
			color: var(--color-secondary);
		}

		&--code {
			color: var(--color-warning);
		}

		&--member-list {
			color: var(--color-secondary);
		}

		&--setting {
			color: var(--color-on-surface-variant);
		}

		&--cicd {
			color: var(--color-success);
		}

		&--log {
			color: var(--color-on-surface-variant);
		}

		&--default {
			color: var(--color-on-surface-variant);
		}
	}

	&__content {
		flex: 1;
		display: flex;
		flex-direction: column;
		overflow: hidden;
		min-height: 0;
	}

	&__main-wrapper {
		flex: 1;
		display: flex;
		overflow: hidden;
		min-height: 0;
	}

	&__content-header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 8px 16px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container);
		flex-shrink: 0;
	}

	&__content-actions {
		display: flex;
		align-items: center;
		gap: 12px;
	}
}
</style>