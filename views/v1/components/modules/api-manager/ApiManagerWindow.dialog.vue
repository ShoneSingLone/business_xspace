<script lang="ts">
export default async function ({ node, closeModal }) {
	const { useDialogProps } = await _.$importVue("/common/utils/hooks.vue");
	const utils = await _.$importVue("@/utils/api-manager.vue");
	const api = await _.$importVue("@/utils/api.vue");

	return {
		inject: ["APP"],
		props: useDialogProps(),
		data() {
			const vm = this;
			return {
				node: node,
				loading: false,
				error: null,
				children: [],
				selectedChild: null,
				viewMode: "list",
				sortField: "name",
				sortDirection: "asc",
				searchQuery: "",
				utils: utils
			};
		},
		computed: {
			isFolder() {
				return utils.isFolderType(this.node?.type);
			},
			isProjectFolder() {
				return this.isFolder && (
					this.node.id === "ps_my_projects" || 
					this.node.id === "ps_followed" || 
					this.node.id?.endsWith("_projects")
				);
			},
			isApi() {
				return this.node?.type === "api";
			},
			filteredChildren() {
				let result = [...this.children];
				if (this.searchQuery) {
					const lowerQuery = this.searchQuery.toLowerCase();
					result = result.filter(c => 
						c.name?.toLowerCase().includes(lowerQuery)
					);
				}
				return result.sort((a, b) => {
					let comparison = 0;
					if (utils.isFolderType(a.type) !== utils.isFolderType(b.type)) {
						return utils.isFolderType(a.type) ? -1 : 1;
					}
					switch (this.sortField) {
						case "name": comparison = a.name.localeCompare(b.name); break;
						case "updatedAt": comparison = new Date(a.updatedAt).getTime() - new Date(b.updatedAt).getTime(); break;
					}
					return this.sortDirection === "asc" ? comparison : -comparison;
				});
			}
		},
		async mounted() {
			await this.loadChildren();
		},
		methods: {
			async loadChildren() {
				if (!this.isFolder) return;
				
				this.loading = true;
				try {
					if (this.isProjectFolder) {
						const isPersonal = this.node.id === "ps_my_projects" || this.node.id === "ps_followed";
						const followedOnly = this.node.id === "ps_followed";
						const groupId = this.node.id?.replace("_projects", "").replace("group_", "");
						
						let result;
						if (isPersonal) {
							result = await api.project_page({ page: 1, size: 1000, isPersonal: true });
							let projects = result.data?.list || [];
							if (followedOnly) {
								projects = projects.filter(p => p.follow);
							} else {
								projects = projects.filter(p => !p.follow);
							}
							this.children = projects.map(p => ({
								id: p._id || p.id,
								name: p.name,
								type: "project",
								role: p.role,
								visibility: p.visibility,
								followed: p.follow,
								updatedAt: p.updatedAt,
								path: p.path,
								content: p
							}));
						} else if (groupId) {
							result = await api.xspace.getProjectByGroupId({ groupId });
							this.children = result.data?.map(p => ({
								id: p._id || p.id,
								name: p.name,
								type: "project",
								role: p.role,
								visibility: p.visibility,
								followed: p.follow,
								updatedAt: p.updatedAt,
								path: p.path,
								content: p
							})) || [];
						}
					} else {
						this.children = this.node.children || [];
					}
				} catch (err) {
					this.error = err.message;
					console.error('Failed to load children:', err);
				} finally {
					this.loading = false;
				}
			},
			selectChild(child) {
				this.selectedChild = child;
			},
			openChild(child) {
				if (utils.isFolderType(child.type)) {
					this.node = child;
					this.selectedChild = null;
					this.loadChildren();
				}
			},
			handleSort(field) {
				this.sortDirection = this.sortField === field ? (this.sortDirection === "asc" ? "desc" : "asc") : "asc";
				this.sortField = field;
			},
			toggleViewMode() {
				this.viewMode = this.viewMode === "list" ? "grid" : "list";
			},
			getVisibilityBadgeClass(visibility) {
				return visibility === "private" 
					? "api-manager__visibility-badge api-manager__visibility-badge--private"
					: "api-manager__visibility-badge api-manager__visibility-badge--public";
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
			close() {
				closeModal();
			},
			goBack() {
				if (this.node.parentId) {
					this.node = { id: this.node.parentId };
					this.loadChildren();
				}
			}
		}
	};
}
</script>

<template>
	<xDialog :style="{ '--xDialog-wrapper-width': '800px', '--xDialog-wrapper-height': '600px' }">
		<div class="api-manager-window">
			<div class="api-manager-window__header">
				<div class="api-manager-window__breadcrumb">
					<button v-if="node.parentId" @click="goBack" class="api-manager-window__back-btn">
						<xIcon icon="arrow-left" :size="14" />
					</button>
					<span class="api-manager-window__title">{{ node?.name }}</span>
				</div>
				<div class="api-manager-window__actions">
					<button 
						class="api-manager-window__view-btn"
						:class="{ 'api-manager-window__view-btn--active': viewMode === 'list' }"
						@click="toggleViewMode" title="列表视图">
						<xIcon icon="list" :size="16" />
					</button>
					<button 
						class="api-manager-window__view-btn"
						:class="{ 'api-manager-window__view-btn--active': viewMode === 'grid' }"
						@click="toggleViewMode" title="网格视图">
						<xIcon icon="grid" :size="16" />
					</button>
				</div>
			</div>

			<div class="api-manager-window__search">
				<input 
					type="text" 
					class="api-manager-window__search-input"
					v-model="searchQuery"
					placeholder="搜索..." />
				<button v-if="searchQuery" @click="searchQuery = ''" class="api-manager-window__search-clear">
					<xIcon icon="x" :size="14" />
				</button>
			</div>

			<div class="api-manager-window__content">
				<div v-if="loading" class="api-manager-window__loading">
					<xLoading />
				</div>

				<div v-else-if="error" class="api-manager-window__error">
					<xIcon icon="alert-circle" :size="32" />
					<p>{{ error }}</p>
				</div>

				<div v-else-if="isApi" class="api-manager-window__api-detail">
					<div class="api-manager-window__api-header">
						<span class="api-manager-window__api-method">{{ node?.content?.method }}</span>
						<span class="api-manager-window__api-endpoint">{{ node?.content?.endpoint }}</span>
					</div>
					<div class="api-manager-window__api-description">
						{{ node?.content?.description }}
					</div>
					<div v-if="node?.content?.params" class="api-manager-window__api-section">
						<h4>参数</h4>
						<ul>
							<li v-for="param in node.content.params" :key="param.name">
								<strong>{{ param.name }}</strong> ({{ param.type }}): {{ param.default || '' }}
							</li>
						</ul>
					</div>
					<div v-if="node?.content?.body" class="api-manager-window__api-section">
						<h4>请求体</h4>
						<pre>{{ node.content.body }}</pre>
					</div>
				</div>

				<div v-else-if="isFolder" class="api-manager-window__list">
					<div v-if="viewMode === 'list'" class="api-manager-window__table-header">
						<div @click="handleSort('name')" class="api-manager-window__table-col api-manager-window__table-col--name">
							名称
							<xIcon v-if="sortField === 'name'" :icon="sortDirection === 'asc' ? 'arrow-up' : 'arrow-down'" :size="12" />
						</div>
						<div v-if="isProjectFolder" class="api-manager-window__table-col">可见性</div>
						<div @click="handleSort('updatedAt')" class="api-manager-window__table-col">
							更新时间
							<xIcon v-if="sortField === 'updatedAt'" :icon="sortDirection === 'asc' ? 'arrow-up' : 'arrow-down'" :size="12" />
						</div>
					</div>

					<div v-if="filteredChildren.length === 0" class="api-manager-window__empty">
						<xIcon icon="_folder-open" :size="48" />
						<p>暂无内容</p>
					</div>

					<div v-else class="api-manager-window__items">
						<div 
							v-for="child in filteredChildren" 
							:key="child.id"
							@click="selectChild(child)"
							@dblclick="openChild(child)"
							class="api-manager-window__item"
							:class="{ 'api-manager-window__item--selected': selectedChild?.id === child.id }">
							<div class="api-manager-window__item-icon">
								<xIcon :icon="utils.isFolderType(child.type) ? 'folder' : 'file'" :size="18" />
							</div>
							<div class="api-manager-window__item-name">{{ child.name }}</div>
							<div v-if="isProjectFolder" class="api-manager-window__item-visibility">
								<span :class="getVisibilityBadgeClass(child.visibility)">
									{{ child.visibility === "private" ? "私有" : "公开" }}
								</span>
							</div>
							<div class="api-manager-window__item-date">{{ formatDate(child.updatedAt) }}</div>
						</div>
					</div>
				</div>

				<div v-else class="api-manager-window__detail">
					<h3>{{ node?.name }}</h3>
					<p>类型: {{ node?.type }}</p>
					<p>路径: {{ node?.path }}</p>
				</div>
			</div>
		</div>

		<template #footer>
			<xBtn :configs="{ label: '关闭', onClick: close }" />
		</template>
	</xDialog>
</template>

<style lang="less">
.api-manager-window {
	display: flex;
	flex-direction: column;
	height: 100%;

	&__header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 12px 16px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		background: var(--color-surface-container);
	}

	&__breadcrumb {
		display: flex;
		align-items: center;
		gap: 8px;
	}

	&__back-btn {
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
			color: var(--color-on-surface);
		}
	}

	&__title {
		font-size: 14px;
		font-weight: 600;
		color: var(--color-on-surface);
	}

	&__actions {
		display: flex;
		gap: 4px;
	}

	&__view-btn {
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
		}

		&--active {
			background: var(--color-primary-container);
			color: var(--color-on-primary-container);
		}
	}

	&__search {
		display: flex;
		align-items: center;
		padding: 8px 16px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 30%, transparent);
	}

	&__search-input {
		flex: 1;
		padding: 6px 10px;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 8px;
		font-size: 13px;
		outline: none;
		background: var(--color-surface-container-lowest);
		color: var(--color-on-surface);

		&:focus {
			border-color: var(--color-primary);
		}
	}

	&__search-clear {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 24px;
		height: 24px;
		margin-left: 8px;
		border: 0;
		background: transparent;
		color: var(--color-on-surface-variant);
		cursor: pointer;
		border-radius: 6px;

		&:hover {
			background: var(--color-surface-variant);
		}
	}

	&__content {
		flex: 1;
		overflow-y: auto;
		padding: 16px;
	}

	&__loading {
		display: flex;
		align-items: center;
		justify-content: center;
		height: 100%;
	}

	&__error {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100%;
		color: var(--color-error);
	}

	&__api-detail {
		padding: 16px;
		background: var(--color-surface-container);
		border-radius: 12px;
	}

	&__api-header {
		display: flex;
		align-items: center;
		gap: 12px;
		margin-bottom: 16px;
	}

	&__api-method {
		padding: 4px 12px;
		border-radius: 6px;
		font-size: 12px;
		font-weight: 700;
		background: color-mix(in srgb, var(--color-success) 15%, transparent);
		color: var(--color-success);
	}

	&__api-endpoint {
		font-size: 16px;
		font-weight: 600;
		color: var(--color-on-surface);
	}

	&__api-description {
		margin-bottom: 16px;
		color: var(--color-on-surface-variant);
	}

	&__api-section {
		margin-bottom: 16px;

		h4 {
			margin: 0 0 8px 0;
			font-size: 13px;
			font-weight: 600;
			color: var(--color-on-surface);
		}

		ul {
			margin: 0;
			padding-left: 20px;
		}

		li {
			margin-bottom: 4px;
			color: var(--color-on-surface-variant);
		}

		pre {
			margin: 0;
			padding: 12px;
			background: var(--color-surface-container-lowest);
			border-radius: 8px;
			font-family: monospace;
			font-size: 12px;
			overflow-x: auto;
			color: var(--color-on-surface);
		}
	}

	&__list {
		height: 100%;
	}

	&__table-header {
		display: grid;
		grid-template-columns: 3fr 1fr 1fr;
		gap: 16px;
		padding: 8px 0;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		font-size: 12px;
		font-weight: 700;
		color: var(--color-on-surface-variant);
		cursor: pointer;
		user-select: none;

		&:hover {
			color: var(--color-on-surface);
		}
	}

	&__table-col {
		display: flex;
		align-items: center;
		gap: 4px;

		&--name {
			grid-column: 1;
		}
	}

	&__empty {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 200px;
		color: var(--color-on-surface-variant);
	}

	&__items {
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	&__item {
		display: grid;
		grid-template-columns: 3fr 1fr 1fr;
		gap: 16px;
		padding: 8px;
		border-radius: 8px;
		cursor: pointer;
		transition: background-color 0.15s ease;
		align-items: center;

		&:hover {
			background: color-mix(in srgb, var(--color-surface-variant) 30%, transparent);
		}

		&--selected {
			background: color-mix(in srgb, var(--color-primary-container) 30%, transparent);
		}
	}

	&__item-icon {
		display: flex;
		align-items: center;
		color: var(--color-on-surface-variant);
	}

	&__item-name {
		color: var(--color-on-surface);
	}

	&__item-visibility {
		display: flex;
		justify-content: center;
	}

	&__item-date {
		display: flex;
		justify-content: flex-end;
		color: var(--color-on-surface-variant);
		font-size: 12px;
	}

	&__detail {
		padding: 16px;
		background: var(--color-surface-container);
		border-radius: 12px;
	}
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
</style>