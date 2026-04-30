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
			},
			sortField: {
				type: String,
				default: "name"
			},
			sortDirection: {
				type: String,
				default: "asc"
			}
		},
		methods: {
			selectFile(file) {
				this.$emit("select-file", file);
			},
			openFile(file) {
				this.$emit("open-file", file);
			},
			handleSort(field) {
				this.$emit("sort", field);
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
			},
			getFileRowClass(file) {
				return {
					'api-manager__file-row': true,
					'api-manager__file-row--selected': this.selectedFileId === file.id
				};
			}
		}
	};
}
</script>

<template>
	<div>
		<div class="api-manager__table-header">
			<div class="api-manager__table-header-cell api-manager__table-header-cell--name" @click="handleSort('name')">
				Name
				<xIcon icon="arrow-up" v-if="sortField === 'name' && sortDirection === 'asc'" :size="12" class="api-manager__sort-icon" />
				<xIcon icon="arrow-down" v-if="sortField === 'name' && sortDirection === 'desc'" :size="12" class="api-manager__sort-icon" />
			</div>
			<div class="api-manager__table-header-cell api-manager__table-header-cell--date" @click="handleSort('updatedAt')">
				Date Modified
				<xIcon icon="arrow-up" v-if="sortField === 'updatedAt' && sortDirection === 'asc'" :size="12" class="api-manager__sort-icon" />
				<xIcon icon="arrow-down" v-if="sortField === 'updatedAt' && sortDirection === 'desc'" :size="12" class="api-manager__sort-icon" />
			</div>
			<div class="api-manager__table-header-cell api-manager__table-header-cell--type" @click="handleSort('type')">
				Type
				<xIcon icon="arrow-up" v-if="sortField === 'type' && sortDirection === 'asc'" :size="12" class="api-manager__sort-icon" />
				<xIcon icon="arrow-down" v-if="sortField === 'type' && sortDirection === 'desc'" :size="12" class="api-manager__sort-icon" />
			</div>
		</div>

		<div class="api-manager__file-list" @click="$emit('selectFile', null)">
			<div v-if="files.length === 0" class="api-manager__empty-state">
				<xIcon icon="folder" :size="48" class="api-manager__empty-icon" />
				<p>This folder is empty</p>
			</div>
			<div v-else class="api-manager__file-list-inner">
				<div 
				v-for="file in files" 
				:key="file.id"
				@click.stop="selectFile(file)" 
				@dblclick.stop="openFile(file)"
				:class="getFileRowClass(file)">
					<div class="api-manager__file-cell api-manager__file-cell--name">
						<xIcon :icon="getIcon(file.type)" :size="20" :class="getIconColor(file.type)" />
						<span class="api-manager__file-name">{{ file.name }}</span>
					</div>
					<div class="api-manager__file-cell api-manager__file-cell--date">
						{{ formatDate(file.updatedAt) }}
					</div>
					<div class="api-manager__file-cell api-manager__file-cell--type">
						{{ file.type.replace("_", " ") }}
					</div>
				</div>
			</div>
		</div>
	</div>
</template>