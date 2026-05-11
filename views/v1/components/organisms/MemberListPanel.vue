<template>
	<div class="MemberListPanel">
		<div class="MemberListPanel__header">
			<div class="MemberListPanel__title">
				<h3 class="MemberListPanel__h">{{ i18n("成员") }}</h3>
				<div class="MemberListPanel__meta">
					<span class="MemberListPanel__group">{{ groupName || "--" }}</span>
					<span class="MemberListPanel__count">{{ i18n("共") }} {{ (members || []).length }} {{ i18n("人") }}</span>
				</div>
			</div>
			<button
				v-if="canManage"
				class="api-manager__action-btn api-manager__action-btn--primary api-manager__action-btn--with-icon"
				@click="$emit('add-member')">
				<xIcon icon="plus" :size="16" />
				{{ i18n("添加成员") }}
			</button>
		</div>

		<div class="MemberListPanel__table-shell">
			<div v-if="loading" class="api-manager__state-panel api-manager__state-panel--loading">
				<xIcon icon="_loader-2" class="api-manager__spinner api-manager__spinner--lg" />
				<p class="api-manager__state-text">{{ i18n("加载中") }}...</p>
			</div>
			<div v-else-if="(members || []).length === 0" class="api-manager__state-panel api-manager__state-panel--dashed">
				<p class="api-manager__state-text">{{ i18n("暂无成员") }}</p>
			</div>

			<table v-else class="api-manager__table">
				<thead class="api-manager__table-head">
					<tr>
						<th class="api-manager__th">{{ i18n("名称") }}</th>
						<th class="api-manager__th">{{ i18n("角色") }}</th>
						<th class="api-manager__th">{{ i18n("邮箱") }}</th>
						<th v-if="canManage" class="api-manager__th api-manager__th--right">{{ i18n("操作") }}</th>
					</tr>
				</thead>
				<tbody>
					<tr v-for="(member, idx) in members" :key="idx" class="api-manager__tr">
						<td class="api-manager__td api-manager__td--strong">
							<MemberIdentity :member="member" />
						</td>
						<td class="api-manager__td">
							<select
								v-if="canManage"
								:value="member.role"
								@change="$emit('change-member-role', { member, role: $event.target.value })"
								class="api-manager__field-input api-manager__field-input--inline">
								<option value="owner">{{ i18n("组长") }}</option>
								<option value="dev">{{ i18n("开发者") }}</option>
								<option value="guest">{{ i18n("访客") }}</option>
							</select>
							<span v-else class="api-manager__inline-badge">{{ cptRoleText(member.role) }}</span>
						</td>
						<td class="api-manager__td api-manager__td--muted">
							<span>{{ member.email || "--" }}</span>
						</td>
						<td v-if="canManage" class="api-manager__td api-manager__td--right">
							<button class="api-manager__icon-action" @click="$emit('remove-member', member)" :title="i18n('移除成员')">
								<xIcon icon="delete" :size="16" />
							</button>
						</td>
					</tr>
				</tbody>
			</table>
		</div>
	</div>
</template>

<script lang="ts">
export default async function () {
	return {
		components: {
			MemberIdentity: () => _.$importVue("@/views/v1/components/molecules/MemberIdentity.vue")
		},
		props: {
			groupName: {
				type: String,
				default: ""
			},
			members: {
				type: Array,
				default: () => []
			},
			loading: {
				type: Boolean,
				default: false
			},
			canManage: {
				type: Boolean,
				default: false
			}
		},
		methods: {
			cptRoleText(role) {
				const ROLE_MAP = {
					owner: i18n("组长"),
					dev: i18n("开发者"),
					guest: i18n("访客"),
					admin: i18n("管理员")
				};
				return ROLE_MAP[String(role || "").toLowerCase()] || String(role || "--");
			}
		}
	};
}
</script>

<style lang="less">
.MemberListPanel__header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	gap: 12px;
	margin-bottom: 12px;
}

.MemberListPanel__title {
	display: flex;
	flex-direction: column;
	gap: 4px;
}

.MemberListPanel__h {
	margin: 0;
	font-size: 14px;
	font-weight: 700;
	color: var(--color-on-surface);
}

.MemberListPanel__meta {
	display: flex;
	align-items: center;
	gap: 10px;
	color: var(--color-on-surface-variant);
	font-size: 12px;
	font-weight: 600;
}

.MemberListPanel__group {
	color: var(--color-on-surface);
	font-weight: 800;
}

.MemberListPanel__count {
	padding: 2px 8px;
	border-radius: 999px;
	border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
	background: var(--color-surface-container-lowest);
}

.MemberListPanel__table-shell {
	border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	border-radius: 14px;
	overflow: hidden;
	background: var(--color-surface-container-lowest);
}
</style>
