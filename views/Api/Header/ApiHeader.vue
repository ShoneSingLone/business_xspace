<template>
	<header class="AppLayoutHeader flex middle">
		<span class="flex middle xspaceLogo pointer" @click="goToGroup">
			<xIcon :icon="icon" :style="logoStyle" />
		</span>
		<XspaceBreadcrumbNavigation />
		<CollabUserContainer />
		<xGap f />
		<TransitionGroup name="fade" mode="out-in" tag="div" class="flex middle">
			<!-- 			v-xtips="{ content: '国际化', trigger: 'hover', placement: 'bottom' }" -->
			<a class="flex middle mr4" :href="i18nHref" target="_blank" title="国际化" key="国际化">
				<xIcon icon="_icon_i18n" />
			</a>
			<!-- 			v-xtips="{ content: 'hoppscotch', trigger: 'hover', placement: 'bottom' }" -->
			<a
				class="flex middle mr4"
				:href="hoppscotchHref"
				target="_blank"
				title="hoppscotch"
				key="hoppscotch">
				<xIcon icon="_hoppscotch" />
			</a>
			<!-- 			v-xtips="{ content: '文档', trigger: 'hover', placement: 'bottom' }" -->
			<a
				class="flex middle mr4"
				:href="publicNoteHref"
				target="_blank"
				title="文档"
				key="文档">
				<xIcon icon="_wikidoc" />
			</a>
			<!-- 			v-xtips="{ content: 'webrtc', trigger: 'hover', placement: 'bottom' }" -->
			<a
				class="flex middle mr4"
				:href="publicRtcHref"
				target="_blank"
				title="webrtc"
				key="webrtc">
				<xIcon icon="_webrtc" />
			</a>
			<!-- 			v-xtips="{ content: '资源访问', trigger: 'hover', placement: 'bottom' }" -->
			<a
				class="flex middle mr4"
				:href="privateExploreHref"
				target="_blank"
				title="资源访问"
				key="资源访问">
				<xIcon icon="_hamburger" />
			</a>
			<!-- 			v-xtips="{ content: '部署项目', trigger: 'hover', placement: 'bottom' }" -->
			<a
				@click="deploy"
				v-if="cptIsAdmin"
				class="flex middle mr4"
				target="_blank"
				title="部署项目"
				key="部署项目">
				<xIcon icon="_deploy" />
			</a>
			<!-- 			v-xtips="{ content: '用户管理', trigger: 'hover', placement: 'bottom' }" -->
			<a
				@click="onClickUserManager"
				v-if="cptIsAdmin"
				class="flex middle mr4"
				target="_blank"
				title="用户管理"
				key="用户管理">
				<xIcon icon="_user_manager" />
			</a>
		</TransitionGroup>
		<xspaceToolUserBar />
	</header>
</template>

<script lang="ts">
export default async function () {
	return {
		inject: ["APP"],
		name: "AppLayoutHeader",
		components: {
			CollabUserContainer: () => _.$importVue("@/views/Api/Header/CollabUserContainer.vue"),
			xspaceToolUserBar: () => _.$importVue("@/components/XspaceToolUserBar.vue"),
			XspaceBreadcrumbNavigation: () =>
				_.$importVue("@/components/XspaceBreadcrumbNavigation.vue")
		},
		data() {
			return {};
		},
		computed: {
			cptIsAdmin() {
				return this.APP.user.role === "admin";
			},
			hoppscotchHref() {
				return _.$aHashLink("/hoppscotch", {});
			},
			i18nHref() {
				return _.$aHashLink("/i18n", {});
			},
			publicNoteHref() {
				return _.$aHashLink("/note", {});
			},
			publicRtcHref() {
				return _.$aHashLink("/rtc", {});
			},
			privateExploreHref() {
				return _.$aHashLink("/explore", {});
			},
			icon() {
				if (["/api/group", "/wiki", "/xI"].includes(this.$route.path)) {
					return "_xspace_logo";
				}
				return "_xspace_logo";
			},
			logoStyle() {
				return { width: "48px", height: "48px" };
			}
		},
		methods: {
			onClickUserManager() {
				this.$router.push("/user");
			},
			async deploy() {
				_.$openModal({
					title: "Deploy Project",
					url: "@/components/deploy/deploy.dialog.vue",
					isHideHeader: true
				});
			},
			goToGroup() {
				this.$router.push("/api/group");
			}
		}
	};
}
</script>

<style lang="less">
.AppLayoutHeader {
	display: flex;
	align-items: center;
	height: 64px;
	padding-left: 20px;
	line-height: 44px;
	background-color: var(--el-color-white);
	box-shadow: var(--el-box-shadow-lighter);
	z-index: 1;

	.xspaceLogo {
		box-shadow: var(--el-box-shadow-lighter);
		// border: 1px solid gray;
		border-radius: 50%;
		margin-right: var(--ui-one);
	}
}
</style>
