<script lang="ts">
export default async function () {
	const api = await _.$importVue("@/utils/api.vue");

	return {
		props: {
			project: {
				type: Object,
				default: () => ({})
			},
			projectData: {
				type: Object,
				default: () => ({})
			}
		},
		data() {
			return {
				interfaces: [],
				isLoading: false,
				searchQuery: "",
				viewMode: "list"
			};
		},
		mounted() {
			this.loadInterfaces();
		},
		computed: {
			filteredInterfaces() {
				if (!this.searchQuery) return this.interfaces;
				const query = this.searchQuery.toLowerCase();
				return this.interfaces.filter(item => 
					item.title?.toLowerCase().includes(query) ||
					item.path?.toLowerCase().includes(query)
				);
			}
		},
		methods: {
			async loadInterfaces() {
				if (!this.project?.id) return;
				
				this.isLoading = true;
				try {
					const response = await api.getInterfaceList({ 
						project_id: this.project.id,
						page: 1,
						limit: 100
					});
					if (response.errcode === 0) {
						this.interfaces = response.data?.list || [];
					}
				} catch (error) {
					console.error("Failed to load interfaces:", error);
				} finally {
					this.isLoading = false;
				}
			},
			getMethodColor(method) {
				const colors = {
					'GET': 'success',
					'POST': 'primary',
					'PUT': 'warning',
					'DELETE': 'error',
					'PATCH': 'secondary'
				};
				return colors[method?.toUpperCase()] || 'default';
			},
			formatDate(dateString) {
				if (!dateString) return '-';
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
			}
		}
	};
}
</script>

<template>
	<div class="project-interfaces">
		<div class="project-interfaces__header">
			<div class="project-interfaces__search">
				<xIcon icon="search" :size="16" class="project-interfaces__search-icon" />
				<input 
					v-model="searchQuery"
					type="text" 
					placeholder="搜索接口..."
					class="project-interfaces__search-input" />
			</div>
			<div class="project-interfaces__actions">
				<button class="project-interfaces__btn project-interfaces__btn--primary">
					<xIcon icon="plus" :size="16" />
					<span>新建接口</span>
				</button>
			</div>
		</div>

		<div v-if="isLoading" class="project-interfaces__loading">
			<xIcon icon="_loader-2" :size="24" class="project-interfaces__loading-icon" />
			<span>加载中...</span>
		</div>

		<div v-else-if="filteredInterfaces.length === 0" class="project-interfaces__empty">
			<xIcon icon="code" :size="48" class="project-interfaces__empty-icon" />
			<p>暂无接口</p>
			<p class="project-interfaces__empty-hint">点击上方按钮创建新接口</p>
		</div>

		<div v-else class="project-interfaces__list">
			<div class="project-interfaces__list-header">
				<div class="project-interfaces__list-cell project-interfaces__list-cell--method">方法</div>
				<div class="project-interfaces__list-cell project-interfaces__list-cell--path">路径</div>
				<div class="project-interfaces__list-cell project-interfaces__list-cell--title">标题</div>
				<div class="project-interfaces__list-cell project-interfaces__list-cell--updated">更新时间</div>
			</div>
			<div class="project-interfaces__list-body">
				<div 
					v-for="item in filteredInterfaces" 
					:key="item._id"
					class="project-interfaces__list-row">
					<div class="project-interfaces__list-cell project-interfaces__list-cell--method">
						<span :class="`project-interfaces__method-badge project-interfaces__method-badge--${getMethodColor(item.method)}`">
							{{ item.method || 'GET' }}
						</span>
					</div>
					<div class="project-interfaces__list-cell project-interfaces__list-cell--path">
						<code>{{ item.path }}</code>
					</div>
					<div class="project-interfaces__list-cell project-interfaces__list-cell--title">
						{{ item.title }}
					</div>
					<div class="project-interfaces__list-cell project-interfaces__list-cell--updated">
						{{ formatDate(item.up_time) }}
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style lang="less">
.project-interfaces {
	height: 100%;
	display: flex;
	flex-direction: column;

	&__header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: 16px;
		margin-bottom: 16px;
	}

	&__search {
		flex: 1;
		max-width: 400px;
		position: relative;
	}

	&__search-icon {
		position: absolute;
		left: 12px;
		top: 50%;
		transform: translateY(-50%);
		color: var(--color-on-surface-variant);
	}

	&__search-input {
		width: 100%;
		padding: 8px 12px 8px 36px;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 8px;
		font-size: 14px;
		background: var(--color-surface-container-low);
		color: var(--color-on-surface);

		&:focus {
			outline: none;
			border-color: var(--color-primary);
		}

		&::placeholder {
			color: var(--color-on-surface-variant);
		}
	}

	&__actions {
		display: flex;
		gap: 8px;
	}

	&__btn {
		display: inline-flex;
		align-items: center;
		gap: 6px;
		padding: 8px 16px;
		border: 0;
		border-radius: 8px;
		font-size: 14px;
		font-weight: 600;
		cursor: pointer;
		transition: all 0.15s ease;

		&--primary {
			background: var(--color-primary);
			color: var(--color-on-primary);

			&:hover {
				background: var(--color-primary-container);
				color: var(--color-on-primary-container);
			}
		}
	}

	&__loading {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		gap: 8px;
		color: var(--color-on-surface-variant);
	}

	&__loading-icon {
		animation: spin 1s linear infinite;
	}

	&__empty {
		flex: 1;
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

	&__empty-icon {
		color: var(--color-outline-variant);
	}

	&__empty-hint {
		font-size: 12px;
		opacity: 0.7;
	}

	&__list {
		flex: 1;
		overflow: hidden;
		display: flex;
		flex-direction: column;
		border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		border-radius: 12px;
		background: var(--color-surface);
	}

	&__list-header {
		display: grid;
		grid-template-columns: 100px 2fr 2fr 1fr;
		gap: 16px;
		padding: 12px 16px;
		background: var(--color-surface-container);
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
		font-size: 12px;
		font-weight: 700;
		text-transform: uppercase;
		letter-spacing: 0.05em;
		color: var(--color-on-surface-variant);
	}

	&__list-body {
		flex: 1;
		overflow-y: auto;
	}

	&__list-row {
		display: grid;
		grid-template-columns: 100px 2fr 2fr 1fr;
		gap: 16px;
		padding: 12px 16px;
		border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 30%, transparent);
		font-size: 14px;
		cursor: pointer;
		transition: background-color 0.15s ease;

		&:hover {
			background: var(--color-surface-variant);
		}

		&:last-child {
			border-bottom: none;
		}
	}

	&__list-cell {
		display: flex;
		align-items: center;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;

		&--path {
			code {
				font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
				font-size: 13px;
				color: var(--color-primary);
				background: var(--color-primary-container);
				padding: 2px 6px;
				border-radius: 4px;
			}
		}
	}

	&__method-badge {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		padding: 4px 8px;
		border-radius: 4px;
		font-size: 11px;
		font-weight: 700;
		font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;

		&--success {
			background: color-mix(in srgb, var(--color-success) 15%, transparent);
			color: var(--color-success);
		}

		&--primary {
			background: color-mix(in srgb, var(--color-primary) 15%, transparent);
			color: var(--color-primary);
		}

		&--warning {
			background: color-mix(in srgb, var(--color-warning) 15%, transparent);
			color: var(--color-warning);
		}

		&--error {
			background: color-mix(in srgb, var(--color-error) 15%, transparent);
			color: var(--color-error);
		}

		&--secondary {
			background: color-mix(in srgb, var(--color-secondary) 15%, transparent);
			color: var(--color-secondary);
		}
	}
}

@keyframes spin {
	from {
		transform: rotate(0deg);
	}
	to {
		transform: rotate(360deg);
	}
}
</style>
