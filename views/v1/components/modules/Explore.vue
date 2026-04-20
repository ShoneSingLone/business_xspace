<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
	const mockFileSystem = {
	id: "root",
	name: "Home",
	type: "folder",
	updatedAt: "2026-03-31T10:00:00Z",
	path: "/Home",
	children: [
		{
			id: "docs",
			name: "Documents",
			type: "folder",
			updatedAt: "2026-03-30T14:20:00Z",
			path: "/Home/Documents",
			children: [
				{
					id: "report",
					name: "Q1_Report.pdf",
					type: "document",
					size: 1024 * 1024 * 2.5,
					updatedAt: "2026-03-29T09:15:00Z",
					path: "/Home/Documents/Q1_Report.pdf",
					content: "Quarterly report for Q1 2026. Revenue increased by 15%."
				},
				{
					id: "notes",
					name: "Meeting_Notes.txt",
					type: "document",
					size: 1024 * 15,
					updatedAt: "2026-03-28T16:45:00Z",
					path: "/Home/Documents/Meeting_Notes.txt",
					content: "Action items:\n1. Follow up with marketing.\n2. Review budget."
				}
			]
		},
		{
			id: "pics",
			name: "Pictures",
			type: "folder",
			updatedAt: "2026-03-25T11:10:00Z",
			path: "/Home/Pictures",
			children: [
				{
					id: "vacation",
					name: "Vacation.jpg",
					type: "image",
					size: 1024 * 1024 * 4.2,
					updatedAt: "2026-03-20T08:30:00Z",
					path: "/Home/Pictures/Vacation.jpg",
					url: "https://picsum.photos/seed/vacation/800/600"
				},
				{
					id: "profile",
					name: "Profile.png",
					type: "image",
					size: 1024 * 1024 * 1.1,
					updatedAt: "2026-03-22T14:00:00Z",
					path: "/Home/Pictures/Profile.png",
					url: "https://picsum.photos/seed/profile/400/400"
				}
			]
		},
		{
			id: "projects",
			name: "Projects",
			type: "folder",
			updatedAt: "2026-03-31T09:00:00Z",
			path: "/Home/Projects",
			children: [
				{
					id: "app",
					name: "App.vue",
					type: "code",
					size: 1024 * 3,
					updatedAt: "2026-03-31T09:00:00Z",
					path: "/Home/Projects/App.vue",
					content: "<template>\n  <div>Hello World</div>\n</template>"
				},
				{
					id: "styles",
					name: "styles.css",
					type: "code",
					size: 1024 * 1.5,
					updatedAt: "2026-03-30T10:00:00Z",
					path: "/Home/Projects/styles.css",
					content: "body {\n  margin: 0;\n  padding: 0;\n  font-family: sans-serif;\n}"
				}
			]
		}
	]
};

	return {
		// system: v1 Desktop Workspace 局部状态（来自 ViewXspace）
		inject: ['system'],
		data() {
			return {
				mockFileSystem: mockFileSystem,
				// State
				tabs: [mockFileSystem],
				activeTabId: mockFileSystem.id,
				selectedFile: null,
				searchQuery: "",
				showPreview: true,
				expandedFolders: new Set([mockFileSystem.id]),
				// Sort State
				sortField: "name",
				sortDirection: "asc"
			};
		},
		computed: {
			activeFolder() {
				return this.tabs.find(t => t.id === this.activeTabId) || this.mockFileSystem;
			},
			filteredAndSortedFiles() {
				let files = this.activeFolder.children || [];

				if (this.searchQuery) {
					const lowerQuery = this.searchQuery.toLowerCase();
					files = files.filter(f => f.name.toLowerCase().includes(lowerQuery));
				}

				return [...files].sort((a, b) => {
					let comparison = 0;

					if ((this.sortField === "name" || this.sortField === "type") && a.type !== b.type) {
						if (a.type === "folder") return -1;
						if (b.type === "folder") return 1;
					}

					switch (this.sortField) {
						case "name":
							comparison = a.name.localeCompare(b.name);
							break;
						case "updatedAt":
							comparison =
								new Date(a.updatedAt).getTime() - new Date(b.updatedAt).getTime();
							break;
						case "size":
							comparison = (a.size || 0) - (b.size || 0);
							break;
						case "type":
							comparison = a.type.localeCompare(b.type);
							break;
					}

					return this.sortDirection === "asc" ? comparison : -comparison;
				});
			},
			canNavigateUp() {
				return this.activeFolder.id !== this.mockFileSystem.id;
			}
		},
		methods: {
			// Utils
			formatBytes(bytes, decimals = 2) {
				if (bytes === undefined || bytes === null) return "--";
				if (!+bytes) return "0 Bytes";
				const k = 1024;
				const dm = decimals < 0 ? 0 : decimals;
				const sizes = ["Bytes", "KB", "MB", "GB", "TB", "PB", "EB", "ZB", "YB"];
				const i = Math.floor(Math.log(bytes) / Math.log(k));
				return `${parseFloat((bytes / Math.pow(k, i)).toFixed(dm))} ${sizes[i]}`;
			},
			formatDate(dateString) {
				try {
					const date = new Date(dateString);
					return date.toLocaleString();
				} catch (e) {
					return dateString;
				}
			},
			// Methods
			handleOpenFolder(folder) {
				if (folder.type !== "folder") return;

				const existingTab = this.tabs.find(t => t.id === folder.id);
				if (existingTab) {
					this.activeTabId = folder.id;
				} else {
					this.tabs.push(folder);
					this.activeTabId = folder.id;
				}
				this.selectedFile = null;
				this.searchQuery = "";
				this.expandedFolders.add(folder.id);
			},
			handleCloseTab(tabId, e) {
				e.stopPropagation();
				this.tabs = this.tabs.filter(t => t.id !== tabId);
				if (this.tabs.length === 0) {
					this.tabs = [this.mockFileSystem];
					this.activeTabId = this.mockFileSystem.id;
				} else if (this.activeTabId === tabId) {
					this.activeTabId = this.tabs[this.tabs.length - 1].id;
				}
			},
			findParentFolder(root, targetId) {
				if (!root.children) return null;
				for (const child of root.children) {
					if (child.id === targetId) return root;
					const found = this.findParentFolder(child, targetId);
					if (found) return found;
				}
				return null;
			},
			handleNavigateUp() {
				if (this.activeFolder.id === this.mockFileSystem.id) return;
				const parent = this.findParentFolder(this.mockFileSystem, this.activeFolder.id);
				if (parent) {
					this.handleOpenFolder(parent);
				}
			},
			handleSort(field) {
				if (this.sortField === field) {
					this.sortDirection = this.sortDirection === "asc" ? "desc" : "asc";
				} else {
					this.sortField = field;
					this.sortDirection = "asc";
				}
			},
			toggleFolder(folderId, e) {
				e.stopPropagation();
				if (this.expandedFolders.has(folderId)) {
					this.expandedFolders.delete(folderId);
				} else {
					this.expandedFolders.add(folderId);
				}
			},
			getIcon(type) {
				switch (type) {
					case "folder":
						return "folder";
					case "image":
						return "image";
					case "document":
						return "file-text";
					case "code":
						return "code";
					case "video":
						return "film";
					default:
						return "file";
				}
			},
			getIconColor(type) {
				switch (type) {
					case "folder":
						return "explore__icon explore__icon--folder";
					case "image":
						return "explore__icon explore__icon--image";
					case "document":
						return "explore__icon explore__icon--document";
					case "code":
						return "explore__icon explore__icon--code";
					case "video":
						return "explore__icon explore__icon--video";
					default:
						return "explore__icon explore__icon--default";
				}
			}
		}
	};
};
</script>

<template>
	<div class="explore">
		<!-- Top Bar -->
		<div class="explore__topbar">
			<!-- Tabs Row -->
			<div class="explore__tabs">
				<div
					v-for="tab in tabs"
					:key="tab.id"
					@click="activeTabId = tab.id"
					class="explore__tab"
					:class="{ 'explore__tab--active': activeTabId === tab.id }">
					<span class="explore__tab-title">{{ tab.name }}</span>
					<button
						@click="e => handleCloseTab(tab.id, e)"
						class="explore__tab-close"
						:class="{ 'explore__tab-close--visible': activeTabId === tab.id }">
						<xIcon icon="close" :size="14" />
					</button>
				</div>
			</div>

			<!-- Toolbar Row -->
			<div class="explore__toolbar">
				<div class="explore__nav">
					<button class="explore__toolbar-btn explore__toolbar-btn--disabled" type="button">
						<xIcon icon="left" :size="20" />
					</button>
					<button class="explore__toolbar-btn explore__toolbar-btn--disabled" type="button">
						<xIcon icon="right" :size="20" />
					</button>
					<button
						@click="handleNavigateUp"
						:disabled="!canNavigateUp"
						class="explore__toolbar-btn"
						:class="{ 'explore__toolbar-btn--disabled': !canNavigateUp }"
						title="Up">
						<xIcon icon="top" :size="20" />
					</button>
				</div>

				<div class="explore__search">
					<div class="explore__search-icon" aria-hidden="true">
						<xIcon icon="search" :size="16" />
					</div>
					<input
						type="text"
						placeholder="Search files..."
						v-model="searchQuery"
						class="explore__search-input" />
				</div>

				<div class="explore__toolbar-right">
					<button
						@click="showPreview = !showPreview"
						class="explore__toggle-btn"
						:class="{ 'explore__toggle-btn--active': showPreview }"
						title="Toggle Preview Pane">
						<xIcon icon="view" :size="20" />
					</button>
				</div>
			</div>
		</div>

		<div class="explore__body">
			<!-- Sidebar (Tree) -->
			<div
				class="explore__sidebar">
				<div class="explore__sidebar-inner">
					<h2
						class="explore__sidebar-title">
						Quick Access
					</h2>

					<!-- Recursive Tree Component inline simulation -->
					<div class="explore__tree">
						<div
							class="explore__tree-item"
							:class="{ 'explore__tree-item--active': activeFolder.id === mockFileSystem.id }"
							style="padding-left: 8px"
							@click="handleOpenFolder(mockFileSystem)">
							<span
								class="explore__tree-toggle"
								@click="e => toggleFolder(mockFileSystem.id, e)">
								<xIcon
									icon="_CaretDownOutlined"
									v-if="expandedFolders.has(mockFileSystem.id)"
									:size="14" />
								<xIcon
									icon="_CaretRightOutlined"
									v-else
									:size="14" />
							</span>
							<span class="explore__tree-icon" aria-hidden="true">
								<xIcon
									icon="folder-opened"
									v-if="expandedFolders.has(mockFileSystem.id)"
									:size="16" />
								<xIcon
									icon="folder"
									v-else
									:size="16" />
							</span>
							<span class="explore__tree-label">{{ mockFileSystem.name }}</span>
						</div>

						<div v-if="expandedFolders.has(mockFileSystem.id)">
							<template v-for="child in mockFileSystem.children" :key="child.id">
								<div v-if="child.type === 'folder'">
									<div
										class="explore__tree-item"
										:class="{ 'explore__tree-item--active': activeFolder.id === child.id }"
										style="padding-left: 20px"
										@click="handleOpenFolder(child)">
										<span
											class="explore__tree-toggle"
											@click="e => toggleFolder(child.id, e)">
											<template
												v-if="
													child.children?.some(c => c.type === 'folder')
												">
												<xIcon
													icon="_CaretDownOutlined"
													v-if="expandedFolders.has(child.id)"
													:size="14" />
												<xIcon
													icon="_CaretRightOutlined"
													v-else
													:size="14" />
											</template>
										</span>
										<span class="explore__tree-icon" aria-hidden="true">
											<xIcon
												icon="folder-opened"
												v-if="expandedFolders.has(child.id)"
												:size="16" />
											<xIcon
												icon="folder"
												v-else
												:size="16" />
										</span>
										<span class="explore__tree-label">{{ child.name }}</span>
									</div>

									<div v-if="expandedFolders.has(child.id) && child.children">
										<template
											v-for="subchild in child.children"
											:key="subchild.id">
											<div
												v-if="subchild.type === 'folder'"
												>
												<div
													class="explore__tree-item"
													:class="{ 'explore__tree-item--active': activeFolder.id === subchild.id }"
													style="padding-left: 32px"
													@click="handleOpenFolder(subchild)">
													<span
														class="explore__tree-toggle explore__tree-toggle--spacer"
														aria-hidden="true"></span>
													<span class="explore__tree-icon" aria-hidden="true">
														<xIcon icon="folder" :size="16" />
													</span>
													<span class="explore__tree-label">{{ subchild.name }}</span>
												</div>
											</div>
										</template>
									</div>
								</div>
							</template>
						</div>
					</div>
				</div>
			</div>

			<!-- Main Area -->
			<div class="explore__main">
				<!-- Breadcrumb -->
				<div class="explore__breadcrumb">
					<template
						v-for="(part, index) in activeFolder.path.split('/').filter(Boolean)"
						:key="index">
						<span
							class="explore__breadcrumb-part"
							:class="{ 'explore__breadcrumb-part--current': index === activeFolder.path.split('/').filter(Boolean).length - 1 }">
							{{ part }}
						</span>
						<span
							v-if="index < activeFolder.path.split('/').filter(Boolean).length - 1"
							class="explore__breadcrumb-sep"
							>/</span
						>
					</template>
				</div>

				<!-- Table Header -->
				<div class="explore__table-header">
					<div
						class="explore__table-header-cell explore__table-header-cell--name"
						@click="handleSort('name')">
						Name
						<xIcon 
									type="arrow-up"
									v-if="sortField === 'name' && sortDirection === 'asc'"
									:size="12"
									class="explore__sort-icon" />
								<xIcon 
									type="arrow-down"
									v-if="sortField === 'name' && sortDirection === 'desc'"
									:size="12"
									class="explore__sort-icon" />
					</div>
					<div
						class="explore__table-header-cell explore__table-header-cell--date"
						@click="handleSort('updatedAt')">
						Date Modified
						<xIcon 
									type="arrow-up"
									v-if="sortField === 'updatedAt' && sortDirection === 'asc'"
									:size="12"
									class="explore__sort-icon" />
								<xIcon 
									type="arrow-down"
									v-if="sortField === 'updatedAt' && sortDirection === 'desc'"
									:size="12"
									class="explore__sort-icon" />
					</div>
					<div
						class="explore__table-header-cell explore__table-header-cell--type"
						@click="handleSort('type')">
						Type
						<xIcon 
									type="arrow-up"
									v-if="sortField === 'type' && sortDirection === 'asc'"
									:size="12"
									class="explore__sort-icon" />
								<xIcon 
									type="arrow-down"
									v-if="sortField === 'type' && sortDirection === 'desc'"
									:size="12"
									class="explore__sort-icon" />
					</div>
					<div
						class="explore__table-header-cell explore__table-header-cell--size"
						@click="handleSort('size')">
						Size
						<xIcon 
									type="arrow-up"
									v-if="sortField === 'size' && sortDirection === 'asc'"
									:size="12"
									class="explore__sort-icon" />
								<xIcon 
									type="arrow-down"
									v-if="sortField === 'size' && sortDirection === 'desc'"
									:size="12"
									class="explore__sort-icon" />
					</div>
				</div>

				<!-- File List -->
				<div class="explore__file-list" @click="selectedFile = null">
					<div
						v-if="filteredAndSortedFiles.length === 0"
							class="explore__empty-state">
						<xIcon icon="folder" :size="48" class="explore__empty-icon" />
						<p>This folder is empty</p>
					</div>
					<div v-else class="explore__file-list-inner">
						<div
							v-for="file in filteredAndSortedFiles"
							:key="file.id"
							@click.stop="selectedFile = file"
							@dblclick.stop="file.type === 'folder' ? handleOpenFolder(file) : null"
							class="explore__file-row"
							:class="{ 'explore__file-row--selected': selectedFile?.id === file.id }">
							<div class="explore__file-cell explore__file-cell--name">
								<xIcon 
												:type="getIcon(file.type)"
												:size="20"
												:class="getIconColor(file.type)" />
								<span class="explore__file-name">{{
									file.name
								}}</span>
							</div>
							<div class="explore__file-cell explore__file-cell--date">
								{{ formatDate(file.updatedAt) }}
							</div>
							<div class="explore__file-cell explore__file-cell--type">
								{{ file.type }}
							</div>
							<div class="explore__file-cell explore__file-cell--size">
								{{ file.type === "folder" ? "--" : formatBytes(file.size) }}
							</div>
						</div>
					</div>
				</div>
			</div>

			<!-- Preview Pane -->
			<div
				v-if="showPreview"
				class="explore__preview">
				<div
					v-if="!selectedFile"
					class="explore__preview-empty">
						<xIcon icon="info-filled" :size="48" class="explore__preview-empty-icon" />
					<p>Select a file to preview</p>
				</div>
				<div v-else class="explore__preview-inner">
					<!-- Preview Content -->
					<div
						v-if="selectedFile.type === 'image'"
						class="explore__preview-media explore__preview-media--image">
						<img
							v-if="selectedFile.url"
							:src="selectedFile.url"
							:alt="selectedFile.name"
							class="explore__preview-img" />
						<xIcon v-else icon="picture" :size="64" class="explore__preview-placeholder" />
					</div>
					<div
						v-else-if="selectedFile.type === 'document' || selectedFile.type === 'code'"
						class="explore__preview-code">
						<pre
							class="explore__preview-code-pre"
							>{{ selectedFile.content || "No content available." }}</pre
						>
					</div>
					<div
						v-else-if="selectedFile.type === 'folder'"
						class="explore__preview-folder">
						<xIcon icon="folder" :size="80" class="explore__icon explore__icon--folder" />
					</div>
					<div
						v-else
						class="explore__preview-generic">
						<xIcon icon="files" :size="80" class="explore__preview-generic-icon" />
					</div>

					<!-- Metadata -->
					<h3 class="explore__preview-title">
						{{ selectedFile.name }}
					</h3>
					<p class="explore__preview-subtitle">{{
						selectedFile.type
					}}</p>

					<div class="explore__preview-meta">
						<div class="explore__preview-section">
							<h4
								class="explore__preview-section-title">
								Information
							</h4>
							<dl class="explore__preview-dl">
								<div class="explore__preview-dl-row">
									<dt class="explore__preview-dt">Size</dt>
									<dd class="explore__preview-dd">
										{{
											selectedFile.type === "folder"
												? "--"
												: formatBytes(selectedFile.size)
										}}
									</dd>
								</div>
								<div class="explore__preview-dl-row">
									<dt class="explore__preview-dt">Modified</dt>
									<dd class="explore__preview-dd">
										{{ formatDate(selectedFile.updatedAt) }}
									</dd>
								</div>
								<div class="explore__preview-dl-row">
									<dt class="explore__preview-dt">Path</dt>
									<dd class="explore__preview-dd explore__preview-dd--break">
										{{ selectedFile.path }}
									</dd>
								</div>
							</dl>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style lang="less">
.explore {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: var(--color-surface);
  color: var(--color-on-surface);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", "PingFang SC",
    "Hiragino Sans GB", "Microsoft Yahei", sans-serif;

  &__topbar {
    display: flex;
    flex-direction: column;
    border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
    background: var(--color-surface-container-lowest);
  }

  &__tabs {
    display: flex;
    align-items: center;
    gap: 4px;
    padding: 8px 8px 0;
    overflow-x: auto;
    background: var(--color-surface-container);
    scrollbar-width: none;

    &::-webkit-scrollbar {
      display: none;
    }
  }

  &__tab {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 6px 12px;
    min-width: 120px;
    max-width: 200px;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    cursor: pointer;
    user-select: none;
    border: 1px solid transparent;
    border-bottom: 0;
    background: transparent;
    color: var(--color-on-surface-variant);
    transition: background-color 0.15s ease, color 0.15s ease, border-color 0.15s ease;

    &:hover {
      background: var(--color-surface-container-high, var(--color-surface-container));
    }
  }

  &__tab--active {
    background: var(--color-surface-container-lowest);
    border-color: color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
    color: var(--color-primary);
  }

  &__tab-title {
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    font-size: 14px;
    font-weight: 600;
  }

  &__tab-close {
    width: 22px;
    height: 22px;
    border-radius: 8px;
    border: 0;
    background: transparent;
    color: var(--color-on-surface-variant);
    display: inline-flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.15s ease, background-color 0.15s ease, color 0.15s ease;
    cursor: pointer;
  }

  &__tab:hover &__tab-close {
    opacity: 1;
  }

  &__tab-close--visible {
    opacity: 1;
  }

  &__tab-close:hover {
    background: var(--color-surface-variant);
    color: var(--color-on-surface);
  }

  &__toolbar {
    display: flex;
    align-items: center;
    gap: 16px;
    padding: 8px 16px;
  }

  &__nav {
    display: flex;
    align-items: center;
    gap: 4px;
  }

  &__search {
    flex: 1;
    max-width: 42rem;
    position: relative;
  }

  &__search-icon {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 12px;
    display: flex;
    align-items: center;
    pointer-events: none;
    color: var(--color-on-surface-variant);
  }

  &__search-input {
    width: 100%;
    padding: 6px 12px 6px 40px;
  }

  &__toolbar-right {
    margin-left: auto;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  &__body {
    flex: 1;
    display: flex;
    overflow: hidden;
    background: var(--color-surface-container-lowest);
  }

  &__sidebar {
    width: 256px;
    flex-shrink: 0;
    border-right: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
    background: var(--color-surface-container);
    overflow-y: auto;
  }

  &__sidebar-inner {
    padding: 8px;
  }

  &__sidebar-title {
    padding: 0 8px;
    margin: 0 0 8px;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--color-on-surface-variant);
  }

  &__tree {
    user-select: none;
  }

  &__tree-item {
    display: flex;
    align-items: center;
    padding: 4px 8px;
    border-radius: 10px;
    cursor: pointer;
    font-size: 14px;
    color: var(--color-on-surface);
    transition: background-color 0.15s ease, color 0.15s ease;

    &:hover {
      background: var(--color-surface-variant);
    }
  }

  &__tree-item--active {
    background: var(--color-primary-container);
    color: var(--color-on-primary-container);

    &:hover {
      background: var(--color-primary-container);
    }
  }

  &__tree-toggle {
    width: 16px;
    height: 16px;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    margin-right: 4px;
    color: var(--color-on-surface-variant);
  }

  &__tree-toggle--spacer {
    visibility: hidden;
    pointer-events: none;
  }

  &__tree-icon {
    margin-right: 8px;
    color: var(--color-primary);
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }

  &__tree-label {
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  &__main {
    flex: 1;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    background: var(--color-surface-container-lowest);
  }

  &__breadcrumb {
    display: flex;
    align-items: center;
    padding: 8px 16px;
    border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
    background: var(--color-surface-container);
    font-size: 14px;
    color: var(--color-on-surface-variant);
  }

  &__breadcrumb-part {
    cursor: pointer;
    transition: color 0.15s ease;

    &:hover {
      color: var(--color-primary);
    }
  }

  &__breadcrumb-part--current {
    cursor: default;
    font-weight: 600;
    color: var(--color-on-surface);

    &:hover {
      color: var(--color-on-surface);
    }
  }

  &__breadcrumb-sep {
    margin: 0 8px;
    color: color-mix(in srgb, var(--color-on-surface-variant) 55%, transparent);
  }

  &__table-header {
    display: grid;
    grid-template-columns: 6fr 3fr 2fr 1fr;
    gap: 16px;
    padding: 8px 16px;
    border-bottom: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
    background: var(--color-surface-container);
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
    color: var(--color-on-surface-variant);
    user-select: none;
  }

  &__table-header-cell {
    display: inline-flex;
    align-items: center;
    cursor: pointer;
    transition: color 0.15s ease;

    &:hover {
      color: var(--color-on-surface);
    }
  }

  &__table-header-cell--size {
    justify-content: flex-end;
  }

  &__sort-icon {
    margin-left: 4px;
  }

  &__file-list {
    flex: 1;
    overflow-y: auto;
  }

  &__file-list-inner {
    padding: 4px 0;
  }

  &__file-row {
    display: grid;
    grid-template-columns: 6fr 3fr 2fr 1fr;
    gap: 16px;
    padding: 8px 16px;
    font-size: 14px;
    align-items: center;
    cursor: pointer;
    user-select: none;
    border-bottom: 1px solid transparent;
    transition: background-color 0.15s ease, border-color 0.15s ease;

    &:hover {
      background: color-mix(in srgb, var(--color-surface-variant) 50%, transparent);
    }
  }

  &__file-row--selected {
    background: color-mix(in srgb, var(--color-primary-container) 50%, transparent);
    border-color: color-mix(in srgb, var(--color-primary) 12%, transparent);
  }

  &__file-cell {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    color: var(--color-on-surface-variant);
  }

  &__file-cell--name {
    display: flex;
    align-items: center;
    gap: 12px;
    color: var(--color-on-surface);
  }

  &__file-cell--size {
    text-align: right;
  }

  &__file-name {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    font-weight: 600;
  }

  &__empty-state {
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 8px;
    color: var(--color-on-surface-variant);
  }

  &__empty-icon {
    margin-bottom: 8px;
    color: var(--color-outline-variant);
  }

  &__preview {
    width: 320px;
    flex-shrink: 0;
    border-left: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
    background: var(--color-surface-container);
    display: flex;
    flex-direction: column;
    overflow-y: auto;
  }

  &__preview-empty {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 24px;
    text-align: center;
    color: var(--color-on-surface-variant);
    gap: 12px;
  }

  &__preview-empty-icon {
    color: var(--color-outline-variant);
  }

  &__preview-inner {
    padding: 24px;
  }

  &__preview-media {
    width: 100%;
    height: 180px;
    border-radius: 14px;
    overflow: hidden;
    border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
    background: var(--color-surface-variant);
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 24px;
  }

  &__preview-img {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }

  &__preview-placeholder {
    color: var(--color-outline-variant);
  }

  &__preview-code {
    padding: 12px;
    border-radius: 18px;
    border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
    background: var(--color-surface-container-lowest);
    margin-bottom: 24px;
  }

  &__preview-code-pre {
    margin: 0;
    max-height: 16rem;
    overflow-y: auto;
    white-space: pre-wrap;
    font-size: 12px;
    color: var(--color-on-surface);
    font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;
  }

  &__preview-folder,
  &__preview-generic {
    width: 160px;
    height: 160px;
    margin: 0 auto 24px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 18px;
    border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
  }

  &__preview-folder {
    background: color-mix(in srgb, var(--color-primary-container) 42%, var(--color-surface-container-lowest));
    border-color: color-mix(in srgb, var(--color-primary) 22%, transparent);
  }

  &__preview-generic {
    background: var(--color-surface-variant);
  }

  &__preview-generic-icon {
    color: var(--color-on-surface-variant);
  }

  &__preview-title {
    margin: 0 0 4px;
    font-size: 18px;
    font-weight: 700;
    color: var(--color-on-surface);
    word-break: break-word;
  }

  &__preview-subtitle {
    margin: 0 0 24px;
    font-size: 14px;
    font-weight: 600;
    color: var(--color-on-surface-variant);
    text-transform: capitalize;
  }

  &__preview-meta {
    display: flex;
    flex-direction: column;
    gap: 16px;
  }

  &__preview-section {
    padding-top: 16px;
    border-top: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
  }

  &__preview-section-title {
    margin: 0 0 12px;
    font-size: 12px;
    font-weight: 800;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--color-on-surface-variant);
  }

  &__preview-dl {
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 10px;
    font-size: 14px;
  }

  &__preview-dl-row {
    display: grid;
    grid-template-columns: 1fr 2fr;
    gap: 8px;
    align-items: start;
  }

  &__preview-dt {
    color: var(--color-on-surface-variant);
  }

  &__preview-dd {
    margin: 0;
    font-weight: 600;
    color: var(--color-on-surface);
  }

  &__preview-dd--break {
    word-break: break-all;
  }

  &__toolbar-btn,
  &__toggle-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-width: 32px;
    min-height: 32px;
    border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
    border-radius: 10px;
    background: var(--color-surface-container-lowest);
    color: var(--color-on-surface-variant);
    cursor: pointer;
    transition: background-color 0.2s ease, border-color 0.2s ease, color 0.2s ease;

    &:hover {
      background: var(--color-surface-container);
    }
  }

  &__toolbar-btn--disabled {
    opacity: 0.45;
    cursor: not-allowed;
    pointer-events: none;
  }

  &__toggle-btn--active {
    background: var(--color-primary-container);
    border-color: color-mix(in srgb, var(--color-primary) 22%, transparent);
    color: var(--color-on-primary-container);
  }

  &__search-input {
    border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
    border-radius: 10px;
    background: var(--color-surface-container-lowest);
    color: var(--color-on-surface);
    outline: none;
    transition: border-color 0.2s ease, box-shadow 0.2s ease;

    &:focus {
      border-color: var(--el-color-primary);
      box-shadow: 0 0 0 2px color-mix(in srgb, var(--el-color-primary) 16%, transparent);
    }
  }

  &__empty-state {
    color: var(--color-on-surface-variant);
  }

  &__preview-code,
  &__preview-folder,
  &__preview-generic {
    border-radius: 18px;
    border: 1px solid color-mix(in srgb, var(--color-outline-variant) 55%, transparent);
    background: var(--color-surface-container-lowest);
  }

  &__preview-folder {
    background: color-mix(in srgb, var(--color-primary-container) 42%, var(--color-surface-container-lowest));
    border-color: color-mix(in srgb, var(--color-primary) 22%, transparent);
  }

  &__preview-generic {
    background: var(--color-surface-variant);
  }

  &__icon--folder,
  &__icon--document {
    color: var(--el-color-primary);
  }

  &__icon--image {
    color: var(--el-color-info-dark-2);
  }

  &__icon--code {
    color: var(--el-color-warning-dark-2);
  }

  &__icon--video {
    color: var(--el-color-danger);
  }

  &__icon--default {
    color: var(--color-on-surface-variant);
  }
}
</style>
