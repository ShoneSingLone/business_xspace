<script lang="ts">
export default async function () {
	const api = await _.$importVue("@/utils/api.vue");
	return {
		components: {
			DemoSection: () => _.$importVue("@/views/v1/components/molecules/DemoSection.vue"),
			DemoCard: () => _.$importVue("@/views/v1/components/molecules/DemoCard.vue"),
			MemberIdentity: () => _.$importVue("@/views/v1/components/molecules/MemberIdentity.vue"),
			MemberListPanel: () => _.$importVue("@/views/v1/components/organisms/MemberListPanel.vue")
		},
		data() {
			return {
				useRealData: false,
				canManage: true,
				loading: false,
				groupName: "Design System",
				members: [
					{ uid: 1001, username: "Alice", email: "alice@example.com", role: "owner" },
					{ uid: 1002, username: "Bob", email: "bob@example.com", role: "dev" },
					{ uid: 1003, username: "Charlie", email: "", role: "guest" }
				],
				colorTokens: [
					{ name: "--color-primary", value: "var(--color-primary)" },
					{ name: "--color-primary-container", value: "var(--color-primary-container)" },
					{ name: "--color-surface", value: "var(--color-surface)" },
					{ name: "--color-surface-container", value: "var(--color-surface-container)" },
					{ name: "--color-surface-container-lowest", value: "var(--color-surface-container-lowest)" },
					{ name: "--color-surface-variant", value: "var(--color-surface-variant)" },
					{ name: "--color-on-surface", value: "var(--color-on-surface)" },
					{ name: "--color-on-surface-variant", value: "var(--color-on-surface-variant)" },
					{ name: "--color-outline-variant", value: "var(--color-outline-variant)" },
					{ name: "--color-error", value: "var(--color-error)" },
					{ name: "--color-success", value: "var(--color-success)" },
					{ name: "--color-warning", value: "var(--color-warning)" }
				],
				realMembers: [],
				realLoading: false,
				realError: ""
			};
		},
		computed: {
			cptGroupIdFromQuery() {
				const q = this.$route?.query || {};
				const id = q.group_id || q.groupId;
				return id ? String(id) : "";
			},
			cptEffectiveMembers() {
				return this.useRealData ? this.realMembers : this.members;
			},
			cptEffectiveLoading() {
				return this.useRealData ? this.realLoading : this.loading;
			},
			cptEffectiveGroupName() {
				if (!this.useRealData) return this.groupName;
				return this.groupName || "Design System";
			},
			cptRealDataTip() {
				if (!this.cptGroupIdFromQuery) return i18n("未传 group_id，使用 mock 数据演示");
				return `${i18n("当前 group_id")}: ${this.cptGroupIdFromQuery}`;
			}
		},
		watch: {
			useRealData: {
				immediate: true,
				async handler(val) {
					if (val) await this.refreshRealMembers();
				}
			},
			"$route.query.group_id": {
				async handler() {
					if (this.useRealData) await this.refreshRealMembers();
				}
			}
		},
		methods: {
			toggleManage() {
				this.canManage = !this.canManage;
			},
			toggleLoading() {
				this.loading = !this.loading;
			},
			toggleDataSource() {
				this.useRealData = !this.useRealData;
			},
			resetMembers() {
				this.members = [
					{ uid: 1001, username: "Alice", email: "alice@example.com", role: "owner" },
					{ uid: 1002, username: "Bob", email: "bob@example.com", role: "dev" },
					{ uid: 1003, username: "Charlie", email: "", role: "guest" }
				];
			},
			clearMembers() {
				this.members = [];
			},
			async refreshRealMembers() {
				const groupId = this.cptGroupIdFromQuery;
				this.realError = "";
				if (!groupId) {
					this.realMembers = [];
					return;
				}
				this.realLoading = true;
				try {
					const res = await api.groupGetMemberListBy(groupId);
					if (res?.errcode !== 0) {
						this.realError = res?.errmsg || i18n("加载失败");
						this.realMembers = [];
						return;
					}
					this.realMembers = res?.data || [];
				} catch (e) {
					this.realError = i18n("加载失败");
					this.realMembers = [];
				} finally {
					this.realLoading = false;
				}
			},
			openAddMember() {
				const vm = this;
				_.$ModalManager.open({
					title: i18n("添加成员"),
					url: "@/views/v1/single/design-system/DesignSystemMemberUpsert.dialog.vue",
					parent: vm,
					windowId: "design-system-add-member",
					modalConfigs: {
						width: 520
					},
					onOk({ member }) {
						vm.members = [...vm.members, member];
					}
				});
			},
			async removeMember(member) {
				try {
					await _.$confirm(i18n("确认移除该成员？"));
				} catch (e) {
					return;
				}
				this.members = this.members.filter(m => m.uid !== member.uid);
				_.$msg(i18n("移除成功"));
			},
			async changeMemberRole({ member, role }) {
				if (member.role === role) return;
				try {
					await _.$confirm(i18n("确认修改该成员权限？"));
				} catch (e) {
					return;
				}
				member.role = role;
				this.members = [...this.members];
				_.$msg(i18n("修改成功"));
			}
		},
		template: `
      <div class="DesignSystemDev">
        <div class="DesignSystemDev__layout">
          <aside class="DesignSystemDev__nav">
            <div class="DesignSystemDev__nav-title">Design System</div>
            <a class="DesignSystemDev__nav-link" href="#atoms">{{ i18n("Atoms") }}</a>
            <a class="DesignSystemDev__nav-link" href="#molecules">{{ i18n("Molecules") }}</a>
            <a class="DesignSystemDev__nav-link" href="#organisms">{{ i18n("Organisms") }}</a>
            <a class="DesignSystemDev__nav-link" href="#apps">{{ i18n("Apps") }}</a>
          </aside>

          <main class="DesignSystemDev__content">
            <div class="DesignSystemDev__topbar">
              <div class="DesignSystemDev__topbar-left">
                <div class="DesignSystemDev__h1">{{ i18n("设计系统演示") }}</div>
                <div class="DesignSystemDev__hint">
                  {{ cptRealDataTip }}
                </div>
                <div class="DesignSystemDev__hint DesignSystemDev__hint--error" v-if="realError">
                  {{ realError }}
                </div>
              </div>
              <div class="DesignSystemDev__toolbar">
                <button class="api-manager__action-btn api-manager__action-btn--secondary" @click="toggleDataSource">
                  {{ useRealData ? i18n("切换 mock 数据") : i18n("切换真实数据") }}
                </button>
                <button class="api-manager__action-btn api-manager__action-btn--secondary" @click="toggleManage">
                  {{ canManage ? i18n("切换只读") : i18n("切换可编辑") }}
                </button>
                <button class="api-manager__action-btn api-manager__action-btn--secondary" @click="toggleLoading" v-if="!useRealData">
                  {{ loading ? i18n("关闭加载态") : i18n("开启加载态") }}
                </button>
                <button class="api-manager__action-btn api-manager__action-btn--secondary" @click="resetMembers" v-if="!useRealData">
                  {{ i18n("重置数据") }}
                </button>
                <button class="api-manager__action-btn api-manager__action-btn--secondary" @click="clearMembers" v-if="!useRealData">
                  {{ i18n("清空数据") }}
                </button>
                <button class="api-manager__action-btn api-manager__action-btn--secondary" @click="refreshRealMembers" v-if="useRealData">
                  {{ i18n("刷新数据") }}
                </button>
              </div>
            </div>

            <DemoSection id="atoms" title="Atoms" subtitle="基础设计令牌与通用控件（字体/颜色/按钮/输入等）">
              <DemoCard title="Typography" desc="字体大小/粗细/颜色示例">
                <div class="DesignSystemDev__typo">
                  <div class="DesignSystemDev__typo-h1">H1 / 16 / 900</div>
                  <div class="DesignSystemDev__typo-h2">H2 / 14 / 800</div>
                  <div class="DesignSystemDev__typo-p">Body / 12~14 / 600</div>
                  <div class="DesignSystemDev__typo-muted">{{ i18n("辅助文字示例") }}</div>
                </div>
              </DemoCard>
              <DemoCard title="Color Tokens" desc="来自全局 CSS 变量的颜色令牌">
                <div class="DesignSystemDev__swatches">
                  <div class="DesignSystemDev__swatch" v-for="t in colorTokens" :key="t.name">
                    <div class="DesignSystemDev__swatch-color" :style="{ background: t.value }"></div>
                    <div class="DesignSystemDev__swatch-name">{{ t.name }}</div>
                  </div>
                </div>
              </DemoCard>
              <DemoCard title="Buttons (xBtn)" desc="通用按钮控件（由 ui-x 提供）">
                <div class="DesignSystemDev__row">
                  <xBtn :configs="{ label: i18n('主要'), preset: 'blue' }" />
                  <xBtn :configs="{ label: i18n('次要') }" />
                  <xBtn :configs="{ label: i18n('危险'), preset: 'danger' }" />
                </div>
              </DemoCard>
              <DemoCard title="Icons (xIcon)" desc="通用图标控件（由 ui-x 提供）">
                <div class="DesignSystemDev__row">
                  <xIcon icon="users" :size="18" />
                  <xIcon icon="plus" :size="18" />
                  <xIcon icon="delete" :size="18" />
                  <xIcon icon="settings" :size="18" />
                </div>
              </DemoCard>
            </DemoSection>

            <DemoSection id="molecules" title="Molecules" subtitle="通过 atoms 组合的可复用片段（偏布局/结构/语义）">
              <DemoCard title="DemoCard" desc="演示页统一卡片容器（分标题/说明/内容）">
                <div class="DesignSystemDev__hint">{{ i18n("用于展示不同组件/状态的统一外观容器") }}</div>
              </DemoCard>
              <DemoCard title="DemoSection" desc="演示页分区容器（锚点 + 栅格布局）">
                <div class="DesignSystemDev__hint">{{ i18n("用于按层级组织展示内容") }}</div>
              </DemoCard>
              <DemoCard title="MemberIdentity" desc="成员身份展示（头像 + 名称 + UID）">
                <div class="DesignSystemDev__stack">
                  <MemberIdentity :member="{ uid: 1001, username: 'Alice', email: 'alice@example.com', role: 'owner' }" />
                  <MemberIdentity :member="{ uid: 1004, username: '', email: '', role: 'dev' }" />
                </div>
              </DemoCard>
              <DemoCard title="MemberIdentity (long name)" desc="超长名称截断展示">
                <MemberIdentity :member="{ uid: 1005, username: 'VeryVeryLongUserNameForEllipsisTest', email: '', role: 'dev' }" />
              </DemoCard>
            </DemoSection>

            <DemoSection id="organisms" title="Organisms" subtitle="可复用业务组件块（props + emits；由业务页面负责数据与权限）">
              <DemoCard title="MemberListPanel (editable)" desc="可编辑态：Add/改角色/删除">
                <MemberListPanel
                  :groupName="useRealData ? (cptEffectiveGroupName || groupName) : groupName"
                  :members="cptEffectiveMembers"
                  :loading="cptEffectiveLoading"
                  :canManage="canManage"
                  @add-member="openAddMember"
                  @remove-member="removeMember"
                  @change-member-role="changeMemberRole" />
              </DemoCard>
              <DemoCard title="MemberListPanel (readonly)" desc="只读态：仅展示列表，不展示操作">
                <MemberListPanel
                  :groupName="groupName"
                  :members="members"
                  :loading="false"
                  :canManage="false" />
              </DemoCard>
              <DemoCard title="MemberListPanel (loading)" desc="加载态：展示 loading">
                <MemberListPanel
                  :groupName="groupName"
                  :members="members"
                  :loading="true"
                  :canManage="false" />
              </DemoCard>
              <DemoCard title="MemberListPanel (empty)" desc="空态：展示 empty">
                <MemberListPanel
                  :groupName="groupName"
                  :members="[]"
                  :loading="false"
                  :canManage="false" />
              </DemoCard>
            </DemoSection>

            <DemoSection id="apps" title="Apps" subtitle="应用/弹窗：用于 CRUD 的操作承载（统一由窗口管理器管理）">
              <DemoCard title="Member Upsert Dialog" desc="演示新增成员弹窗（表单 + 校验 + 回调）">
                <button class="api-manager__action-btn api-manager__action-btn--primary api-manager__action-btn--with-icon" @click="openAddMember">
                  <xIcon icon="plus" :size="16" />
                  {{ i18n("打开弹窗") }}
                </button>
              </DemoCard>
              <DemoCard title="Interaction Notes" desc="交互模式：确认→调用→刷新/更新">
                <div class="DesignSystemDev__hint">
                  {{ i18n("删除/改角色都走确认弹窗；业务接入时替换为真实接口调用并刷新列表") }}
                </div>
              </DemoCard>
            </DemoSection>
          </main>
        </div>
      </div>
    `
	};
}
</script>

<style lang="less">
.DesignSystemDev {
	height: 100%;
	width: 100%;
	background: var(--color-surface-container-lowest);
	color: var(--color-on-surface);
}

.DesignSystemDev__layout {
	height: 100%;
	display: grid;
	grid-template-columns: 220px 1fr;
	gap: 16px;
	padding: 16px;
	min-height: 0;
}

.DesignSystemDev__nav {
	position: sticky;
	top: 0;
	align-self: start;
	height: calc(100vh - 32px);
	border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	border-radius: 14px;
	background: var(--color-surface);
	padding: 12px;
	display: flex;
	flex-direction: column;
	gap: 6px;
}

.DesignSystemDev__nav-title {
	font-weight: 900;
	color: var(--color-on-surface);
	padding: 8px 10px;
	border-radius: 10px;
	background: color-mix(in srgb, var(--color-surface-container) 30%, transparent);
}

.DesignSystemDev__nav-link {
	display: block;
	padding: 8px 10px;
	border-radius: 10px;
	color: var(--color-on-surface-variant);
	text-decoration: none;
	font-weight: 700;
}

.DesignSystemDev__nav-link:hover {
	background: var(--color-surface-variant);
	color: var(--color-on-surface);
}

.DesignSystemDev__content {
	min-height: 0;
	overflow: auto;
	display: flex;
	flex-direction: column;
	gap: 18px;
	padding-right: 8px;
}

.DesignSystemDev__topbar {
	display: flex;
	align-items: flex-start;
	justify-content: space-between;
	gap: 12px;
	background: var(--color-surface);
	border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	border-radius: 14px;
	padding: 14px;
}

.DesignSystemDev__topbar-left {
	display: flex;
	flex-direction: column;
	gap: 6px;
	min-width: 0;
}

.DesignSystemDev__h1 {
	font-size: 16px;
	font-weight: 900;
	color: var(--color-on-surface);
}

.DesignSystemDev__hint {
	font-size: 12px;
	font-weight: 600;
	color: var(--color-on-surface-variant);
}

.DesignSystemDev__hint--error {
	color: var(--color-error);
}

.DesignSystemDev__toolbar {
	display: flex;
	gap: 8px;
	flex-wrap: wrap;
	justify-content: flex-end;
}

.DesignSystemDev__stack {
	display: flex;
	flex-direction: column;
	gap: 10px;
}

.DesignSystemDev__row {
	display: flex;
	align-items: center;
	gap: 10px;
	flex-wrap: wrap;
}

.DesignSystemDev__typo {
	display: flex;
	flex-direction: column;
	gap: 8px;
}

.DesignSystemDev__typo-h1 {
	font-size: 16px;
	font-weight: 900;
	color: var(--color-on-surface);
}

.DesignSystemDev__typo-h2 {
	font-size: 14px;
	font-weight: 800;
	color: var(--color-on-surface);
}

.DesignSystemDev__typo-p {
	font-size: 14px;
	font-weight: 600;
	color: var(--color-on-surface);
}

.DesignSystemDev__typo-muted {
	font-size: 12px;
	font-weight: 600;
	color: var(--color-on-surface-variant);
}

.DesignSystemDev__swatches {
	display: grid;
	grid-template-columns: repeat(3, minmax(0, 1fr));
	gap: 10px;
}

.DesignSystemDev__swatch {
	border: 1px solid color-mix(in srgb, var(--color-outline-variant) 50%, transparent);
	border-radius: 12px;
	overflow: hidden;
	background: var(--color-surface);
}

.DesignSystemDev__swatch-color {
	height: 28px;
}

.DesignSystemDev__swatch-name {
	padding: 8px 10px;
	font-size: 12px;
	font-weight: 700;
	color: var(--color-on-surface-variant);
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

@media (max-width: 960px) {
	.DesignSystemDev__layout {
		grid-template-columns: 1fr;
	}
	.DesignSystemDev__nav {
		position: relative;
		height: auto;
	}
	.DesignSystemDev__swatches {
		grid-template-columns: repeat(2, minmax(0, 1fr));
	}
}
</style>
