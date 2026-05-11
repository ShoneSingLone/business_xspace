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
			const vm = this;
			return {
				logs: [],
				isLoading: false,
				searchQuery: "",
				configsSearch: defItem({
					isSearch: false,
					value: "",
					placeholder: "搜索日志...",
					onEmitValue() {
						vm.searchQuery = vm.configsSearch.value;
					},
					clearable: true
				}),
				configsTable: {
					onQuery() {
						vm.loadLogs();
					},
					pagination: {
						page: 1,
						size: 10,
						total: 0
					}
				}
			};
		},
		mounted() {
			this.loadLogs();
		},
		computed: {
			filteredLogs() {
				if (!this.searchQuery) return this.logs;
				const query = this.searchQuery.toLowerCase();
				return this.logs.filter(log => 
					log.content?.toLowerCase().includes(query) ||
					log.username?.toLowerCase().includes(query)
				);
			}
		},
		methods: {
			async loadLogs() {
				if (!this.project?.id) return;
				
				this.isLoading = true;
				try {
					const response = await api.getLogList({
						type: 'project',
						typeid: this.project.id,
						page: this.configsTable.pagination.page,
						limit: this.configsTable.pagination.size
					});
					if (response.errcode === 0) {
						this.logs = response.data?.list || [];
						this.configsTable.pagination.total = response.data?.total || 0;
					}
				} catch (error) {
					console.error("Failed to load logs:", error);
				} finally {
					this.isLoading = false;
				}
			},
			getTime(dateString) {
				if (!dateString) return '-';
				try {
					const date = new Date(dateString);
					return date.toLocaleString("zh-CN", {
						year: "numeric",
						month: "short",
						day: "numeric",
						hour: "2-digit",
						minute: "2-digit"
					});
				} catch (e) {
					return dateString;
				}
			},
			getTimeAgo(dateString) {
				if (!dateString) return '-';
				try {
					const date = new Date(dateString);
					const now = new Date();
					const diff = now.getTime() - date.getTime();
					const minutes = Math.floor(diff / (1000 * 60));
					const hours = Math.floor(diff / (1000 * 60 * 60));
					const days = Math.floor(diff / (1000 * 60 * 60 * 24));
					
					if (minutes < 60) return `${minutes}分钟前`;
					if (hours < 24) return `${hours}小时前`;
					return `${days}天前`;
				} catch (e) {
					return dateString;
				}
			},
			getTitle(action) {
				const names = {
					add: '新增',
					edit: '编辑',
					delete: '删除'
				};
				return names[action] || action;
			},
			hasDiff(data) {
				return false;
			},
			showDiff(data) {
				console.log('Show diff:', data);
			}
		}
	};
}
</script>

<template>
	<div class="project-logs">
		<div class="project-logs__header">
			<xItem :configs="configsSearch" class="flex1" />
		</div>

		<div v-if="isLoading" class="project-logs__loading">
			<xIcon icon="_loader-2" :size="24" class="project-logs__loading-icon" />
			<span>加载中...</span>
		</div>

		<div v-else-if="filteredLogs.length === 0" class="project-logs__empty">
			<xIcon icon="activity" :size="48" class="project-logs__empty-icon" />
			<p>暂无日志</p>
		</div>

		<div v-else class="project-logs__content">
			<section class="mb mt el-card x-padding flex1 log-wrapper beautiful-scroll">
				<xTimeline>
					<xTimelineItem
						center
						:timestamp="getTime(log.add_time)"
						placement="top"
						v-for="(log, index) in filteredLogs"
						:key="index">
						<xCard>
							<template #header>
								<div class="logtype flex middle">
									<span class="logHead">{{ getTitle(log.action) }}</span>
									<span class="logtime ml mr">{{ getTimeAgo(log.add_time) }}</span>
									<xBtn @click="showDiff(log.data)" v-if="hasDiff(log.data)">改动详情</xBtn>
								</div>
							</template>
							<span class="logcontent" v-html="log.content" />
						</xCard>
					</xTimelineItem>
				</xTimeline>
			</section>
			<div class="flex end">
				<xPagination :configs="configsTable" />
			</div>
		</div>
	</div>
</template>

<style lang="less">
.project-logs {
	height: 100%;
	display: flex;
	flex-direction: column;

	&__header {
		display: flex;
		align-items: center;
		gap: 16px;
		margin-bottom: 16px;
	}

	&__loading,
	&__empty {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		color: var(--color-on-surface-variant);
	}

	&__loading-icon,
	&__empty-icon {
		margin-bottom: 12px;
		color: var(--color-on-surface-variant);
	}

	&__content {
		flex: 1;
		display: flex;
		flex-direction: column;
		overflow: hidden;
	}

	.log-wrapper {
		flex: 1;
		overflow-y: auto;
	}

	.logtype {
		gap: 8px;
	}

	.logHead {
		font-weight: 600;
	}

	.logtime {
		font-size: 12px;
		color: var(--color-on-surface-variant);
	}

	.logcontent {
		font-size: 14px;
		color: var(--color-on-surface);
	}
}
</style>
