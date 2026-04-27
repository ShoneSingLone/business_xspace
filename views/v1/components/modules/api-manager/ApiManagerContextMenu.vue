<script lang="ts">
export default async function () {
	return {
		props: {
			show: {
				type: Boolean,
				default: false
			},
			x: {
				type: Number,
				default: 0
			},
			y: {
				type: Number,
				default: 0
			},
			node: {
				type: Object,
				default: null
			}
		},
		methods: {
			handleAction(action) {
				this.$emit("action", action, this.node);
			},
			hide() {
				this.$emit("hide");
			}
		}
	};
}
</script>

<template>
	<div 
		v-if="show" 
		class="api-manager__context-menu"
		:style="{ left: x + 'px', top: y + 'px' }" 
		@click.stop>
		<div class="api-manager__context-menu-item" @click="handleAction('rename')">
			<xIcon icon="edit" :size="14" />
			<span>Rename</span>
		</div>
		<div class="api-manager__context-menu-item" @click="handleAction('move')">
			<xIcon icon="move" :size="14" />
			<span>Move</span>
		</div>
		<div class="api-manager__context-menu-item api-manager__context-menu-item--danger" @click="handleAction('delete')">
			<xIcon icon="delete" :size="14" />
			<span>Delete</span>
		</div>
		<div class="api-manager__context-menu-divider"></div>
		<div class="api-manager__context-menu-item" @click="handleAction('copyLink')">
			<xIcon icon="copy" :size="14" />
			<span>Copy Link</span>
		</div>
		<div class="api-manager__context-menu-item" @click="handleAction('openNewTab')">
			<xIcon icon="external-link" :size="14" />
			<span>Open in New Tab</span>
		</div>
	</div>
</template>