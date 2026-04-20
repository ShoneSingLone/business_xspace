<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  const mockApiData = {
    id: "root",
    name: "API Workspaces",
    type: "folder",
    updatedAt: "2026-03-31T10:00:00Z",
    path: "/API Workspaces",
    children: [
      {
        id: "global_resources",
        name: "Global Resources",
        type: "folder",
        updatedAt: "2026-04-01T10:00:00Z",
        path: "/API Workspaces/Global Resources",
        children: [
          {
            id: "gr_models",
            name: "Data Models",
            type: "doc_folder",
            updatedAt: "2026-04-01T10:00:00Z",
            path: "/API Workspaces/Global Resources/Data Models",
            children: [
              {
                id: "gr_user_model",
                name: "UserSchema.json",
                type: "code",
                updatedAt: "2026-04-01T10:00:00Z",
                path: "/API Workspaces/Global Resources/Data Models/UserSchema.json",
                content:
                  '{\n  "type": "object",\n  "properties": {\n    "id": { "type": "string" },\n    "name": { "type": "string" },\n    "email": { "type": "string", "format": "email" }\n  },\n  "required": ["id", "name"]\n}',
              },
            ],
          },
        ],
      },
      {
        id: "group_backend",
        name: "Backend Team",
        type: "group",
        updatedAt: "2026-03-31T10:00:00Z",
        path: "/API Workspaces/Backend Team",
        children: [
          {
            id: "gb_info",
            name: "Group Info",
            type: "setting",
            updatedAt: "2026-03-31T10:00:00Z",
            path: "/API Workspaces/Backend Team/Group Info",
            content: {
              description: "Core backend services team handling microservices.",
              createdBy: "Admin",
              createdAt: "2025-01-01",
            },
          },
          {
            id: "gb_members",
            name: "Group Members",
            type: "member_list",
            updatedAt: "2026-03-31T10:00:00Z",
            path: "/API Workspaces/Backend Team/Group Members",
            content: [
              {
                id: 1,
                name: "Alice Smith",
                role: "Admin",
                email: "alice@example.com",
              },
              {
                id: 2,
                name: "Bob Jones",
                role: "Developer",
                email: "bob@example.com",
              },
            ],
          },
          {
            id: "gb_docs",
            name: "Group Docs",
            type: "doc_folder",
            updatedAt: "2026-03-31T10:00:00Z",
            path: "/API Workspaces/Backend Team/Group Docs",
            children: [
              {
                id: "gb_doc_1",
                name: "Architecture.md",
                type: "doc",
                updatedAt: "2026-03-31T10:00:00Z",
                path: "/API Workspaces/Backend Team/Group Docs/Architecture.md",
                content:
                  "# Architecture\n\nOur system is based on Node.js microservices communicating via gRPC.",
              },
            ],
          },
          {
            id: "gb_projects",
            name: "Projects",
            type: "folder",
            updatedAt: "2026-03-31T10:00:00Z",
            path: "/API Workspaces/Backend Team/Projects",
            children: [
              {
                id: "proj_main",
                name: "Main API",
                type: "project",
                updatedAt: "2026-03-31T10:00:00Z",
                path: "/API Workspaces/Backend Team/Projects/Main API",
                children: [
                  {
                    id: "pm_api",
                    name: "API Management",
                    type: "api_folder",
                    updatedAt: "2026-03-31T10:00:00Z",
                    path: "/API Workspaces/Backend Team/Projects/Main API/API Management",
                    children: [
                      {
                        id: "api_get_users",
                        name: "GET /users",
                        type: "api",
                        updatedAt: "2026-03-31T10:00:00Z",
                        path: "/API Workspaces/Backend Team/Projects/Main API/API Management/GET users",
                        content: {
                          method: "GET",
                          endpoint: "/users",
                          description: "Get a list of all users",
                          params: [
                            { name: "page", type: "number", default: 1 },
                          ],
                        },
                      },
                      {
                        id: "api_post_users",
                        name: "POST /users",
                        type: "api",
                        updatedAt: "2026-03-31T10:00:00Z",
                        path: "/API Workspaces/Backend Team/Projects/Main API/API Management/POST users",
                        content: {
                          method: "POST",
                          endpoint: "/users",
                          description: "Create a new user",
                          body: '{\n  "name": "string",\n  "email": "string"\n}',
                        },
                      },
                    ],
                  },
                  {
                    id: "pm_docs",
                    name: "Documentation",
                    type: "doc_folder",
                    updatedAt: "2026-03-31T10:00:00Z",
                    path: "/API Workspaces/Backend Team/Projects/Main API/Documentation",
                    children: [],
                  },
                  {
                    id: "pm_cicd",
                    name: "CI/CD Pipeline",
                    type: "cicd",
                    updatedAt: "2026-03-31T10:00:00Z",
                    path: "/API Workspaces/Backend Team/Projects/Main API/CI/CD Pipeline",
                    content: {
                      status: "passing",
                      lastRun: "2026-03-31T09:00:00Z",
                      stages: ["Build", "Test", "Deploy"],
                    },
                  },
                  {
                    id: "pm_settings",
                    name: "Project Settings",
                    type: "setting",
                    updatedAt: "2026-03-31T10:00:00Z",
                    path: "/API Workspaces/Backend Team/Projects/Main API/Project Settings",
                    content: {
                      env: "Production",
                      version: "1.2.0",
                      baseUrl: "https://api.example.com",
                    },
                  },
                  {
                    id: "pm_members",
                    name: "Project Members",
                    type: "member_list",
                    updatedAt: "2026-03-31T10:00:00Z",
                    path: "/API Workspaces/Backend Team/Projects/Main API/Project Members",
                    content: [{ id: 1, name: "Alice Smith", role: "Owner" }],
                  },
                ],
              },
            ],
          },
        ],
      },
    ],
  };

  return {
    props: {
      windowData: Object,
    },
    // system: v1 Desktop Workspace 局部状态（来自 ViewXspace）
    inject: ["system"],
    data() {
      return {
        mockApiData: mockApiData,
        // Global Environment State
        environments: ["Development", "Test", "Staging", "Production"],
        activeEnvironment: "Development",
        // State
        selectedFile: null,
        searchQuery: "",
        showPreview: true,
        expandedFolders: new Set([mockApiData.id]),
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
      };
    },
    computed: {
      activeNode() {
        const node = this.windowData || this.mockApiData;
        this.expandAncestors(node.id);
        return node;
      },
      filteredAndSortedFiles() {
        if (!this.isFolderType(this.activeNode.type)) return [];

        let files = this.activeNode.children || [];

        if (this.searchQuery) {
          const lowerQuery = this.searchQuery.toLowerCase();
          files = files.filter((f) =>
            f.name.toLowerCase().includes(lowerQuery),
          );
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
                new Date(a.updatedAt).getTime() -
                new Date(b.updatedAt).getTime();
              break;
            case "type":
              comparison = a.type.localeCompare(b.type);
              break;
          }

          return this.sortDirection === "asc" ? comparison : -comparison;
        });
      },
      canNavigateUp() {
        return this.activeNode.id !== this.mockApiData.id;
      },
    },
    methods: {
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
          "group",
          "project",
          "api_folder",
          "doc_folder",
          "folder",
        ].includes(type);
      },
      findParentNode(root, targetId) {
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
        while (currentId !== this.mockApiData.id) {
          const parent = this.findParentNode(this.mockApiData, currentId);
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
        if (this.activeNode.id === this.mockApiData.id) return;
        const parent = this.findParentNode(
          this.mockApiData,
          this.activeNode.id,
        );
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
      },
      getIcon(type) {
        switch (type) {
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
          default:
            return "file";
        }
      },
      getIconColor(type) {
        switch (type) {
          case "group":
            return "api-manager__icon api-manager__icon--group";
          case "project":
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
          default:
            return "api-manager__icon api-manager__icon--default";
        }
      },
      // Editors Methods
      startEditing() {
        this.editingContent = JSON.parse(
          JSON.stringify(this.activeNode.content || {}),
        );
      },
      saveEditing() {
        this.activeNode.content = this.editingContent;
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
                  { id: 2, name: "Bob Jones", email: "bob@example.com" },
                ],
                meta: { total: 2, page: 1 },
              };
            } else if (
              endpoint.includes("users") &&
              this.activeNode.content?.method === "POST"
            ) {
              mockResponse = {
                status: "success",
                environment: this.activeEnvironment,
                message: "User created successfully",
                data: { id: 3, name: "New User", email: "new@example.com" },
              };
            } else {
              mockResponse = {
                status: "success",
                environment: this.activeEnvironment,
                message: "Request executed successfully",
                timestamp: new Date().toISOString(),
              };
            }

            this.responseData = JSON.stringify(mockResponse, null, 2);
          },
          800 + Math.random() * 500,
        );
      },
    },
  };

}

</script>

<template>
	<div class="api-manager">
		<!-- Top Bar -->
		<div class="api-manager__topbar">
			<!-- Toolbar Row -->
			<div class="api-manager__toolbar">
				<div class="api-manager__nav">
					<button class="api-manager__toolbar-btn api-manager__toolbar-btn--disabled" type="button">
						<xIcon icon="left" :size="20" />
					</button>
					<button class="api-manager__toolbar-btn api-manager__toolbar-btn--disabled" type="button">
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
						<select
							v-model="activeEnvironment"
							class="api-manager__select">
							<option v-for="env in environments" :key="env" :value="env">{{
								env
							}}</option>
						</select>
					</div>

					<div class="api-manager__divider" aria-hidden="true"></div>

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
			<div
				class="api-manager__sidebar">
				<div class="api-manager__sidebar-inner">
					<h2
						class="api-manager__sidebar-title">
						API Explorer
					</h2>

					<!-- Recursive Tree Component inline simulation -->
					<div class="api-manager__tree">
						<!-- Root -->
						<div
							class="api-manager__tree-item"
							:class="{ 'api-manager__tree-item--active': activeNode.id === mockApiData.id }"
							style="padding-left: 8px"
							@click="handleOpenNode(mockApiData)">
							<span
								class="api-manager__tree-toggle"
								@click="e => toggleFolder(mockApiData.id, e)">
								<xIcon 
													type="chevron-down"
													v-if="expandedFolders.has(mockApiData.id)"
													:size="14" />
												<xIcon 
													type="chevron-right" 
													v-else 
													:size="14" />
							</span>
							<span class="api-manager__tree-icon">
								<xIcon 
												:type="getIcon(mockApiData.type)"
												:size="16"
												:class="getIconColor(mockApiData.type)" />
							</span>
							<span class="api-manager__tree-label">{{ mockApiData.name }}</span>
						</div>

						<!-- Level 1 -->
						<div v-if="expandedFolders.has(mockApiData.id)">
							<template v-for="child in mockApiData.children" :key="child.id">
								<div>
									<div
										class="api-manager__tree-item"
										:class="{ 'api-manager__tree-item--active': activeNode.id === child.id }"
										style="padding-left: 20px"
										@click="handleOpenNode(child)">
										<span
											class="api-manager__tree-toggle"
											@click="e => toggleFolder(child.id, e)">
											<template
												v-if="
													isFolderType(child.type) &&
													child.children?.length
												">
												<xIcon 
															type="chevron-down"
															v-if="expandedFolders.has(child.id)"
															:size="14" />
														<xIcon 
															type="chevron-right" 
															v-else 
															:size="14" />
											</template>
										</span>
										<span class="api-manager__tree-icon">
											<xIcon 
														:type="getIcon(child.type)"
														:size="16"
														:class="getIconColor(child.type)" />
										</span>
										<span class="api-manager__tree-label">{{ child.name }}</span>
									</div>

									<!-- Level 2 -->
									<div v-if="expandedFolders.has(child.id) && child.children">
										<template
											v-for="subchild in child.children"
											:key="subchild.id">
											<div>
												<div
													class="api-manager__tree-item"
													:class="{ 'api-manager__tree-item--active': activeNode.id === subchild.id }"
													style="padding-left: 32px"
													@click="handleOpenNode(subchild)">
													<span
														class="api-manager__tree-toggle"
														@click="e => toggleFolder(subchild.id, e)">
														<template
															v-if="
																isFolderType(subchild.type) &&
																subchild.children?.length
															">
															<xIcon 
																	type="chevron-down"
																	v-if="
																		expandedFolders.has(subchild.id)
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
																:type="getIcon(subchild.type)"
																:size="16"
																:class="getIconColor(subchild.type)" />
													</span>
													<span class="api-manager__tree-label">{{ subchild.name }}</span>
												</div>

												<!-- Level 3 -->
												<div
													v-if="
														expandedFolders.has(subchild.id) &&
														subchild.children
													">
													<template
														v-for="subsubchild in subchild.children"
														:key="subsubchild.id">
														<div>
															<div
																class="api-manager__tree-item"
																:class="{ 'api-manager__tree-item--active': activeNode.id === subsubchild.id }"
																style="padding-left: 44px"
																@click="
																	handleOpenNode(subsubchild)
																">
																<span
																	class="api-manager__tree-toggle"
																	@click="
																		e =>
																			toggleFolder(
																				subsubchild.id,
																				e
																			)
																	">
																	<template
																		v-if="
																			isFolderType(
																				subsubchild.type
																			) &&
																			subsubchild.children
																				?.length
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
																				:type="
																					getIcon(
																						subsubchild.type
																					)
																				"
																				:size="16"
																				:class="
																					getIconColor(
																						subsubchild.type
																					)
																				" />
																</span>
																<span class="api-manager__tree-label">{{ subsubchild.name }}</span>
															</div>

															<!-- Level 4 -->
															<div
																v-if="
																	expandedFolders.has(
																		subsubchild.id
																	) && subsubchild.children
																">
																<template
																	v-for="leaf in subsubchild.children"
																	:key="leaf.id">
																	<div>
																		<div
																			class="api-manager__tree-item"
																			:class="{ 'api-manager__tree-item--active': activeNode.id === leaf.id }"
																			style="
																				padding-left: 56px;
																			"
																			@click="
																				handleOpenNode(leaf)
																			">
																			<span
																				class="api-manager__tree-toggle api-manager__tree-toggle--spacer"
																				aria-hidden="true"></span>
																			<span class="api-manager__tree-icon">
																				<xIcon 
																					:type="
																						getIcon(
																							leaf.type
																						)
																					"
																					:size="16"
																					:class="
																						getIconColor(
																							leaf.type
																						)
																					" />
																			</span>
																			<span class="api-manager__tree-label">{{ leaf.name }}</span>
																		</div>
																	</div>
																</template>
															</div>
														</div>
													</template>
												</div>
											</div>
										</template>
									</div>
								</div>
							</template>
						</div>
					</div>
				</div>
			</div>

			<!-- Main Area -->
			<div class="api-manager__main">
				<!-- Breadcrumb -->
				<div class="api-manager__breadcrumb">
					<template
						v-for="(part, index) in activeNode.path.split('/').filter(Boolean)"
						:key="index">
						<span
							class="api-manager__breadcrumb-part"
							:class="{ 'api-manager__breadcrumb-part--current': index === activeNode.path.split('/').filter(Boolean).length - 1 }">
							{{ part }}
						</span>
						<span
							v-if="index < activeNode.path.split('/').filter(Boolean).length - 1"
							class="api-manager__breadcrumb-sep"
							>/</span
						>
					</template>
				</div>

				<!-- Folder View -->
				<template v-if="isFolderType(activeNode.type)">
					<!-- Table Header -->
					<div class="api-manager__table-header">
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

					<!-- File List -->
					<div class="api-manager__file-list" @click="selectedFile = null">
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
								:class="{ 'api-manager__file-row--selected': selectedFile?.id === file.id }">
								<div class="api-manager__file-cell api-manager__file-cell--name">
									<xIcon 
												:type="getIcon(file.type)"
												:size="20"
												:class="getIconColor(file.type)" />
									<span class="api-manager__file-name">{{
										file.name
									}}</span>
								</div>
								<div class="api-manager__file-cell api-manager__file-cell--date">
									{{ formatDate(file.updatedAt) }}
								</div>
								<div class="api-manager__file-cell api-manager__file-cell--type">
									{{ file.type.replace("_", " ") }}
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
											>Cancel</button
										>
										<button
											@click="saveEditing"
											class="api-manager__action-btn api-manager__action-btn--primary api-manager__action-btn--with-icon">
											<xIcon icon="save" :size="16" /> Save
										</button>
									</template>
									<template v-else>
										<button
											@click="startEditing"
											class="api-manager__action-btn api-manager__action-btn--secondary api-manager__action-btn--with-icon">
											<xIcon icon="edit" :size="16" /> Edit
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
												<xIcon icon="plus" :size="16" /> Add Member
											</button>
										</div>
										<div
											class="api-manager__table-shell">
											<table class="api-manager__table">
												<thead
													class="api-manager__table-head">
													<tr>
														<th class="api-manager__th">Name</th>
														<th class="api-manager__th">Role</th>
														<th class="api-manager__th">Email</th>
														<th
															v-if="editingContent"
															class="api-manager__th api-manager__th--right"
															>Actions</th
														>
													</tr>
												</thead>
												<tbody>
													<tr
														v-for="(member, idx) in editingContent ||
														activeNode.content"
														:key="idx"
														class="api-manager__tr">
														<td
															class="api-manager__td api-manager__td--strong">
															<input
																v-if="editingContent"
																v-model="member.name"
																class="api-manager__field-input api-manager__field-input--inline" />
															<span v-else>{{ member.name }}</span>
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
																class="api-manager__icon-action"
																><xIcon icon="delete" :size="16" /></button>
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
											<label
												class="api-manager__setting-label"
												>{{
													String(key)
														.replace(/([A-Z])/g, " $1")
														.trim()
												}}</label
											>
											<div class="api-manager__setting-control">
												<input
													v-if="editingContent"
													v-model="editingContent[key]"
													class="api-manager__field-input api-manager__field-input--block" />
												<div
													v-else
													class="api-manager__field-value"
													>{{ val }}</div
												>
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
												<option>GET</option
												><option>POST</option
												><option>PUT</option
												><option>DELETE</option>
											</select>
											<span
												v-else
												:class="getMethodColor(activeNode.content?.method)">
												{{ activeNode.content?.method }}
											</span>

											<input
												v-if="editingContent"
												v-model="editingContent.endpoint"
												class="api-manager__field-input api-manager__field-input--code api-manager__endpoint-input" />
											<span
												v-else
												class="api-manager__endpoint-text"
												>{{ activeNode.content?.endpoint }}</span
											>

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
											<label
												class="api-manager__field-label"
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
												editingContent?.params || activeNode.content?.params
											">
											<label
												class="api-manager__field-label api-manager__field-label--spaced"
												>Parameters</label
											>
											<div
												class="api-manager__code-block">
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
											v-if="editingContent?.body || activeNode.content?.body">
											<label
												class="api-manager__field-label api-manager__field-label--spaced"
												>Request Body</label
											>
											<div
												class="api-manager__code-block">
												<pre>{{
													editingContent?.body || activeNode.content?.body
												}}</pre>
											</div>
										</div>

										<!-- Response Section -->
										<div
											v-if="!editingContent"
											class="api-manager__response">
											<div class="api-manager__response-header">
												<h3
													class="api-manager__response-title">
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
												<pre>{{
													responseData
												}}</pre>
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

			<!-- Preview Pane (Only for Folders) -->
			<div
				v-if="showPreview && isFolderType(activeNode.type)"
				class="api-manager__preview">
				<div
					v-if="!selectedFile"
					class="api-manager__preview-empty">
					<xIcon icon="info-filled" :size="48" class="api-manager__preview-empty-icon" />
					<p>Select a resource to preview</p>
				</div>
				<div v-else class="api-manager__preview-inner">
					<!-- Preview Content -->
					<div
						class="api-manager__preview-hero">
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
							<h4
								class="api-manager__preview-section-title">
								Information
							</h4>
							<dl class="api-manager__preview-dl">
								<div class="api-manager__preview-dl-row">
									<dt class="api-manager__preview-dt">Modified</dt>
									<dd class="api-manager__preview-dd">
										{{ formatDate(selectedFile.updatedAt) }}
									</dd>
								</div>
								<div class="api-manager__preview-dl-row">
									<dt class="api-manager__preview-dt">Path</dt>
									<dd class="api-manager__preview-dd api-manager__preview-dd--break">
										{{ selectedFile.path }}
									</dd>
								</div>
							</dl>
						</div>

						<div
							v-if="selectedFile.content"
							class="api-manager__preview-section">
							<h4
								class="api-manager__preview-section-title">
								Quick Preview
							</h4>
							<div
								class="api-manager__code-block api-manager__code-block--preview">
								<pre
									class="api-manager__preview-code"
									>{{
										typeof selectedFile.content === "string"
											? selectedFile.content
											: JSON.stringify(selectedFile.content, null, 2)
									}}</pre
								>
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
</template>

<style lang="less">
.api-manager {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: var(--color-surface);
  color: var(--color-on-surface);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", "PingFang SC",
    "Hiragino Sans GB", "Microsoft Yahei", sans-serif;

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
    font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
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
    transition: background-color 0.2s ease, border-color 0.2s ease, color 0.2s ease;

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

  &__search-input,
  &__field-input,
  &__select-shell {
    border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
    border-radius: 10px;
    background: var(--color-surface-container-lowest);
    color: var(--color-on-surface);
    transition: border-color 0.2s ease, box-shadow 0.2s ease, background-color 0.2s ease;
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
    transition: background-color 0.2s ease, border-color 0.2s ease, color 0.2s ease,
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
  &__icon--group,
  &__icon--member-list {
    color: var(--el-color-primary);
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
}
</style>
