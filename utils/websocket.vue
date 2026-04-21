<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
	const $$io_val = {
		user_login: "user_login",
		user_logout: "user_logout",
		event_id: "event_id"
	};

	_.$$io_val = $$io_val;

	return {
		async initWebsocket({ vm, user }) {
			const uid = user._id;
			return new Promise(async resolve => {
				const wsOptions = {
					TRIGGER_EVENT_NAME: $$io_val.event_id,
					onConnection: ({ ws, id, current_online_user }) => {
						/* ws:连接之后通知所有人用户登录 */
						ws.emit("all", {
							type: $$io_val.user_login,
							payload: _.merge(
								{ current_online_user },
								_.pick(user, ["username", "_id"])
							)
						});
						resolve(id);
					},
					namespace: `/xspace?uid=${uid}`
				};

				vm.WS = await _.$importVue("/common/libs/socket.io.vue", wsOptions);

				$(window).on($$io_val.event_id, function (event, params) {
					try {
						const { type, payload } = params;
						if (type === $$io_val.user_login) {
							const { current_online_user } = payload;
							_.$val(
								vm,
								`current_online_user`,
								_.map(current_online_user, _id => {
									return {
										username: "",
										_id
									};
								})
							);
							return;
						}

						if (type === $$io_val.user_logout) {
							const { uid } = payload;
							return;
						}
						vm.$emit(type, payload);
					} catch (error) {
						_.$msgError(error);
					}
				});
			});
		}
	};
}
</script>
