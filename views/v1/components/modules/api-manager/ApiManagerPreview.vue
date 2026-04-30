<script lang="ts">
export default async function () {
	return {
		props: {
			selectedFile: {
				type: Object,
				default: null
			},
			isFolderType: {
				type: Boolean,
				default: false
			}
		},
		methods: {
			openNode() {
				if (this.selectedFile) {
					this.$emit("open-node", this.selectedFile);
				}
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
	<div class="api-manager__preview">
		<div v-if="!selectedFile" class="api-manager__preview-empty">
			<xIcon icon="info-filled" :size="48" class="api-manager__preview-empty-icon" />
			<p>Select a resource to preview</p>
		</div>
		<div v-else class="api-manager__preview-inner">
			<div class="api-manager__preview-hero">
				<xIcon :icon="getIcon(selectedFile.type)" :size="60" :class="getIconColor(selectedFile.type)" />
			</div>

			<h3 class="api-manager__preview-title">{{ selectedFile.name }}</h3>
			<p class="api-manager__preview-subtitle">{{ selectedFile.type.replace("_", " ") }}</p>

			<div class="api-manager__preview-meta">
				<div class="api-manager__preview-section">
					<h4 class="api-manager__preview-section-title">Information</h4>
					<dl class="api-manager__preview-dl">
						<div class="api-manager__preview-dl-row">
							<dt class="api-manager__preview-dt">Modified</dt>
							<dd class="api-manager__preview-dd">{{ formatDate(selectedFile.updatedAt) }}</dd>
						</div>
						<div class="api-manager__preview-dl-row">
							<dt class="api-manager__preview-dt">Path</dt>
							<dd class="api-manager__preview-dd api-manager__preview-dd--break">{{ selectedFile.path }}</dd>
						</div>
					</dl>
				</div>

				<div v-if="selectedFile.content" class="api-manager__preview-section">
					<h4 class="api-manager__preview-section-title">Quick Preview</h4>
					<div class="api-manager__code-block api-manager__code-block--preview">
						<pre class="api-manager__preview-code">
							{{ typeof selectedFile.content === "string" ? selectedFile.content : JSON.stringify(selectedFile.content, null, 2) }}
						</pre>
					</div>
				</div>

				<button v-if="!isFolderType" @click="openNode" class="api-manager__action-btn api-manager__action-btn--primary api-manager__preview-action">
					Open Editor
				</button>
			</div>
		</div>
	</div>
</template>