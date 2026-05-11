<template>
	<div class="MemberIdentity">
		<xItem
			:configs="{
				value: member?.uid || '',
				itemType: 'xspaceItemAvatar',
				disabled: true
			}"
			class="MemberIdentity__avatar" />
		<div class="MemberIdentity__text">
			<div class="MemberIdentity__name">{{ cptName }}</div>
			<div class="MemberIdentity__sub">UID: {{ member?.uid || "--" }}</div>
		</div>
	</div>
</template>

<script lang="ts">
export default async function () {
	return {
		props: {
			member: {
				type: Object,
				default: null
			}
		},
		computed: {
			cptName() {
				const m = this.member || {};
				return m.username || m.name || (m.uid ? `${i18n("用户")} ${m.uid}` : "--");
			}
		}
	};
}
</script>

<style lang="less">
.MemberIdentity {
	display: flex;
	align-items: center;
	gap: 12px;
	min-width: 0;
}

.MemberIdentity__avatar {
	width: 28px;
	flex: 0 0 28px;
}

.MemberIdentity__text {
	display: flex;
	flex-direction: column;
	gap: 2px;
	min-width: 0;
}

.MemberIdentity__name {
	overflow: hidden;
	text-overflow: ellipsis;
	white-space: nowrap;
}

.MemberIdentity__sub {
	font-size: 12px;
	font-weight: 600;
	color: var(--color-on-surface-variant);
}
</style>
