<template>
	<xDialog class="as-div">
		<div class="container width100">
			<div class="terminal_toolbar flex">
				<xGap f />
				<div class="butt">
					<button class="btn btn-color" @click="closeModal">X</button>
				</div>
			</div>
			<div class="terminal_body">
				<div class="terminal_promt flex vertical pl pr">
					<div class="terminal_user" v-for="(item, index) in state.msgArray" :key="index">
						{{ item }}
					</div>
					<span class="terminal_cursor"></span>
				</div>
			</div>
		</div>
	</xDialog>
</template>
<script lang="ts">
export default async function ({ row, onSuccess }) {
	const { useDialogProps } = await _.$importVue("/common/utils/hooks.vue");
	return defineComponent({
		inject: ["APP"],
		props: useDialogProps(),
		setup() {
			const clone_git_repo_terminal_output = "clone_git_repo_terminal_output";
			const state = reactive({ msgArray: [] });
			onMounted(() => {
				try {
					this.APP.WS.on(clone_git_repo_terminal_output, msg => {
						if (msg === "克隆成功") {
							onSuccess();
						}
						state.msgArray.unshift(msg);
					});
					/* 发送clone仓库的请求 */
					_api.xspace.apiCicdGitInitRepo({
						git_address: row.git_address
					});
				} catch (error) {
					console.error(error);
				}
			});
			onBeforeUnmount(() => {
				this.APP.WS.off(clone_git_repo_terminal_output);
			});

			return {
				state
			};
		},
		computed: {
			isUpdate() {
				return !!row;
			},
			btnOk() {
				const vm = this;
				return {
					label: i18n("ok"),
					preset: "blue",
					async onClick() {
						vm.onClickOk();
					}
				};
			}
		},
		methods: {
			async onClickOk() {
				const [error] = await _.$validateForm(this.el);
				if (error) {
					return;
				}
				this.closeModal();
			}
		}
	});
}
</script>
<style lang="less">
.container {
	width: 100%;
	height: 250px;
	overflow: auto;
}

.terminal_toolbar {
	display: flex;
	height: 30px;
	align-items: center;
	padding: 0 8px;
	box-sizing: border-box;
	border-top-left-radius: 5px;
	border-top-right-radius: 5px;
	background: #212121;
	justify-content: space-between;
}

.butt {
	display: flex;
	align-items: center;
}

.btn {
	display: flex;
	justify-content: center;
	align-items: center;
	padding: 0;
	margin-right: 5px;
	font-size: 8px;
	height: 12px;
	width: 12px;
	box-sizing: border-box;
	border: none;
	border-radius: 100%;
	background: linear-gradient(#7d7871 0%, #595953 100%);
	text-shadow: 0px 1px 0px rgba(255, 255, 255, 0.2);
	box-shadow:
		0px 0px 1px 0px #41403a,
		0px 1px 1px 0px #474642;
}

.btn-color {
	background: #ee411a;
}

.btn:hover {
	cursor: pointer;
}

.btn:focus {
	outline: none;
}

.butt--exit {
	background: linear-gradient(#f37458 0%, #de4c12 100%);
}

.add_tab {
	border: 1px solid #fff;
	color: #fff;
	padding: 0 6px;
	border-radius: 4px 4px 0 0;
	border-bottom: none;
	cursor: pointer;
}

.user {
	color: #d5d0ce;
	margin-left: 6px;
	font-size: 14px;
	line-height: 15px;
}

.terminal_body {
	background: rgba(0, 0, 0, 0.6);
	height: calc(100% - 30px);
	padding-top: 2px;
	margin-top: -1px;
	font-size: 12px;
	border-bottom-left-radius: 5px;
	border-bottom-right-radius: 5px;
	overflow: auto;
}

.terminal_promt {
	display: flex;
}

.terminal_promt span {
	margin-left: 4px;
}

.terminal_user {
	color: #1eff8e;
}

.terminal_location {
	color: #4878c0;
}

.terminal_bling {
	color: #dddddd;
}

.terminal_cursor {
	display: block;
	height: 14px;
	width: 5px;
	margin-left: 10px;
	animation: curbl 1200ms linear infinite;
}

@keyframes curbl {
	0% {
		background: #ffffff;
	}

	49% {
		background: #ffffff;
	}

	60% {
		background: transparent;
	}

	99% {
		background: transparent;
	}

	100% {
		background: #ffffff;
	}
}
</style>
f
