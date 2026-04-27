<script lang="ts">
export default async function () {
	return {
		props: {
			searchQuery: {
				type: String,
				default: ""
			},
			activeEnvironment: {
				type: String,
				default: "Development"
			},
			viewMode: {
				type: String,
				default: "list"
			},
			showPreview: {
				type: Boolean,
				default: true
			},
			canNavigateUp: {
				type: Boolean,
				default: false
			},
			environments: {
				type: Array,
				default: () => ["Development", "Test", "Staging", "Production"]
			}
		},
		methods: {
			updateSearchQuery(event) {
				this.$emit("update:searchQuery", event.target.value);
			},
			updateEnvironment(event) {
				this.$emit("update:activeEnvironment", event.target.value);
			},
			updateViewMode(mode) {
				this.$emit("update:viewMode", mode);
			},
			togglePreview() {
				this.$emit("update:showPreview", !this.showPreview);
			},
			navigateUp() {
				this.$emit("navigateUp");
			}
		}
	};
}
</script>

<template>
	<div class="api-manager__topbar">
		<div class="api-manager__toolbar">
			<div class="api-manager__nav">
				<button class="api-manager__toolbar-btn api-manager__toolbar-btn--disabled" type="button">
					<xIcon icon="back" :size="20" />
				</button>
				<button class="api-manager__toolbar-btn api-manager__toolbar-btn--disabled" type="button">
					<xIcon icon="right" :size="20" />
				</button>
				<button 
					@click="navigateUp" 
					:disabled="!canNavigateUp" 
					class="api-manager__toolbar-btn"
					:class="{ 'api-manager__toolbar-btn--disabled': !canNavigateUp }" 
					title="Up">
					<xIcon icon="top" :size="20" />
				</button>
			</div>

			<div class="api-manager__search">
				<div class="api-manager__search-icon" aria-hidden="true">
					<xIcon icon="search" :size="16" />
				</div>
				<input 
					type="text" 
					placeholder="Search resources..." 
					:value="searchQuery"
					@input="updateSearchQuery"
					class="api-manager__search-input" />
			</div>

			<div class="api-manager__toolbar-right">
				<div class="api-manager__select-shell api-manager__select-shell--toolbar">
					<xIcon icon="tools" :size="16" class="api-manager__select-icon" />
					<select :value="activeEnvironment" @change="updateEnvironment" class="api-manager__select">
						<option v-for="env in environments" :key="env" :value="env">{{ env }}</option>
					</select>
				</div>

				<div class="api-manager__divider" aria-hidden="true"></div>

				<div class="api-manager__view-toggle">
					<button 
						@click="updateViewMode('list')" 
						class="api-manager__view-toggle-btn"
						:class="{ 'api-manager__view-toggle-btn--active': viewMode === 'list' }" 
						title="List View">
						<xIcon icon="list" :size="18" />
					</button>
					<button 
						@click="updateViewMode('grid')" 
						class="api-manager__view-toggle-btn"
						:class="{ 'api-manager__view-toggle-btn--active': viewMode === 'grid' }" 
						title="Grid View">
						<xIcon icon="grid" :size="18" />
					</button>
				</div>

				<button 
					@click="togglePreview" 
					class="api-manager__toggle-btn"
					:class="{ 'api-manager__toggle-btn--active': showPreview }" 
					title="Toggle Preview Pane">
					<xIcon icon="view" :size="20" />
				</button>
			</div>
		</div>
	</div>
</template>