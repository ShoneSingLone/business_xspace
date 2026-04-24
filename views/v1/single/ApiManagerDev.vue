<script lang="ts">
export default async function () {
	const mockApiData = {
		id: "root",
		name: "API Workspaces",
		type: "folder",
		updatedAt: "2026-03-31T10:00:00Z",
		path: "/API Workspaces",
		children: [
			{
				id: "personal_space",
				name: "个人空间",
				type: "personal",
				role: "owner",
				updatedAt: "2026-04-01T12:00:00Z",
				path: "/API Workspaces/个人空间",
				visibility: "private",
				children: [
					{
						id: "ps_my_projects",
						name: "我的项目",
						type: "folder",
						updatedAt: "2026-04-01T12:00:00Z",
						path: "/API Workspaces/个人空间/我的项目",
						children: [
							{
								id: "ps_proj_private",
								name: "个人项目",
								type: "project",
								role: "owner",
								updatedAt: "2026-04-01T11:00:00Z",
								path: "/API Workspaces/个人空间/我的项目/个人项目",
								visibility: "private",
								description: "个人开发项目",
								tags: ["个人", "开发"],
								children: [
									{
										id: "ps_p_api",
										name: "API",
										type: "api_folder",
										updatedAt: "2026-04-01T10:00:00Z",
										path: "/API Workspaces/个人空间/我的项目/个人项目/API",
										children: []
									},
									{
										id: "ps_p_docs",
										name: "文档",
										type: "doc_folder",
										updatedAt: "2026-04-01T10:00:00Z",
										path: "/API Workspaces/个人空间/我的项目/个人项目/文档",
										children: []
									}
								]
							}
						]
					}
				]
			},
			{
				id: "group_backend",
				name: "Backend Team",
				type: "group",
				role: "admin",
				updatedAt: "2026-03-31T10:00:00Z",
				path: "/API Workspaces/Backend Team",
				visibility: "internal",
				description: "Core backend services team handling microservices.",
				createdBy: "Admin",
				createdAt: "2025-01-01",
				stats: {
					projectCount: 3,
					docCount: 15,
					memberCount: 8,
					activityDays: 7
				},
				children: [
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
								role: "owner",
								updatedAt: "2026-03-31T10:00:00Z",
								path: "/API Workspaces/Backend Team/Projects/Main API",
								visibility: "internal",
								description: "核心 API 服务",
								tags: ["核心", "API"],
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
														{ name: "page", type: "number", default: 1 }
													]
												}
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
													body: '{\n  "name": "string",\n  "email": "string"\n}'
												}
											}
										]
									},
									{
										id: "pm_docs",
										name: "Documentation",
										type: "doc_folder",
										updatedAt: "2026-03-31T10:00:00Z",
										path: "/API Workspaces/Backend Team/Projects/Main API/Documentation",
										children: []
									}
								]
							}
						]
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
								role: "owner",
								email: "alice@example.com"
							},
							{ id: 2, name: "Bob Jones", role: "admin", email: "bob@example.com" },
							{
								id: 3,
								name: "Charlie Brown",
								role: "developer",
								email: "charlie@example.com"
							}
						]
					}
				]
			}
		]
	};

	return {
		provide: {
			system: {
				openApp: (appId, newWindow, node) => {
					console.log("openApp called:", appId, newWindow, node);
				}
			}
		},
		data() {
			return {
				mockData: mockApiData
			};
		},
		template: `
      <div class="api-manager-dev-container">
        <ApiManager :windowData="null" style="height: 100%;" />
      </div>
    `,
		components: {
			ApiManager: () => _.$importVue("@/views/v1/components/modules/ApiManager.vue")
		}
	};
}
</script>

<style lang="less">
html,
body,
#app {
	height: 100%;
	margin: 0;
	padding: 0;
}

.api-manager-dev-container {
	height: 100vh;
	min-height: 100vh;
	background: #1e1e1e;
	display: flex;
	flex-direction: column;

	--color-surface: #1e1e1e;
	--color-surface-container: #252526;
	--color-surface-container-lowest: #1e1e1e;
	--color-surface-container-low: #2d2d30;
	--color-surface-variant: #3c3c3c;
	--color-on-surface: #cccccc;
	--color-on-surface-variant: #9d9d9d;
	--color-outline-variant: #4a4a4a;
	--color-primary: #007acc;
	--color-primary-container: #007acc;
	--color-error: #f14c4c;
	--color-success: #6a9955;
	--color-warning: #ce9178;
}
</style>
