<template>
	<div
		class="xspaceExplorerCard"
		:class="cptClass"
		@click="$emit('click', $event)"
		@dblclick="$emit('dblclick', $event)"
		@contextmenu="$emit('contextmenu', $event)">
		<div class="xspaceExplorerCard__icon">
			<slot name="icon" />
		</div>
		<div class="xspaceExplorerCard__content">
			<div class="xspaceExplorerCard__title">
				<slot name="title" />
			</div>
			<div class="xspaceExplorerCard__metaRow">
				<div class="xspaceExplorerCard__meta">
					<slot name="meta" />
				</div>
				<div class="xspaceExplorerCard__date">
					<slot name="date" />
				</div>
			</div>
		</div>
		<div v-if="hasActions" class="xspaceExplorerCard__actions">
			<slot name="actions" />
		</div>
	</div>
</template>

<script lang="ts">
export default async function () {
	return defineComponent({
		props: {
			selected: {
				type: Boolean,
				default: false
			}
		},
		computed: {
			hasActions() {
				return Boolean(this.$slots.actions);
			},
			cptClass() {
				return {
					"xspaceExplorerCard--selected": this.selected,
					"xspaceExplorerCard--has-actions": this.hasActions
				};
			}
		}
	});
}
</script>

<style lang="less">
.xspaceExplorerCard {
	display: flex;
	align-items: center;
	gap: 10px;
	padding: 8px 10px;
	border-radius: 12px;
	background: var(--color-surface-container);
	border: 1px solid color-mix(in srgb, var(--color-outline-variant) 35%, transparent);
	cursor: pointer;
	user-select: none;
	transition: background-color 0.15s ease, border-color 0.15s ease, transform 0.15s ease;

	&:hover {
		background: color-mix(in srgb, var(--color-surface-variant) 25%, transparent);
		border-color: color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
		transform: translateY(-1px);
	}

	&--selected {
		background: color-mix(in srgb, var(--color-primary-container) 28%, transparent);
		border-color: color-mix(in srgb, var(--color-primary) 22%, transparent);
	}

	&__icon {
		flex: none;
		width: 34px;
		height: 34px;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 10px;
		background: color-mix(in srgb, var(--color-surface-variant) 55%, transparent);
	}

	&__content {
		flex: 1;
		min-width: 0;
		display: flex;
		flex-direction: column;
		gap: 2px;
	}

	&__title {
		font-size: 13px;
		font-weight: 700;
		color: var(--color-on-surface);
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		line-height: 18px;
	}

	&__metaRow {
		display: flex;
		align-items: center;
		gap: 8px;
		min-width: 0;
	}

	&__meta {
		flex: 1;
		min-width: 0;
		font-size: 12px;
		color: var(--color-on-surface-variant);
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	&__date {
		flex: none;
		font-size: 11px;
		color: color-mix(in srgb, var(--color-on-surface-variant) 85%, transparent);
		white-space: nowrap;
	}

	&__actions {
		flex: none;
		display: flex;
		align-items: center;
		gap: 6px;
		opacity: 0;
		pointer-events: none;
		transition: opacity 0.15s ease;
	}

	&:hover &__actions,
	&--selected &__actions {
		opacity: 1;
		pointer-events: auto;
	}
}
</style>

