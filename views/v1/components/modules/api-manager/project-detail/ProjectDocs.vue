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
				docs: [],
				isLoading: false,
				searchQuery: ""
			};
		},
		mounted() {
			this.loadDocs();
		},
		computed: {
			filteredDocs() {
				if (!this.searchQuery) return this.docs;
				const query = this.searchQuery.toLowerCase();
				return this.docs.filter(doc => 
					doc.title?.toLowerCase().includes(query) ||
					doc.desc?.toLowerCase().includes(query)
				);
			}
		},
		methods: {
			async loadDocs() {
				if (!this.project?.id) return;
				
				this.isLoading = true;
				try {
					const response = await api.getDocList({
						project_id: this.project.id,
						page: 1,
						limit: 100
					});
					if (response.errcode === 0) {
						this.docs = response.data?.list || [];
					}
				} catch (error) {
					console.error("Failed to load docs:", error);
				} finally {
					this.isLoading = false;
				}
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
	<div class="project-docs">
		<div class="project-docs__header">
			<div class="project-docs__search">
				<xIcon icon="search" :size="16" class="project-docs__search-icon" />
				<input 
					v-model="searchQuery"
					type="text" 
					placeholder="搜索文档..."
					class="project-docs__search-input" />
			</div>
			<div class="project-docs__actions">
				<button class="project-docs__btn project-docs__btn--primary">
					<xIcon icon="file-plus" :size="16" />
					<span>新建文档</span>
				</button>
			</div>
		</div>

		<div v-if="isLoading" class="project-docs__loading">
			<xIcon icon="_loader-2" :size="24" class="project-docs__loading-icon" />
			<span>加载中...</span>
		</div>

		<div v-else-if="filteredDocs.length === 0" class="project-docs__empty">
			<xIcon icon="file-text" :size="48" class="project-docs__empty-icon" />
			<p>暂无文档</p>
			<p class="project-docs__empty-hint">点击上方按钮创建新文档</p>
		</div>

		<div v-else class="project-docs__list">
			<div class="project-docs__list-header">
				<div class="project-docs__list-cell project-docs__list-cell--title">标题</div>
				<div class="project-docs__list-cell project-docs__list-cell--desc">描述</div>
				<div class="project-docs__list-cell project-docs__list-cell--updated">更新时间</div>
			</div>
			<div class="project-docs__list-body">
				<div 
					v-for="doc in filteredDocs" 
					:key="doc._id"
					class="project-docs__list-row">
					<div class="project-docs__list-cell project-docs__list-cell--title">
						<xIcon icon="file-text" :size="16" class="project-docs__doc-icon" />
						<span class="project-docs__doc-title">{{ doc.title }}</span>
					</div>
					<div class="project-docs__list-cell project-docs__list-cell--desc">
						{{ doc.desc || '-' }}
					</div>
					<div class="project-docs__list-cell project-docs__list-cell--updated">
						{{ formatDate(doc.up_time) }}
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style lang="less">
.project-docs {
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
		grid-template-columns: 2fr 3fr 1fr;
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
		grid-template-columns: 2fr 3fr 1fr;
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

		&--title {
			gap: 8px;
		}
	}

	&__doc-icon {
		color: var(--color-secondary);
		flex-shrink: 0;
	}

	&__doc-title {
		font-weight: 600;
		color: var(--color-on-surface);
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
