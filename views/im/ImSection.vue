<style lang="less">
#ImSection {
	width: 1px;
	/* 聊天对话框容器 */

	.chat-dialog {
		width: 100%;
		background-color: #fff;
		box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
		border-radius: 10px;
	}

	/* 聊天消息区域 */

	.chat-messages {
		padding: 15px;
		overflow-y: auto;
	}

	.chat-with-avatar {
		width: 32px;
		height: 32px;
	}

	/* 自己发送的消息框样式 */

	.message-self {
		background-color: #e6f7ff;
		border-radius: 5px;
		padding: 10px;
		margin-bottom: 10px;
		position: relative;
	}

	/* 自己发送消息框的尖角 */

	.message-self::after {
		content: "";
		position: absolute;
		right: -10px;
		top: 10px;
		border-top: 10px solid transparent;
		border-bottom: 10px solid transparent;
		border-left: 10px solid #e6f7ff;
	}

	/* 对方发送的消息框样式 */

	.message-other {
		background-color: #ffffff;
		border-radius: 5px;
		padding: 10px;
		margin-bottom: 10px;
		position: relative;
	}

	/* 对方发送消息框的尖角 */

	.message-other::after {
		content: "";
		position: absolute;
		left: -10px;
		top: 10px;
		border-top: 10px solid transparent;
		border-bottom: 10px solid transparent;
		border-right: 10px solid #ffffff;
	}

	/* 消息内容文字样式 */

	.message-content {
		color: #333333;
		font-size: 14px;
	}

	/* 发送时间样式 */

	.message-time {
		color: #999999;
		font-size: 12px;
		text-align: right;
	}

	/* 容器样式 */

	.container {
		text-align: center;
		padding: 40px 20px;
		background-color: white;
		border-radius: 12px;
		box-shadow: 0 10px 25px rgba(0, 0, 0, 0.05);
		transition: transform 0.3s ease;
	}

	.container:hover {
		transform: translateY(-5px);
	}

	/* 图标样式 */

	.icon-container {
		width: 100px;
		height: 100px;
		margin: 0 auto 30px;
		position: relative;
	}

	.user-icon {
		* {
			padding: 10px;
		}

		width: 100%;
		height: 100%;
		background-color: #e1f5fe;
		border-radius: 50%;
		position: relative;
		overflow: hidden;
	}

	.user-icon::before {
		content: "";
		position: absolute;
		top: 25px;
		left: 25px;
		width: 50px;
		height: 50px;
		border-radius: 50%;
		background-color: #2196f3;
	}

	.user-icon::after {
		content: "";
		position: absolute;
		bottom: 15px;
		left: 0;
		width: 100px;
		height: 40px;
		border-radius: 50%;
		background-color: #2196f3;
	}

	.question-mark {
		position: absolute;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		font-size: 32px;
		font-weight: bold;
		color: white;
		z-index: 1;
	}

	/* 文本样式 */

	h1 {
		font-size: 24px;
		margin-bottom: 15px;
		color: #2d3748;
	}

	p {
		font-size: 16px;
		color: #718096;
		margin-bottom: 30px;
		line-height: 1.6;
	}
}
</style>
<template>
	<section class="x-page-view flex1 flash-when" id="ImSection">
		<xPageContent v-if="inject_im.cpt_im_chat_with.avatar">
			<div class="chat-width-wrapper flex middle">
				<div>
					<xIcon
						:key="inject_im.cpt_im_chat_with.uid"
						:img="inject_im.cpt_im_chat_with.avatar"
						class="chat-with-avatar"
						iscache="true" />
					<div>
						{{ inject_im.cpt_im_chat_with.username }}
					</div>
				</div>
			</div>
			<div class="flex1 overflow-auto chat-dialog mt mb" ref="refChatContent">
				<div ref="refChatContentWrapper" class="chat-messages">
					<xspaceChatContentItem
						class="content-item"
						:item="item"
						v-for="item in contentArray"
						:key="item.add_time" />
				</div>
			</div>
			<div class="chat-content-editor">
				<TuiEditor
					:value="{ md: newChatContent }"
					@change="val => (newChatContent = val.md)"
					:asRender="false"
					style="height: 300px"
					@keydown="onKeydown" />
			</div>
			<div class="flex center width100 mt">
				<xBtn :configs="cptSendBtn" ref="refSendNewChatBtn" />
			</div>
		</xPageContent>
		<xPageContent v-else>
			<div class="flex middle vertical center height100">
				<div class="icon-container">
					<div class="user-icon"></div>
					<div class="question-mark">?</div>
				</div>
				<h1>未选择对话对象</h1>
				<p>请先从联系人列表中选择一个对话对象，然后才能开始聊天。</p>
			</div>
		</xPageContent>
	</section>
</template>
<script lang="ts">
export default async function () {
	return defineComponent({
		inject: ["APP", "inject_im"],
		components: {
			xspaceChatContentItem: () => _.$importVue("@/components/XspaceChatContentItem.vue")
		},
		setup() {
			onMounted(() => {
				this.APP.$on("chat_one", item => {
					this.contentArray.push(item);
					this.setPosition();
				});
			});

			onUnmounted(() => {
				this.APP.$off("chat_one");
			});
		},
		data() {
			return {
				loading: false,
				newChatContent: "",
				contentArray: []
			};
		},
		computed: {
			cptSendBtn({ inject_im, newChatContent, sendNewChat }) {
				return {
					label: "发送(Ctrl+Enter)",
					preset: "primary",
					disabled: !inject_im.cpt_im_chat_with.uid || !newChatContent,
					onClick: () => sendNewChat()
				};
			},
			cptBelongParams() {
				return {
					wsId: this.APP.WS_ID,
					belong_type: "chat_one",
					belong_id: [this.APP.user._id, this.inject_im.cpt_im_chat_with.uid]
						.map(i => Number(i))
						.sort((a, b) => a - b)
						.join("_")
				};
			}
		},
		methods: {
			setPosition() {
				setTimeout(() => {
					const height = $(this.$refs.refChatContentWrapper).height();
					this.$refs.refChatContent.scrollTo({
						top: height,
						behavior: "smooth"
					});
				}, 1000);
			},
			async updateContent() {
				try {
					const {
						data: { list }
					} = await _api.xspace.wikiList(this.cptBelongParams);
					this.contentArray = list;
					this.setPosition();
				} catch (error) {
					console.error(error);
				} finally {
				}
			},
			onKeydown({ event, type }) {
				const isCtrlOrCommand = event.ctrlKey || event.metaKey;
				if (isCtrlOrCommand) {
					if (event.key === "Enter" && this.$refs?.refSendNewChatBtn) {
						this.$refs.refSendNewChatBtn.handleClick();
					}
				}
			},
			async sendNewChat() {
				const vm = this;
				try {
					_.$loading(true);
					const params = _.merge(this.cptBelongParams, {
						title: "chat_content",
						type: Vue._xspace_var.ARTICLE,
						p_id: vm.pid,
						markdown: String(vm.newChatContent)
					});
					await _api.xspace.wiki_upsert_one(params);
					vm.$nextTick(() => {
						vm.newChatContent = "";
					});
				} catch (error) {
					console.error(error);
				} finally {
					_.$loading();
				}
			}
		},
		watch: {
			"inject_im.cpt_im_chat_with"(cpt_im_chat_with) {
				if (cpt_im_chat_with.uid) {
					this.updateContent();
				}
			}
		}
	});
}
</script>
