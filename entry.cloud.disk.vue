<script lang="ts">
export default async function () {
	const [, [VueRouter, routes, { useMusic }]] = await Promise.all([
		Promise.all([
			_.$importVue("/common/ui-x/useXui.vue")
			// _.$importVue("/common/ui-element/useElementUI.NoJS.vue", { size: "small", I18N_LANGUAGE: window.I18N_LANGUAGE })
		]),
		_.$importVue([
			"/common/libs/VueRouter.vue",
			"@/router/cloudDisk/routes.vue",
			"@/views/CloudDisk/CloudDiskMine.useMusic.vue",
			/*常量*/
			"@/utils/var.vue",
			/*接口*/
			"@/utils/api.vue",
			/*校验规则*/
			"/common/utils/rules.vue",
			/*枚举选项*/
			"@/utils/utils.vue",
			/*工具函数*/
			// "@/utils/helper.vue",
			/*复用组件*/
			// "@/utils/reuse.vue"
			/*通用下拉项*/
			"@/utils/opts.vue",
			/* 项目独有样式 */
			"@/xspace.defaul.style.vue"
		])
	]);
	/* app entry  */
	const router = new VueRouter({ routes });
	const LOADING_STATUS = 0;
	const GUEST_STATUS = 1;
	const MEMBER_STATUS = 2;
	/*  */
	_.$xspaceRouter = router;

	// router.beforeEach(function (to, from) {});

	const fileRecords = (await _.$idb.get(`FILE_RECORDS`)) || {};

	var rootApp = new Vue({
		el: "#app",
		router,
		components: {
			ViewApp: () => _.$importVue("@/layout/AppLayoutLevel.vue")
		},
		template: `<ViewApp/>`,
		provide() {
			return {
				APP: this
			};
		},
		setup() {
			const vm = this;
			const { LOOP_TYPE_NAME_ARRAY, playAudio, playMedia, stateAudio, methodsMusicPlayer } =
				useMusic(this);

			onMounted(() => {
				vm.$watch(
					() => {
						return vm.homeListSearchKey;
					},
					_.debounce(function () {
						vm.getResourceArray();
					}, 1000)
				);
			});

			return {
				LOOP_TYPE_NAME_ARRAY,
				playAudio,
				playMedia,
				stateAudio,
				methodsMusicPlayer
			};
		},
		mounted() {
			this.initMobileModel();
			this.refreshUserInfo();
			this.loadDirs().then(res => {
				if (!_.$isSame(this.fileId, 0)) {
					const breadcrumbItems = [];
					let item;
					let id = this.fileId;
					while ((item = this.dirs[id])) {
						breadcrumbItems.unshift({
							label: item.name,
							fileId: item._id
						});
						id = item.fileId;
					}
					this.breadcrumbItems = [...this.breadcrumbItems, ...breadcrumbItems];
				}
			});
		},
		data() {
			const vm = this;
			vm.refreshUserInfo = _.$asyncDebounce(
				vm,
				async function refreshUserInfo() {
					try {
						if (!vm.user.isLogin) {
							const { data: userInfo } = await _api.xspace.userStatus();
							vm._setUser(userInfo);
						}

						if (vm.user.isLogin) {
							const { data: all_user } = await _api.xspace.userSearch({});
							vm.all_user = all_user;

							/* TODO: 跳转到首页 或者应用*/
							if (["/resource", "/transfer", "/me"].includes(vm.$route.path)) {
								return;
							} else {
								vm.$router.push({
									path: "/resource",
									query: {
										...this.$route.query
									}
								});
							}
						}
						return true;
					} catch (error) {
						/* 未登录，跳转登录界面 */
						vm.$router.push("/login");
					} finally {
						$("body").removeClass("x-loading");
					}
				},
				10
			);

			vm.saveFileRecords = _.debounce(function () {
				_.$idb.set(`FILE_RECORDS`, vm.fileRecords);
			}, 1000 * 3);

			return {
				dirs: {},
				dirTree: [],
				fileRecords,
				breadcrumbItems: [
					{
						label: "我的空间",
						fileId: 0
					}
				],
				/* *********************** */
				homeListSearchKey: "",
				tabArray: [
					{ label: "资源", icon: "_cloud_home_tab", path: "/resource" },
					{ label: "传输", icon: "_cloud_trans_tab", path: "/transfer" },
					{ label: "我的", icon: "_UserOutlined", path: "/me" }
				],
				/* header 按钮 */
				isShowBMoreDrawer: false,
				isShowResourceDrawer: false,
				/* ****************** */
				isMobile: /Mobile/gi.test(window.navigator.userAgent),
				useMobileView: true,
				BASE_URL: window.__BASE_URL || window.location.origin,
				user: {
					add_time: "",
					email: "",
					role: "",
					study: false,
					type: "",
					up_time: "",
					username: "",
					_id: "",
					isLogin: false,
					isLDAP: false,
					uid: null,
					loginState: LOADING_STATUS,
					breadcrumb: [],
					studyTip: 0,
					imageUrl: "",
					cloudDiskSizeUsed: 0,
					cloudDiskSizeTotal: 0
				},
				listSortBy: "name",
				/* resource 资源列表 */
				resourceList: [],
				selectedItems: []
			};
		},
		methods: {
			async getResourceArray() {
				_.$loading(true);
				try {
					const params = {
						fileId: this.fileId || 0,
						orderby: this.listSortBy,
						name: this.homeListSearchKey || ""
					};
					const { data } = await _api.xspace.resourceCloudDiskFileList(params);
					this.resourceList = data;
					this.mSetResources();
				} catch (error) {
					console.error(error);
				} finally {
					_.$loading(false);
				}
			},
			mSetResources() {
				const vm = this;
				let { resourceList } = vm;
				resourceList = resourceList || [];
				const resource = _.groupBy(resourceList, "isdir");
				const sort = (groupedResourceArray = []) => {
					return _.map(groupedResourceArray, item => {
						let type = "none";

						if (item.isdir) {
							type = "dir";
						} else if (
							/^image/.test(item.type) ||
							["image/jpeg", "image/png"].includes(item.type)
						) {
							type = "image";
						} else if (/^video/.test(item.type)) {
							type = "video";
						} else if (/^audio/.test(item.type)) {
							type = "audio";
						}

						return {
							...item,
							type
						};
					}).sort((a, b) => {
						if (a.isdir === 1) {
							return -1;
						} else if (["name", "type"].includes(vm.sortBy)) {
							try {
								return a[vm.sortBy].localeCompare(b[vm.sortBy]);
							} catch (error) {
								return 0;
							}
						} else if (vm.sortBy === "add_time") {
							return a.add_time - b.add_time;
						} else {
							return 0;
						}
					});
				};
				const dirs = sort(resource[1]);
				const unDirs = sort(resource[0]);
				this.resourceList = [...dirs, ...unDirs];
				this.selectedItems = [];
			},
			async loadDirs() {
				const { data } = await _api.xspace.resourceCloudDiskGetDirs();
				const { TREE, NODES_OBJ: dirs } = _.$arrayToTree({
					rootId: "0",
					data,
					id: "_id",
					pid: "fileId",
					label: "name",
					value: "_id"
				});
				this.dirs = dirs;
				this.dirTree = [
					{
						id: "0",
						value: "0",
						label: "我的空间",
						children: TREE
					}
				];
			},
			triggerUploadFileChange(args) {
				const { md5 } = args;
				this.$set(this.fileRecords, md5, _.merge({}, this.fileRecords[md5], args));
				$(window).trigger("UPLOAD_FILE_CHANGE", md5);
				this.saveFileRecords();
			},
			pushDir(item) {
				this.fileId = item._id;
				this.breadcrumbItems.push({
					label: item.name,
					fileId: item._id
				});
			},
			popDir(index, item) {
				this.fileId = item.fileId;
				this.breadcrumbItems.splice(index + 1);
			},
			getShareUrl({ state }) {
				// #ifndef H5
				uni.getClipboardData({
					success: res => {
						if (res.data.includes("http://127.0.0.1:7001/")) {
							let key = res.data.substring(
								res.data.lastIndexOf("\/") + 1,
								res.data.length
							);
							if (!key) {
								return;
							}
							uni.showModal({
								content: "检测到有分享内容，是否打开？",
								success: res => {
									if (res.confirm) {
										uni.navigateTo({
											url: "/pages/shareurl/shareurl?key=" + key
										});
										// 清空剪切板
										uni.setClipboardData({
											data: ""
										});
									}
								}
							});
						}
					}
				});
				// #endif
			},
			clearList({ state }) {
				if (state.user) {
					uni.removeStorageSync("downlist_" + state.user.id);
					uni.removeStorageSync("uploadList_" + state.user.id);

					state.uploadList = [];
					state.downlist = [];
				}
			},
			initList({ state }) {
				if (state.user) {
					let d = uni.getStorageSync("downlist_" + state.user.id);
					let u = uni.getStorageSync("uploadList_" + state.user.id);

					state.downlist = d ? JSON.parse(d) : [];
					state.uploadList = u ? JSON.parse(u) : [];
				}
			},
			// 创建一个下载任务
			createDownLoadJob({ state }, obj) {
				state.downlist.unshift(obj);
				uni.setStorage({
					key: "downlist_" + state.user.id,
					data: JSON.stringify(state.downlist)
				});
			},
			// 更新下载任务进度
			updateDownLoadJob({ state }, obj) {
				let i = state.downlist.findIndex(item => item.key === obj.key);
				if (i !== -1) {
					state.downlist[i].progress = obj.progress;
					state.downlist[i].status = obj.status;
					uni.setStorage({
						key: "downlist_" + state.user.id,
						data: JSON.stringify(state.downlist)
					});
				}
			},
			// 创建一个上传任务
			createUploadJob({ state }, obj) {
				state.uploadList.unshift(obj);
				uni.setStorage({
					key: "uploadList_" + state.user.id,
					data: JSON.stringify(state.uploadList)
				});
			},
			// 更新上传任务进度
			updateUploadJob({ state }, obj) {
				let i = state.uploadList.findIndex(item => item.key === obj.key);
				if (i !== -1) {
					state.uploadList[i].progress = obj.progress;
					state.uploadList[i].status = obj.status;
					uni.setStorage({
						key: "uploadList_" + state.user.id,
						data: JSON.stringify(state.uploadList)
					});
				}
			},
			updateSize({ state }, e) {
				state.user.total_size = e.total_size;
				state.user.used_size = e.used_size;
			},
			/* ******************** */
			routerUpsertQuery(query) {
				this.$router.push({
					path: this.$route.path,
					query: {
						...this.$route.query,
						...query
					}
				});
			},
			initMobileModel() {
				if (this.isMobile) {
					$("body").addClass("app-mobile");
				}
			},
			/* ************************methods************************ */
			_setUser(userInfo) {
				const isLogin = !!userInfo._id;
				this.user = {
					...userInfo,
					isLogin,
					isLDAP: !!userInfo.ladp,
					role: userInfo ? userInfo.role : null,
					loginState: isLogin ? MEMBER_STATUS : GUEST_STATUS,
					username: userInfo ? userInfo.username : null,
					uid: userInfo ? isLogin : null,
					type: userInfo ? userInfo.type : null,
					study: userInfo ? userInfo.study : false
				};
			},
			/**
			 * 如果group是对象，直接赋值，
			 * 如果是Id(可能不是数字),则需要request
			 * @param group_id
			 * @returns {Promise<void>}
			 */
			async logoutActions() {
				try {
					const { data } = await _api.xspace.userLogout();
					if (data === "ok") {
						_.$lStorage.x_token = "";
						this._setUser({
							isLogin: false,
							loginState: GUEST_STATUS,
							username: null,
							uid: null,
							role: "",
							type: ""
						});
						this.$router.push("/login");
						_.$msg(i18n("退出成功! "));
					}
				} catch (error) {
					_.$msgError(error);
				}
			}
		},
		computed: {
			cptNavBarName: {
				get() {
					let item = _.find(this.tabArray, item => item.path === this.$route.path);
					if (!item) {
						item = _.first(this.tabArray);
						this.$nextTick(() => {
							this.$router.push({
								path: item.path,
								query: {
									...this.$route.query
								}
							});
						});
					}
					return item.label;
				},
				set(label) {
					const { path } = _.find(this.tabArray, { label });

					this.$nextTick(() => {
						this.$router.push({
							path,
							query: {
								...this.$route.query
							}
						});
					});
				}
			},
			fileId: {
				get() {
					return this.$route.query.fileId || 0;
				},
				set(fileId) {
					this.$router.push({
						path: this.$route.path,
						query: {
							...this.$route.query,
							fileId
						}
					});
				}
			}
		},
		watch: {
			fileId(fileId) {
				if (fileId !== 0) {
					if (this.breadcrumbItems.length === 1) {
						console.log("🚀 ~ fileId ~ this.breadcrumbItems:", this.breadcrumbItems);
					}
				}
			},
			$route: {
				deep: true,
				handler($route) {
					console.log("🚀 ~ handler ~ $route:", $route);
				}
			}
		}
	});

	return rootApp;
}
</script>

<style lang="less">
* {
	font-size: 14px;
}
</style>
