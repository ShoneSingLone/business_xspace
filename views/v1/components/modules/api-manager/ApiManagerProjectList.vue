<script lang="ts">
export default async function () {
	return {
		props: {
			projects: {
				type: Array,
				default: () => []
			},
			selectedProjectId: {
				type: String,
				default: null
			},
			sortField: {
				type: String,
				default: "name"
			},
			sortDirection: {
				type: String,
				default: "asc"
			},
			viewMode: {
				type: String,
				default: "list"
			},
			searchQuery: {
				type: String,
				default: ""
			}
		},
		
		computed: {
			filteredAndSortedProjects() {
				let result = [...this.projects];
				
				if (this.searchQuery) {
					const lowerQuery = this.searchQuery.toLowerCase();
					result = result.filter(p => 
						p.name?.toLowerCase().includes(lowerQuery) ||
						p.description?.toLowerCase().includes(lowerQuery)
					);
				}
				
				result.sort((a, b) => {
					let comparison = 0;
					switch (this.sortField) {
						case "name":
							comparison = a.name.localeCompare(b.name);
							break;
						case "updatedAt":
							comparison = new Date(a.updatedAt).getTime() - new Date(b.updatedAt).getTime();
							break;
						case "visibility":
							comparison = a.visibility.localeCompare(b.visibility);
							break;
					}
					return this.sortDirection === "asc" ? comparison : -comparison;
				});
				
				return result;
			},
			hasProjects() {
				return this.filteredAndSortedProjects.length > 0;
			}
		},
		methods: {
			selectProject(project) {
				this.$emit("select-project", project);
			},
			openProject(project) {
				this.$emit("open-project", project);
			},
			handleSort(field) {
				this.$emit("sort", field);
			},
			toggleStar(project) {
				this.$emit("toggle-star", project);
			},
			handleContextMenu(project, event) {
				this.$emit("context-menu", project, event);
			},
			formatDate(dateString) {
				try {
					const date = new Date(dateString);
					return date.toLocaleDateString("zh-CN", {
						year: "numeric",
						month: "short",
						day: "numeric"
					});
				} catch (e) {
					return dateString;
				}
			},
			getProjectCardClass(project) {
				return {
					'api-manager__project-card': true,
					'api-manager__project-card--selected': this.selectedProjectId === project.id
				};
			},
			getProjectRowClass(project) {
				return {
					'api-manager__project-row': true,
					'api-manager__project-row--selected': this.selectedProjectId === project.id
				};
			},
			getVisibilityBadgeClass(visibility) {
				return visibility === "private" 
					? "api-manager__visibility-badge api-manager__visibility-badge--private"
					: "api-manager__visibility-badge api-manager__visibility-badge--public";
			}
		}
	};
}
</script>

<template>
	<div class="api-manager__project-list-container">

		<div class="api-manager__project-table-header" v-if="viewMode === 'list'">
			<div 
				class="api-manager__project-table-header-cell api-manager__project-table-header-cell--name" 
				@click="handleSort('name')">
				<xIcon icon="_folder" :size="14" />
				<span>项目名称</span>
				<xIcon icon="arrow-up" v-if="sortField === 'name' && sortDirection === 'asc'" :size="12" class="api-manager__sort-icon" />
				<xIcon icon="arrow-down" v-if="sortField === 'name' && sortDirection === 'desc'" :size="12" class="api-manager__sort-icon" />
			</div>
			<div 
				class="api-manager__project-table-header-cell api-manager__project-table-header-cell--visibility" 
				@click="handleSort('visibility')">
				<xIcon icon="_eye" :size="14" />
				<span>可见性</span>
				<xIcon icon="arrow-up" v-if="sortField === 'visibility' && sortDirection === 'asc'" :size="12" class="api-manager__sort-icon" />
				<xIcon icon="arrow-down" v-if="sortField === 'visibility' && sortDirection === 'desc'" :size="12" class="api-manager__sort-icon" />
			</div>
			<div 
				class="api-manager__project-table-header-cell api-manager__project-table-header-cell--updated" 
				@click="handleSort('updatedAt')">
				<xIcon icon="_clock" :size="14" />
				<span>更新时间</span>
				<xIcon icon="arrow-up" v-if="sortField === 'updatedAt' && sortDirection === 'asc'" :size="12" class="api-manager__sort-icon" />
				<xIcon icon="arrow-down" v-if="sortField === 'updatedAt' && sortDirection === 'desc'" :size="12" class="api-manager__sort-icon" />
			</div>
			<div class="api-manager__project-table-header-cell api-manager__project-table-header-cell--actions">
				<span>操作</span>
			</div>
		</div>

		<div class="api-manager__project-list" v-if="viewMode === 'list'">
			<div v-if="!hasProjects" class="api-manager__empty-state">
				<xIcon icon="_folder-open" :size="48" class="api-manager__empty-icon" />
				<p>暂无项目</p>
				<p class="api-manager__empty-hint">点击右上角按钮创建新项目</p>
			</div>
			<div v-else class="api-manager__project-list-inner">
				<div 
					v-for="project in filteredAndSortedProjects" 
					:key="project.id"
					@click.stop="selectProject(project)" 
					@dblclick.stop="openProject(project)"
					@contextmenu="handleContextMenu(project, $event)"
					:class="getProjectRowClass(project)">
					<div class="api-manager__project-row-cell api-manager__project-row-cell--name">
						<xIcon icon="folder" :size="18" class="api-manager__project-icon" />
						<span class="api-manager__project-name">{{ project.name }}</span>
						<span v-if="project.role" class="api-manager__project-role">{{ project.role }}</span>
					</div>
					<div class="api-manager__project-row-cell api-manager__project-row-cell--visibility">
						<span :class="getVisibilityBadgeClass(project.visibility)">
							{{ project.visibility === "private" ? "私有" : "公开" }}
						</span>
					</div>
					<div class="api-manager__project-row-cell api-manager__project-row-cell--updated">
						{{ formatDate(project.updatedAt) }}
					</div>
					<div class="api-manager__project-row-cell api-manager__project-row-cell--actions">
						<button 
							class="api-manager__project-action-btn" 
							@click.stop="toggleStar(project)"
							:title="project.followed ? '取消星标' : '添加星标'">
							<xIcon 
								:icon="project.followed ? 'star-filled' : 'star'" 
								:size="14" 
								:class="{ 'api-manager__star-icon--active': project.followed }" />
						</button>
						<button 
							class="api-manager__project-action-btn" 
							@click.stop="handleContextMenu(project, $event)"
							title="更多操作">
							<xIcon icon="_more-horizontal" :size="14" />
						</button>
					</div>
				</div>
			</div>
		</div>

		<div class="api-manager__project-grid" v-else>
			<div v-if="!hasProjects" class="api-manager__empty-state">
				<xIcon icon="_folder-open" :size="48" class="api-manager__empty-icon" />
				<p>暂无项目</p>
				<p class="api-manager__empty-hint">点击右上角按钮创建新项目</p>
			</div>
			<div v-else class="api-manager__project-grid-inner">
				<div 
					v-for="project in filteredAndSortedProjects" 
					:key="project.id"
					@click.stop="selectProject(project)" 
					@dblclick.stop="openProject(project)"
					@contextmenu="handleContextMenu(project, $event)"
					:class="getProjectCardClass(project)">
					<div class="api-manager__project-card-icon">
						<xIcon icon="folder" :size="32" class="api-manager__project-card-icon-inner" />
					</div>
					<div class="api-manager__project-card-content">
						<h4 class="api-manager__project-card-name">{{ project.name }}</h4>
						<p class="api-manager__project-card-meta">
							<span :class="getVisibilityBadgeClass(project.visibility)">
								{{ project.visibility === "private" ? "私有" : "公开" }}
							</span>
							<span v-if="project.role" class="api-manager__project-card-role">{{ project.role }}</span>
						</p>
						<p class="api-manager__project-card-date">{{ formatDate(project.updatedAt) }}</p>
					</div>
					<div class="api-manager__project-card-actions">
						<button 
							class="api-manager__project-card-action-btn" 
							@click.stop="toggleStar(project)"
							:title="project.followed ? '取消星标' : '添加星标'">
							<xIcon 
								:icon="project.followed ? 'star-filled' : 'star'" 
								:size="14" 
								:class="{ 'api-manager__star-icon--active': project.followed }" />
						</button>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style lang="less">
.api-manager__project-list-container {
	height: 100%;
	display: flex;
	flex-direction: column;
}

.api-manager__project-table-header {
	display: grid;
	grid-template-columns: 3fr 1fr 1fr 1fr;
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

.api-manager__project-table-header-cell {
	display: flex;
	align-items: center;
	gap: 8px;
	cursor: pointer;
	transition: color 0.15s ease;

	&:hover {
		color: var(--color-on-surface);
	}

	&--name {
		grid-column: 1;
	}

	&--visibility {
		grid-column: 2;
	}

	&--updated {
		grid-column: 3;
	}

	&--actions {
		grid-column: 4;
	}
}

.api-manager__sort-icon {
	margin-left: 4px;
}

.api-manager__project-list {
	flex: 1;
	overflow-y: auto;
}

.api-manager__project-list-inner {
	padding: 4px 0;
	width: 100%;
}

.api-manager__project-row {
	display: grid;
	grid-template-columns: 3fr 1fr 1fr 1fr;
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

	&--selected {
		background: color-mix(in srgb, var(--color-primary-container) 50%, transparent);
		border-color: color-mix(in srgb, var(--color-primary) 12%, transparent);
	}
}

.api-manager__project-row-cell {
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;

	&--name {
		display: flex;
		align-items: center;
		gap: 12px;
		color: var(--color-on-surface);
	}

	&--visibility {
		display: flex;
		align-items: center;
	}

	&--updated {
		color: var(--color-on-surface-variant);
	}

	&--actions {
		display: flex;
		align-items: center;
		gap: 8px;
		justify-content: flex-end;
	}
}

.api-manager__project-icon {
	color: var(--color-on-surface-variant);
}

.api-manager__project-name {
	font-weight: 600;
}

.api-manager__project-role {
	font-size: 11px;
	font-weight: 600;
	padding: 2px 6px;
	border-radius: 4px;
	background: color-mix(in srgb, var(--color-secondary) 15%, transparent);
	color: var(--color-secondary);
	margin-left: 8px;
}

.api-manager__visibility-badge {
	font-size: 11px;
	font-weight: 700;
	padding: 2px 8px;
	border-radius: 4px;

	&--private {
		background: color-mix(in srgb, var(--color-error) 15%, transparent);
		color: var(--color-error);
	}

	&--public {
		background: color-mix(in srgb, var(--color-success) 15%, transparent);
		color: var(--color-success);
	}
}

.api-manager__project-action-btn {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 24px;
	height: 24px;
	border: 0;
	background: transparent;
	border-radius: 6px;
	cursor: pointer;
	color: var(--color-on-surface-variant);
	transition: all 0.15s ease;

	&:hover {
		background: var(--color-surface-variant);
		color: var(--color-on-surface);
	}
}

.api-manager__star-icon--active {
	color: var(--color-warning);
}

.api-manager__empty-state {
	height: 100%;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	gap: 8px;
	color: var(--color-on-surface-variant);

	p {
		margin: 0;
	}
}

.api-manager__empty-icon {
	margin-bottom: 8px;
	color: var(--color-outline-variant);
}

.api-manager__empty-hint {
	font-size: 12px;
	color: var(--color-on-surface-variant);
	opacity: 0.7;
}

.api-manager__project-grid {
	flex: 1;
	overflow-y: auto;
	padding: 16px;
	display: flex;
	justify-content: center;
}

.api-manager__project-grid-inner {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
	gap: 12px;
}

.api-manager__project-card {
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

	&--selected {
		background: color-mix(in srgb, var(--color-primary-container) 30%, transparent);
		border-color: color-mix(in srgb, var(--color-primary) 20%, transparent);
	}
}

.api-manager__project-card-icon {
	display: flex;
	justify-content: center;
}

.api-manager__project-card-icon-inner {
	color: var(--color-on-surface-variant);
}

.api-manager__project-card-content {
	display: flex;
	flex-direction: column;
	gap: 4px;
}

.api-manager__project-card-name {
	margin: 0;
	font-size: 14px;
	font-weight: 600;
	color: var(--color-on-surface);
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}

.api-manager__project-card-meta {
	display: flex;
	align-items: center;
	gap: 6px;
	margin: 0;
}

.api-manager__project-card-role {
	font-size: 11px;
	font-weight: 600;
	padding: 2px 6px;
	border-radius: 4px;
	background: color-mix(in srgb, var(--color-secondary) 15%, transparent);
	color: var(--color-secondary);
}

.api-manager__project-card-date {
	margin: 0;
	font-size: 12px;
	color: var(--color-on-surface-variant);
}

.api-manager__project-card-actions {
	display: flex;
	justify-content: flex-end;
}

.api-manager__project-card-action-btn {
	display: flex;
	align-items: center;
	justify-content: center;
	width: 24px;
	height: 24px;
	border: 0;
	background: transparent;
	border-radius: 6px;
	cursor: pointer;
	color: var(--color-on-surface-variant);
	transition: all 0.15s ease;

	&:hover {
		background: var(--color-surface-variant);
		color: var(--color-on-surface);
	}
}
</style>