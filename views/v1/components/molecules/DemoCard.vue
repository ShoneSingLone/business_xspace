<template>
	<div class="DemoCard">
		<div class="DemoCard__header">
			<div class="DemoCard__title">{{ title }}</div>
			<div class="DemoCard__desc" v-if="desc">{{ desc }}</div>
		</div>
		<div class="DemoCard__body">
			<slot />
		</div>
	</div>
</template>

<script lang="ts">
export default async function () {
	return {
		props: {
			title: {
				type: String,
				default: ""
			},
			desc: {
				type: String,
				default: ""
			}
		}
	};
}
</script>

<style lang="less">
.DemoCard {
	border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	border-radius: 14px;
	background: var(--color-surface-container-lowest);
	overflow: hidden;
}

.DemoCard__header {
	padding: 12px 14px;
	border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	background: color-mix(in srgb, var(--color-surface-container) 30%, transparent);
}

.DemoCard__title {
	font-size: 13px;
	font-weight: 800;
	color: var(--color-on-surface);
}

.DemoCard__desc {
	margin-top: 4px;
	font-size: 12px;
	font-weight: 600;
	color: var(--color-on-surface-variant);
}

.DemoCard__body {
	padding: 14px;
}
</style>
