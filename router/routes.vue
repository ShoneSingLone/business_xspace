<script lang="ts">
export default async function () {
	const ComponentRouterView = {
		render(h) {
			return h("router-view");
		}
	};
	return [
		_.$newRoute("/v1", "@/views/v1/ViewXspace.vue"),
		_.$newRoute("/demo_test", "@/views/demo/test.vue"),
		_.$newRoute("/cicd", "@/views/CiCd/ViewCiCd.vue", {
			redirect: "/cicd/task_list",
			children: [
				_.$newRoute("/cicd/dashboard", "@/views/CiCd/views/CiCdDashboard.vue"),
				_.$newRoute("/cicd/task_list", "@/views/CiCd/views/CiCdTask.vue"),
				_.$newRoute("/cicd/builds", "@/views/CiCd/views/CiCdBuilds.vue"),
				_.$newRoute("/cicd/artifacts", "@/views/CiCd/views/CiCdArtifacts.vue"),
				_.$newRoute("/cicd/webhooks", "@/views/CiCd/views/CiCdWebhooks.vue"),
				_.$newRoute("/cicd/settings", "@/views/CiCd/views/CiCdSettings.vue")
			]
		}),
		_.$newRoute("/user", "@/views/User/ViewUser.vue"),
		_.$newRoute("/i18n", "@/views/I18n/I18nLayout.vue", {
			redirect: "/i18n/all",
			children: [_.$newRoute("/i18n/all", "@/views/I18n/I18n.vue")]
		}),
		_.$newRoute("/note", "@/views/Note/Note.vue"),
		_.$newRoute("/office", "@/views/office/AppOffice.vue"),
		_.$newRoute("/im", "@/views/im/Im.vue"),
		_.$newRoute("/rtc", "@/views/Rtc/ViewRtc.vue", {
			children: [
				_.$newRoute("/rtc/demo", "@/views/Rtc/ViewRtcDemo.vue"),
				_.$newRoute("/rtc/peer_connection", "@/views/Rtc/ViewRtcPeerConnection.vue"),
				_.$newRoute(
					"/rtc/peer_connection_canvas",
					"@/views/Rtc/ViewRtcPeerConnectionCanvas.vue"
				),
				_.$newRoute("/rtc/data_channel", "@/views/Rtc/ViewRtcDataChannel.vue"),
				_.$newRoute("/rtc/data_channel_file", "@/views/Rtc/ViewRtcDataChannelFile.vue"),
				_.$newRoute("/rtc/p2p", "@/views/Rtc/ViewRtcP2PLogin.vue")
			]
		}),
		_.$newRoute("/hoppscotch", "@/views/hoppscotch/ViewHoppscotch.vue"),
		_.$newRoute("/explore", "@/views/explore/ViewExplore.vue"),
		_.$newRoute("/login", "@/views/Login/Login.vue"),
		_.$newRoute("/docflow", "@/views/DocFlow/ViewDocFlow.vue", {
			redirect: "/docflow/list",
			children: [
				_.$newRoute("/docflow/list", "@/views/DocFlow/views/DocFlowList.vue"),
				_.$newRoute("/docflow/document/:id", "@/views/DocFlow/views/DocFlowDocument.vue")
			]
		}),
		_.$newRoute("/", ComponentRouterView, {
			redirect: "/api/group"
		}),
		_.$newRoute("/api", "@/views/Api/Api.vue", {
			redirect: "/api/group",
			children: [
				_.$newRoute("/api/group", "@/views/Api/Group/Group.vue"),
				_.$newRoute("/api/project", "@/views/Api/Project/Project.vue")
			]
		}),
		_.$newRoute("/404", "@/views/NotFound.vue"),
		{
			/* 兜底 */
			path: "*",
			redirect: "/404"
		}
	];
}
</script>
