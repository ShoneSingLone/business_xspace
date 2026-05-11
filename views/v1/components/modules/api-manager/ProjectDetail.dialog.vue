<script lang="ts">
export default async function ({ PRIVATE_GLOBAL, closeModal, project }) {
	const { useDialogProps } = await _.$importVue("/common/utils/hooks.vue");
	const api = await _.$importVue("@/utils/api.vue");

	return {
		inject: ["APP"],
		props: useDialogProps(),
		components: {
			ProjectInterfaces: () => _.$importVue("@/views/v1/components/modules/api-manager/project-detail/ProjectInterfaces.vue"),
			ProjectMembers: () => _.$importVue("@/views/v1/components/modules/api-manager/project-detail/ProjectMembers.vue"),
			ProjectDocs: () => _.$importVue("@/views/v1/components/modules/api-manager/project-detail/ProjectDocs.vue"),
			ProjectSettings: () => _.$importVue("@/views/v1/components/modules/api-manager/project-detail/ProjectSettings.vue"),
			ProjectLogs: () => _.$importVue("@/views/v1/components/modules/api-manager/project-detail/ProjectLogs.vue")
		},
		data() {
			return {
				project: project,
				activeTab: "interfaces",
				searchQuery: "",
				isLoading: false,
				projectData: null,
				sidebarItems: [
					{ id: "interfaces", name: "接口", icon: "code" },
					{ id: "members", name: "项目成员", icon: "_member_list" },
					{ id: "docs", name: "项目文档", icon: "file-text" },
					{ id: "settings", name: "项目设置", icon: "_configs" },
					{ id: "logs", name: "项目日志", icon: "_article" }
				]
			};
		},
		mounted() {
			this.loadProjectData();
		},
		setup() {
			const size = ref({ width: 0, height: 0 });
			this.setSize = ({ width, height }) => {
				this.size = { width, height };
			};
			return { size };
		},
		methods: {
			handleTabChange(tabId) {
				this.activeTab = tabId;
			},
			async loadProjectData() {
				if (!this.project?.id) return;
				
				this.isLoading = true;
				try {
					const response = await api.projectGet(this.project.id);
					if (response.errcode === 0) {
						this.projectData = response.data;
					}
				} catch (error) {
					console.error("Failed to load project data:", error);
				} finally {
					this.isLoading = false;
				}
			}
		}
	};
}
</script>

<template>
	<div class="project-detail">
		<div class="project-detail__sidebar">
			<div class="project-detail__sidebar-header">
				<div class="project-detail__project-icon">
					<xIcon icon="folder" :size="24" />
				</div>
				<h3 class="project-detail__project-name">{{ project?.name || '项目详情' }}</h3>
				<p class="project-detail__project-desc" v-if="projectData?.desc">{{ projectData.desc }}</p>
			</div>
			<div class="project-detail__sidebar-nav">
				<div 
					v-for="item in sidebarItems" 
					:key="item.id"
					:class="{
						'project-detail__nav-item': true,
						'project-detail__nav-item--active': activeTab === item.id
					}"
					@click="handleTabChange(item.id)">
					<xIcon :icon="item.icon" :size="16" />
					<span>{{ item.name }}</span>
				</div>
			</div>
		</div>
		
		<div class="project-detail__content">
			<div class="project-detail__content-header">
				<xBlock class="project-detail__breadcrumb-wrapper" :bodyClass="{ 'flex middle': true }">
					<span class="project-detail__breadcrumb-item">{{ project?.name }}</span>
					<span class="project-detail__breadcrumb-sep">/</span>
					<span class="project-detail__breadcrumb-item project-detail__breadcrumb-item--current">
						{{ sidebarItems.find(item => item.id === activeTab)?.name }}
					</span>
					<xGap f />
					<xIcon icon="close" @click="closeModal" class="pointer" />
				</xBlock>
			</div>
			
			<div class="project-detail__content-body">
				<xPageContent>
					<xAutoResizer :onResize="setSize">
						<ProjectInterfaces 
							v-if="activeTab === 'interfaces'" 
							:project="project" 
							:project-data="projectData" />
						<ProjectMembers 
							v-else-if="activeTab === 'members'" 
							:project="project" 
							:project-data="projectData" />
						<ProjectDocs 
							v-else-if="activeTab === 'docs'" 
							:project="project" 
							:project-data="projectData" />
						<ProjectSettings 
							v-else-if="activeTab === 'settings'" 
							:project="project" 
							:project-data="projectData"
							@refresh="loadProjectData" />
						<ProjectLogs 
							v-else-if="activeTab === 'logs'" 
							:project="project" 
							:project-data="projectData" />
					</xAutoResizer>
				</xPageContent>
			</div>
		</div>
	</div>
</template>

<style lang="less">
.project-detail {
	height: 100%;
	display: flex;
	overflow: hidden;
	background: var(--color-surface);
	color: var(--color-on-surface);
	font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", "PingFang SC", "Hiragino Sans GB", "Microsoft Yahei", sans-serif;

	&__sidebar {
		width: 240px;
		flex-shrink: 0;
		border-right: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container);
		display: flex;
		flex-direction: column;
	}

	&__sidebar-header {
		padding: 20px 16px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	}

	&__project-icon {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 48px;
		height: 48px;
		border-radius: 12px;
		background: var(--color-primary-container);
		color: var(--color-on-primary-container);
		margin-bottom: 12px;
	}

	&__project-name {
		margin: 0 0 4px;
		font-size: 16px;
		font-weight: 700;
		color: var(--color-on-surface);
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	&__project-desc {
		margin: 0;
		font-size: 12px;
		color: var(--color-on-surface-variant);
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	&__sidebar-nav {
		flex: 1;
		overflow-y: auto;
		padding: 8px;
	}

	&__nav-item {
		display: flex;
		align-items: center;
		gap: 12px;
		padding: 10px 12px;
		border-radius: 10px;
		cursor: pointer;
		font-size: 14px;
		color: var(--color-on-surface-variant);
		transition: all 0.15s ease;
		margin-bottom: 4px;

		&:hover {
			background: var(--color-surface-variant);
			color: var(--color-on-surface);
		}

		&--active {
			background: var(--color-primary-container);
			color: var(--color-on-primary-container);
			font-weight: 600;

			&:hover {
				background: var(--color-primary-container);
			}
		}
	}

	&__content {
		flex: 1;
		overflow: hidden;
		display: flex;
		flex-direction: column;
		background: var(--color-surface-container-lowest);
	}

	&__content-header {
		padding: 12px 20px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container);
		flex-shrink: 0;
	}

	&__breadcrumb {
		display: flex;
		align-items: center;
		gap: 8px;
		font-size: 14px;
	}

	&__breadcrumb-item {
		color: var(--color-on-surface-variant);

		&--current {
			color: var(--color-on-surface);
			font-weight: 600;
		}
	}

	&__breadcrumb-sep {
		color: var(--color-outline-variant);
	}

	&__content-body {
		flex: 1;
		overflow-y: auto;
		padding: 20px;
	}
}
</style>
