<template>
	<section class="DemoSection" :id="id">
		<div class="DemoSection__head">
			<div class="DemoSection__title">{{ title }}</div>
			<div class="DemoSection__subtitle" v-if="subtitle">{{ subtitle }}</div>
		</div>
		<div class="DemoSection__body">
			<slot />
		</div>
	</section>
</template>

<script lang="ts">
export default async function () {
	return {
		props: {
			id: {
				type: String,
				default: ""
			},
			title: {
				type: String,
				default: ""
			},
			subtitle: {
				type: String,
				default: ""
			}
		}
	};
}
</script>

<style lang="less">
.DemoSection__head {
	display: flex;
	flex-direction: column;
	gap: 4px;
	margin-bottom: 12px;
}

.DemoSection__title {
	font-size: 16px;
	font-weight: 900;
	color: var(--color-on-surface);
}

.DemoSection__subtitle {
	font-size: 12px;
	font-weight: 600;
	color: var(--color-on-surface-variant);
}

.DemoSection__body {
	display: grid;
	grid-template-columns: repeat(2, minmax(0, 1fr));
	gap: 12px;
}

@media (max-width: 960px) {
	.DemoSection__body {
		grid-template-columns: 1fr;
	}
}
</style>
