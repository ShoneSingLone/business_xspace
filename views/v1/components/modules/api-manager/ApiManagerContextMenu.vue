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
				console.log("ApiManagerContextMenu: handleAction called", action, this.node?.name);
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
			<xIcon icon="_edit" :size="14" />
			<span>Rename</span>
		</div>
		<div class="api-manager__context-menu-item" @click="handleAction('move')">
			<xIcon icon="_move_file" :size="14" />
			<span>Move</span>
		</div>
		<div class="api-manager__context-menu-item api-manager__context-menu-item--danger" @click="handleAction('delete')">
			<xIcon icon="_delete" :size="14" />
			<span>Delete</span>
		</div>
		<div class="api-manager__context-menu-divider"></div>
		<div class="api-manager__context-menu-item" @click="handleAction('copyLink')">
			<xIcon icon="_copy" :size="14" />
			<span>Copy Link</span>
		</div>
		<div class="api-manager__context-menu-item" @click="handleAction('openNewTab')">
			<xIcon icon="_external-link" :size="14" />
			<span>Open in New Tab</span>
		</div>
		<div class="api-manager__context-menu-item" @click="handleAction('openNewTabWithAuth')">
			<xIcon icon="_lockStrok" :size="14" />
			<span>Open in New Tab (with Auth)</span>
		</div>
		<div class="api-manager__context-menu-item" @click="handleAction('openNewWindow')">
			<xIcon icon="_window-maximize" :size="14" />
			<span>Open in New Window</span>
		</div>
	</div>
</template>