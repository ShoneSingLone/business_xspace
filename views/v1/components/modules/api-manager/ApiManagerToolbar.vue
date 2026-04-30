<script lang="ts">
export default async function () {
	return {
		components: {
			ApiManagerSearchPopover: () => _.$importVue("@/views/v1/components/modules/api-manager/ApiManagerSearchPopover.vue")
		},
		props: {
			searchQuery: {
				type: String,
				default: ""
			},
			canNavigateUp: {
				type: Boolean,
				default: false
			},
			searchResults: {
				type: Array,
				default: () => []
			},
			isSearchLoading: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				localSearchQuery: "",
				showSearchPopover: false,
				searchInputRef: null,
				popoverPosition: { top: 0, left: 0 }
			};
		},
		methods: {
			updateSearchQuery(event) {
				const value = event.target.value;
				this.localSearchQuery = value;
				this.$emit("update:searchQuery", value);
				
				if (value.trim()) {
					this.showSearchPopover = true;
					this.updatePopoverPosition();
				} else {
					this.showSearchPopover = false;
				}
			},
			clearSearch() {
				this.localSearchQuery = "";
				this.$emit("update:searchQuery", "");
				this.showSearchPopover = false;
				if (this.searchInputRef) {
					this.searchInputRef.focus();
				}
			},
			navigateUp() {
				this.$emit("navigateUp");
			},
			getNavBtnClass() {
				return {
					'api-manager__toolbar-btn': true,
					'api-manager__toolbar-btn--disabled': !this.canNavigateUp
				};
			},
			handleSearchInputFocus() {
				if (this.localSearchQuery.trim()) {
					this.showSearchPopover = true;
					this.updatePopoverPosition();
				}
			},
			handleSearchInputBlur() {
				setTimeout(() => {
					if (!this.$el.contains(document.activeElement)) {
						this.showSearchPopover = false;
					}
				}, 200);
			},
			updatePopoverPosition() {
				this.$nextTick(() => {
					const searchEl = this.$el.querySelector('.api-manager__search');
					if (searchEl) {
						const rect = searchEl.getBoundingClientRect();
						this.$emit("update:popoverPosition", {
							top: rect.bottom + 8,
							left: rect.left
						});
					}
				});
			},
			handleSelectResult(result) {
				this.$emit("selectSearchResult", result);
				this.clearSearch();
			},
			handleClosePopover() {
				this.showSearchPopover = false;
			}
		},
		watch: {
			searchQuery(newVal) {
				this.localSearchQuery = newVal;
			}
		},
		mounted() {
			document.addEventListener('keydown', this.handleGlobalKeydown);
		},
		beforeDestroy() {
			document.removeEventListener('keydown', this.handleGlobalKeydown);
		},
		handleGlobalKeydown(event) {
			if (event.key === '/' && !event.ctrlKey && !event.metaKey) {
				const activeEl = document.activeElement;
				if (activeEl.tagName !== 'INPUT' && activeEl.tagName !== 'TEXTAREA') {
					event.preventDefault();
					this.searchInputRef?.focus();
				}
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
					:class="getNavBtnClass()"
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
					placeholder="搜索资源..." 
					:value="localSearchQuery"
					@input="updateSearchQuery"
					@focus="handleSearchInputFocus"
					@blur="handleSearchInputBlur"
					class="api-manager__search-input" 
					ref="searchInputRef"/>
				<button v-if="localSearchQuery" @click="clearSearch" class="api-manager__search-clear">
					<xIcon icon="x" :size="14" />
				</button>
			</div>
		</div>

		<ApiManagerSearchPopover
			:show="showSearchPopover"
			:search-query="localSearchQuery"
			:search-results="searchResults"
			:is-loading="isSearchLoading"
			:position="popoverPosition"
			@select="handleSelectResult"
			@close="handleClosePopover" />
	</div>
</template>