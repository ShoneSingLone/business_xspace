<style lang="less">
:root {
	--color-white: white;
}

:root {
  --color-primary: #0061a4;
  --color-on-primary: #ffffff;
  --color-primary-container: #d1e4ff;
  --color-on-primary-container: #001d36;
  --color-secondary: #535f70;
  --color-on-secondary: #ffffff;
  --color-secondary-container: #d7e3f7;
  --color-on-secondary-container: #101c2b;
  --color-tertiary: #6b5778;
  --color-on-tertiary: #ffffff;
  --color-tertiary-container: #f2daff;
  --color-on-tertiary-container: #251431;
  --color-error: #ba1a1a;
  --color-on-error: #ffffff;
  --color-error-container: #ffdad6;
  --color-on-error-container: #410002;
  --color-background: #fdfcff;
  --color-on-background: #1a1c1e;
  --color-surface: #fdfcff;
  --color-on-surface: #1a1c1e;
  --color-surface-variant: #dfe2eb;
  --color-on-surface-variant: #43474e;
  --color-outline: #73777f;
  --color-outline-variant: #c3c7cf;
  --color-shadow: #000000;
  --color-scrim: #000000;
  --color-inverse-surface: #2f3033;
  --color-inverse-on-surface: #f1f0f4;
  --color-inverse-primary: #9ecaef;
  --color-surface-dim: #ded8e1;
  --color-surface-bright: #fdfcff;
  --color-surface-container-lowest: #ffffff;
  --color-surface-container-low: #f7f2fa;
  --color-surface-container: #f3edf7;
  --color-surface-container-high: #ece6f0;
  --color-surface-container-highest: #e6e0e9;

  --font-sans: "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-mono: "JetBrains Mono", ui-monospace, SFMono-Regular, monospace;
}

/* 重置基础样式 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: var(--font-sans);
  background-color: var(--color-background);
  color: var(--color-on-background);
  line-height: 1.5;
}

/* 工具类 */
.no-scrollbar::-webkit-scrollbar {
  display: none;
}
.no-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
}

.elevation-1 {
  box-shadow: 0px 1px 2px 0px rgba(0, 0, 0, 0.3), 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
}
.elevation-2 {
  box-shadow: 0px 1px 2px 0px rgba(0, 0, 0, 0.3), 0px 2px 6px 2px rgba(0, 0, 0, 0.15);
}
.elevation-3 {
  box-shadow: 0px 1px 3px 0px rgba(0, 0, 0, 0.3), 0px 4px 8px 3px rgba(0, 0, 0, 0.15);
}
.elevation-4 {
  box-shadow: 0px 2px 3px 0px rgba(0, 0, 0, 0.3), 0px 6px 10px 4px rgba(0, 0, 0, 0.15);
}
.elevation-5 {
  box-shadow: 0px 4px 4px 0px rgba(0, 0, 0, 0.3), 0px 8px 12px 6px rgba(0, 0, 0, 0.15);
}

.rounded-m3-small { border-radius: 8px; }
.rounded-m3-medium { border-radius: 12px; }
.rounded-m3-large { border-radius: 16px; }
.rounded-m3-extra-large { border-radius: 28px; }

/* 组件样式 */
.m3-button-primary {
  padding: 0.625rem 1.5rem;
  background-color: var(--color-primary);
  color: var(--color-on-primary);
  border-radius: 9999px;
  font-weight: 500;
  font-size: 0.875rem;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease;
}
.m3-button-primary:hover {
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
}
.m3-button-primary:active {
  background-color: rgba(0, 97, 164, 0.9);
}

.m3-card {
  background-color: var(--color-surface-container-low);
  border-radius: 1rem;
  border: 1px solid rgba(195, 199, 207, 0.3);
}


.x-sider_wrapper {
	height: 100%;
	width: 300px;
	overflow: hidden;
	flex-flow: column nowrap;
	display: flex;
	transition: 0.1s ease;

	.resize_bar {
		&:hover {
			cursor: ew-resize;
		}
		position: absolute;
		top: 40px;
		bottom: 0;
		right: 0;
		width: var(--ui-one);
	}
}
.x-sider_wrapper_tree {
	height: 100px;
	flex: 1;
	padding: var(--ui-one);

	[role="treeitem"] {
		&.el-tree-node.treeitem-is-active {
			background-color: var(--el-color-primary-light-7) !important;
			&:hover {
				background-color: var(--el-color-primary-light-7) !important;
			}
		}
	}

	.el-tree-treenode {
		width: 100%;
		font-size: 14px;

		span {
			transition: 0.3s all ease-in-out;
		}

		span[title] {
			flex: 1;
		}

		.el-tree-node-content-wrapper {
			padding: 0;

			[data-interface-all-menu] {
				transform: translateX(-32px);
			}
		}
	}
}

.asideTreeItem {
	display: flex;
	flex: 1;
	width: 1px;
	// padding: var(--ui-one);
	align-items: center;
	&:hover {
		.node-icon-add {
			display: inline-block;
		}
	}

	.node-name {
		width: 1px;
		flex: 1;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}
	&.is-current {
		.node-icon-current {
			display: inline-block;
		}
	}
	.node-icon-current,
	.node-icon-add {
		display: none;
	}
}
</style>

<script lang="ts">
export default async function () {
	return {};
}
</script>
