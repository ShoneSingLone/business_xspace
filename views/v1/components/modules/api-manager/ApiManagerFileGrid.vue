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
				this.$emit("select-file", file);
			},
			openFile(file) {
				this.$emit("open-file", file);
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
			<XspaceExplorerCard
				v-for="file in files"
				:key="file.id"
				:selected="selectedFileId === file.id"
				@click.stop="selectFile(file)"
				@dblclick.stop="openFile(file)">
				<template #icon>
					<xIcon :icon="getIcon(file.type)" :size="18" :class="getIconColor(file.type)" />
				</template>
				<template #title>
					{{ file.name }}
				</template>
				<template #meta>
					{{ file.type.replace("_", " ") }}
				</template>
				<template #date>
					{{ formatDate(file.updatedAt) }}
				</template>
			</XspaceExplorerCard>
		</div>
	</div>
</template>
