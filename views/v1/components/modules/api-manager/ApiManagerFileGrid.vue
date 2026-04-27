<script lang="ts">
export default async function () {
	return {
		props: {
			files: {
				type: Array,
				default: () => []
			},
			selectedFileId: {
				type: String,
				default: null
			}
		},
		methods: {
			selectFile(file) {
				this.$emit("selectFile", file);
			},
			openFile(file) {
				this.$emit("openFile", file);
			},
			formatDate(dateString) {
				try {
					const date = new Date(dateString);
					return date.toLocaleString();
				} catch (e) {
					return dateString;
				}
			},
			getIcon(type) {
				switch (type) {
					case "api": return "code";
					case "doc": return "file-text";
					case "code": return "code";
					default: return "folder";
				}
			},
			getIconColor(type) {
				switch (type) {
					case "api": return "api-manager__icon api-manager__icon--api";
					case "doc": return "api-manager__icon api-manager__icon--doc";
					case "code": return "api-manager__icon api-manager__icon--code";
					default: return "api-manager__icon api-manager__icon--folder";
				}
			}
		}
	};
}
</script>

<template>
	<div class="api-manager__file-grid">
		<div v-if="files.length === 0" class="api-manager__empty-state">
			<xIcon icon="folder" :size="48" class="api-manager__empty-icon" />
			<p>This folder is empty</p>
		</div>
		<div v-else class="api-manager__file-grid-inner">
			<div 
				v-for="file in files" 
				:key="file.id"
				@click.stop="selectFile(file)" 
				@dblclick.stop="openFile(file)"
				class="api-manager__file-card" 
				:class="{ 'api-manager__file-card--selected': selectedFileId === file.id }">
				<div class="api-manager__file-card-icon">
					<xIcon :icon="getIcon(file.type)" :size="32" :class="getIconColor(file.type)" />
				</div>
				<div class="api-manager__file-card-content">
					<h4 class="api-manager__file-card-name">{{ file.name }}</h4>
					<p class="api-manager__file-card-meta">{{ file.type.replace("_", " ") }}</p>
					<p class="api-manager__file-card-date">{{ formatDate(file.updatedAt) }}</p>
				</div>
			</div>
		</div>
	</div>
</template>