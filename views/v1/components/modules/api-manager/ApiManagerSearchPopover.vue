<script lang="ts">
export default async function () {
	return {
		props: {
			show: {
				type: Boolean,
				default: false
			},
			searchQuery: {
				type: String,
				default: ""
			},
			searchResults: {
				type: Array,
				default: () => []
			},
			isLoading: {
				type: Boolean,
				default: false
			},
			position: {
				type: Object,
				default: () => ({ top: 0, left: 0 })
			}
		},
		data() {
			return {
				selectedIndex: 0,
				queryType: "contains"
			};
		},
		computed: {
			groupedResults() {
				const groups = {
					groups: { label: "分组", icon: "folder", items: [] },
					projects: { label: "项目", icon: "folder", items: [] },
					apis: { label: "API", icon: "code", items: [] },
					docs: { label: "文档", icon: "file-text", items: [] },
					other: { label: "其他", icon: "file", items: [] }
				};

				this.searchResults.forEach(result => {
					switch (result.type) {
						case "group":
							groups.groups.items.push(result);
							break;
						case "project":
							groups.projects.items.push(result);
							break;
						case "api":
							groups.apis.items.push(result);
							break;
						case "doc":
							groups.docs.items.push(result);
							break;
						default:
							groups.other.items.push(result);
					}
				});

				return Object.values(groups).filter(g => g.items.length > 0);
			},
			flattenedResults() {
				return this.searchResults;
			}
		},
		methods: {
			selectResult(result) {
				this.$emit("select", result);
			},
			handleKeyDown(event) {
				switch (event.key) {
					case "ArrowDown":
						event.preventDefault();
						this.selectedIndex = Math.min(this.selectedIndex + 1, this.flattenedResults.length - 1);
						break;
					case "ArrowUp":
						event.preventDefault();
						this.selectedIndex = Math.max(this.selectedIndex - 1, 0);
						break;
					case "Enter":
						if (this.flattenedResults[this.selectedIndex]) {
							this.selectResult(this.flattenedResults[this.selectedIndex]);
						}
						break;
					case "Escape":
						this.$emit("close");
						break;
				}
			},
			getIcon(type) {
				switch (type) {
					case "group": return "folder";
					case "project": return "folder";
					case "api_folder": return "folder";
					case "doc_folder": return "folder";
					case "folder": return "folder";
					case "api": return "code";
					case "doc": return "file-text";
					case "code": return "code";
					default: return "file";
				}
			},
			getIconClass(type) {
				switch (type) {
					case "group": return "api-manager__search-result-icon--group";
					case "project": return "api-manager__search-result-icon--project";
					case "api": return "api-manager__search-result-icon--api";
					case "doc": return "api-manager__search-result-icon--doc";
					default: return "api-manager__search-result-icon--default";
				}
			},
			getTypeLabel(type) {
				switch (type) {
					case "group": return "分组";
					case "project": return "项目";
					case "api": return "API";
					case "doc": return "文档";
					case "api_folder": return "API文件夹";
					case "doc_folder": return "文档文件夹";
					case "folder": return "文件夹";
					case "personal": return "个人空间";
					default: return "其他";
				}
			},
			getTypeClass(type) {
				return `api-manager__search-result-type--${type}`;
			},
			highlightMatch(text, query) {
				if (!query) return text;
				const regex = new RegExp(`(${query})`, "gi");
				return text.replace(regex, "<mark>$1</mark>");
			}
		},
		watch: {
			show(newVal) {
				if (newVal) {
					this.selectedIndex = 0;
				}
			},
			searchQuery() {
				this.selectedIndex = 0;
			}
		}
	};
}
</script>

<template>
	<div 
		v-if="show" 
		class="api-manager__search-popover"
		:style="{ top: position.top + 'px', left: position.left + 'px' }"
		@keydown="handleKeyDown"
		tabindex="0"
	>
		<div class="api-manager__search-popover-header">
			<div class="api-manager__search-popover-query-type">
				<button 
					v-for="type in [{ value: 'contains', label: '模糊' }, { value: 'exact', label: '精确' }]"
					:key="type.value"
					@click="queryType = type.value"
					:class="{ 
						'api-manager__search-query-type-btn': true,
						'api-manager__search-query-type-btn--active': queryType === type.value
					}"
				>
					{{ type.label }}
				</button>
			</div>
			<div class="api-manager__search-popover-result-count">
				{{ searchResults.length }} 个结果
			</div>
		</div>

		<div class="api-manager__search-popover-content">
			<div v-if="isLoading" class="api-manager__search-popover-loading">
				<xIcon icon="_loader-2" :size="20" class="api-manager__search-loading-icon" />
				<span>搜索中...</span>
			</div>

			<div v-else-if="searchResults.length === 0" class="api-manager__search-popover-empty">
				<xIcon icon="search" :size="32" class="api-manager__search-empty-icon" />
				<p>未找到匹配的结果</p>
				<p class="api-manager__search-empty-hint">尝试使用其他关键词</p>
			</div>

			<div v-else class="api-manager__search-popover-results">
				<div v-for="group in groupedResults" :key="group.label" class="api-manager__search-result-group">
					<div class="api-manager__search-result-group-header">
						<xIcon :icon="group.icon" :size="14" />
						<span>{{ group.label }} ({{ group.items.length }})</span>
					</div>
					<div 
						v-for="(result, index) in group.items" 
						:key="result.id"
						class="api-manager__search-result-item"
						:class="{ 'api-manager__search-result-item--selected': flattenedResults.indexOf(result) === selectedIndex }"
						@click="selectResult(result)"
						@mouseenter="selectedIndex = flattenedResults.indexOf(result)"
					>
						<span class="api-manager__search-result-icon">
							<xIcon :icon="getIcon(result.type)" :size="16" :class="getIconClass(result.type)" />
						</span>
						<div class="api-manager__search-result-info">
							<div class="api-manager__search-result-name" v-html="highlightMatch(result.name, searchQuery)"></div>
							<div class="api-manager__search-result-path">{{ result.path }}</div>
						</div>
						<span :class="['api-manager__search-result-type', getTypeClass(result.type)]">
							{{ getTypeLabel(result.type) }}
						</span>
					</div>
				</div>
			</div>
		</div>

		<div class="api-manager__search-popover-footer">
			<span class="api-manager__search-popover-hint">
				<xIcon icon="arrow-up" :size="12" />
				<xIcon icon="arrow-down" :size="12" />
				导航
				<xIcon icon="enter" :size="12" />
				打开
				<xIcon icon="x" :size="12" />
				关闭
			</span>
		</div>
	</div>
</template>

<style lang="less">
.api-manager__search-popover {
	position: fixed;
	z-index: 1000;
	width: 560px;
	max-height: 500px;
	background: var(--color-surface);
	border-radius: 12px;
	box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
	border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	display: flex;
	flex-direction: column;
	overflow: hidden;
}

.api-manager__search-popover-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 12px 16px;
	border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	background: var(--color-surface-container-low);
}

.api-manager__search-popover-query-type {
	display: flex;
	gap: 4px;
}

.api-manager__search-query-type-btn {
	padding: 4px 10px;
	border: 1px solid transparent;
	border-radius: 6px;
	background: transparent;
	color: var(--color-on-surface-variant);
	font-size: 12px;
	cursor: pointer;
	transition: all 0.15s ease;

	&:hover {
		background: var(--color-surface-variant);
	}

	&--active {
		border-color: var(--color-primary);
		background: var(--color-primary-container);
		color: var(--color-on-primary-container);
	}
}

.api-manager__search-popover-result-count {
	font-size: 12px;
	color: var(--color-on-surface-variant);
}

.api-manager__search-popover-content {
	flex: 1;
	overflow-y: auto;
	padding: 8px;
	min-height: 0;
}

.api-manager__search-popover-loading {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 40px;
	gap: 8px;
	color: var(--color-on-surface-variant);
}

.api-manager__search-loading-icon {
	animation: spin 1s linear infinite;
}

@keyframes spin {
	from { transform: rotate(0deg); }
	to { transform: rotate(360deg); }
}

.api-manager__search-popover-empty {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 40px;
	gap: 8px;
	color: var(--color-on-surface-variant);
}

.api-manager__search-empty-icon {
	color: var(--color-on-surface-variant);
}

.api-manager__search-empty-hint {
	font-size: 12px;
	color: var(--color-on-surface-variant);
	opacity: 0.7;
}

.api-manager__search-popover-results {
	display: flex;
	flex-direction: column;
	gap: 8px;
}

.api-manager__search-result-group {
	display: flex;
	flex-direction: column;
	gap: 4px;
}

.api-manager__search-result-group-header {
	display: flex;
	align-items: center;
	gap: 6px;
	padding: 4px 8px;
	font-size: 11px;
	font-weight: 600;
	text-transform: uppercase;
	letter-spacing: 0.05em;
	color: var(--color-on-surface-variant);
}

.api-manager__search-result-item {
	display: flex;
	align-items: center;
	gap: 10px;
	padding: 8px 12px;
	border-radius: 8px;
	cursor: pointer;
	transition: background-color 0.15s ease;

	&:hover {
		background: var(--color-surface-container-low);
	}

	&--selected {
		background: var(--color-primary-container);
	}
}

.api-manager__search-result-icon {
	flex-shrink: 0;

	&--group { color: #6750A4; }
	&--project { color: #3182ce; }
	&--api { color: #6a9955; }
	&--doc { color: #ce9178; }
	&--default { color: var(--color-on-surface-variant); }
}

.api-manager__search-result-info {
	flex: 1;
	min-width: 0;
	overflow: hidden;
}

.api-manager__search-result-name {
	font-size: 14px;
	color: var(--color-on-surface);
	font-weight: 500;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;

	mark {
		background: var(--color-primary-container);
		color: var(--color-on-primary-container);
		padding: 1px 2px;
		border-radius: 2px;
	}
}

.api-manager__search-result-path {
	font-size: 12px;
	color: var(--color-on-surface-variant);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.api-manager__search-result-type {
	flex-shrink: 0;
	padding: 2px 8px;
	border-radius: 12px;
	font-size: 11px;
	font-weight: 500;

	&--group {
		background: rgba(103, 80, 164, 0.15);
		color: #6750A4;
	}

	&--project {
		background: rgba(49, 130, 206, 0.15);
		color: #3182ce;
	}

	&--api {
		background: rgba(106, 153, 85, 0.15);
		color: #6a9955;
	}

	&--doc {
		background: rgba(206, 145, 120, 0.15);
		color: #ce9178;
	}

	&--default {
		background: var(--color-surface-variant);
		color: var(--color-on-surface-variant);
	}
}

.api-manager__search-popover-footer {
	padding: 8px 16px;
	border-top: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	background: var(--color-surface-container-low);
}

.api-manager__search-popover-hint {
	display: flex;
	align-items: center;
	gap: 8px;
	font-size: 11px;
	color: var(--color-on-surface-variant);

	.xIcon {
		opacity: 0.6;
	}
}
</style>