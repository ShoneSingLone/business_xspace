<script lang="ts">
export default async function () {
	if (!window._api.xspace) {
		(function () {
			_.$ajax.requestInjector = function (req) {
				req.url = Vue._common_utils.appendToken(req.url);
				return req;
			};
			_.$ajax.responseInjector = function (response) {
				if (response?.errcode == 40011) {
					/* 登录过期 */
					_.$msgError("登录过期，请重新登录");
					_.$xspaceRouter.push("/login");
				} else if (response?.errcode === 10001) {
					_.$msg.error("登录已失效，请重新登录");
					_.$router.push("/login");
				}

				if (response?.errcode !== 0) {
				}

				return response;
			};

			window._api.xspace = {
				async system_dicts(typeObject = {}) {
					const typeArray = Object.keys(typeObject);
					const _this = window._api.xspace;
					_this.typesCache = _this.typesCache || {};
					const needRequest = [];
					_.each(typeArray, type => {
						if (!_this.typesCache[type]) {
							needRequest.push(type);
						}
					});
					const { data } = await _.$ajax.post("/api/system/dicts", {
						data: { types: needRequest }
					});
					_this.typesCache = _.merge(_this.typesCache, data);
					return _this.typesCache;
				},

				i18nTranslate(data) {
					return _.$ajax.post("/api/i18n/translate", {
						data
					});
				},
				i18nUpsertOne(data) {
					return _.$ajax.post("/api/i18n/upsert_one", {
						data
					});
				},
				i18nGetList(data) {
					return _.$ajax.post("/api/i18n/get_list", {
						data
					});
				},
				wikiResetMenuOrder(data) {
					return _.$ajax.post("/api/wiki/reset_menu_order", {
						data
					});
				},
				wiki_upsert_one(data) {
					return _.$ajax.post("/api/wiki/upsert_one", {
						data
					});
				},
				saveImgByBase64(data) {
					return _.$ajax.post("/api/resource/base64img", {
						data
					});
				},
				wikiDetail(data) {
					return _.$ajax.get("/api/wiki/detail", {
						data
					});
				},
				/**
				 *  wiki左侧的菜单
				 *  http://192.168.0.107:3002/static/business_xspace/xspace.html#/api/project?project_id=319&group_id=431&project_tab_name=%E6%8E%A5%E5%8F%A3&interface_type=interface&interface_id=591&project_interface_tab=1&project_setting_tab=3/#/api/project?project_id=319&group_id=431&interface_type=interface&interface_id=591&project_interface_tab=1&project_setting_tab=3&project_tab_name=接口
				 */
				wiki_menu(data) {
					return _.$ajax.post("/api/wiki/menu", {
						data
					});
				},
				/**
				 *  文档 list
				 *  http://192.168.0.107:3002/static/business_xspace/xspace.html#/api/project?project_id=319&group_id=431&project_tab_name=%E6%8E%A5%E5%8F%A3&interface_type=interface&interface_id=589&project_interface_tab=1&project_setting_tab=3/#/api/project?project_id=319&group_id=431&interface_type=interface&interface_id=589&project_interface_tab=1&project_setting_tab=3&project_tab_name=接口
				 */
				wikiList(data) {
					return _.$ajax.post("/api/wiki/list", {
						data
					});
				},
				getSwaggerDataByUrl(data) {
					return _.$ajax.post("/api/project/swagger_url", {
						data
					});
				},
				resourceLs(data) {
					return _.$ajax.post("/api/resource/ls", {
						data
					});
				},
				resourceCloudDiskCheckChunks(data) {
					return _.$ajax.post("/api/resource/cloud_disk_check_chunks", {
						data
					});
				},
				resourceCloudDiskRename(data) {
					return _.$ajax.put("/api/resource/cloud_disk_rename", {
						data
					});
				},
				resourceCloudDiskDir(data) {
					return _.$ajax.post("/api/resource/cloud_disk_dir", {
						data
					});
				},
				resourceCloudDiskMoveDir(data) {
					return _.$ajax.post("/api/resource/cloud_disk_move_dir", {
						data
					});
				},
				resourceCloudDiskGetDirs(data) {
					return _.$ajax.get("/api/resource/cloud_disk_get_dirs", {
						data
					});
				},
				resourceCloudDiskFileList(data) {
					return _.$ajax.get("/api/resource/cloud_disk_file_list", {
						data
					});
				},
				audioDetail(data) {
					return _.$ajax.get("/api/audio/detail", {
						data
					});
				},
				resourceCloudDiskShardUpload({ formData, callback }) {
					return _.$ajax.upload({
						url: "/api/resource/cloud_disk_shard_upload",
						formData,
						callback
					});
				},
				/**
				 *  上传单个文件
				 */
				async apiResourceSingleUpload({ formData, callback }) {
					return _.$ajax.upload({
						url: "/api/resource/single_upload",
						formData,
						callback
					});
				},
				apiInterfaceListMenu(project_id) {
					return _.$ajax.get("/api/interface/list_menu", {
						data: { project_id }
					});
				},
				interface_get_by_id(data) {
					return _.$ajax.get("/api/interface/get", {
						data
					});
				},
				interface_del_by_ids(ids) {
					return _.$ajax.post("/api/interface/del", {
						data: {
							ids
						}
					});
				},
				interface_up(data) {
					return _.$ajax.post("/api/interface/up", {
						data
					});
				},
				interface_upsert(data) {
					return _.$ajax.post("/api/interface/upsert", {
						data
					});
				},
				interface_add(data) {
					return _.$ajax.post("/api/interface/add", {
						data
					});
				},
				interface_add_cat(data) {
					return _.$ajax.post("/api/interface/add_cat", {
						data
					});
				},
				interfaceUpCat(data) {
					return _.$ajax.post("/api/interface/up_cat", {
						data
					});
				},
				interface_usecase_upsert(data) {
					return _.$ajax.post("/api/usecase/upsert", {
						data
					});
				},
				interface_copy_to_project(data) {
					return _.$ajax.post("/api/interface/copy_to_project", {
						data
					});
				},
				interface_usecase_get_all(interface_id) {
					return _.$ajax.get("/api/usecase/get_all", {
						data: {
							id: interface_id
						}
					});
				},
				/* log */
				get_log_list({ typeid, type, page, size, query_params }) {
					return _.$ajax.post("/api/log/list", {
						data: { typeid, type, page, size, query_params }
					});
				},
				log_update(data) {
					return _.$ajax.post("/api/log/list_by_update", {
						data
					});
				},
				/* project */
				project_page({ group_id, page, size, name }) {
					return _.$ajax.get("/api/project/page", {
						data: { group_id, page, size, name: name || "" }
					});
				},
				getProjectByGroupId(group_id) {
					return _.$ajax.get("/api/project/list", {
						data: { group_id }
					});
				},
				getProjectById(id) {
					return _.$ajax.get("/api/project/get", {
						data: { id }
					});
				},
				copyProject(data) {
					return _.$ajax.post("/api/project/copy", {
						data
					});
				},
				project_update(data) {
					return _.$ajax.post("/api/project/up", {
						data
					});
				},
				project_add(formData) {
					return _.$ajax.post("/api/project/add", {
						data: formData
					});
				},
				project_del(data) {
					return _.$ajax.post("/api/project/del", {
						data
					});
				},
				project_add_member(data) {
					return _.$ajax.post("/api/project/add_member", {
						data
					});
				},
				project_del_member(data) {
					return _.$ajax.post("/api/project/del_member", {
						data
					});
				},
				project_change_member_role(data) {
					return _.$ajax.post("/api/project/change_member_role", {
						data
					});
				},
				projectAddFollow(data) {
					return _.$ajax.post("/api/follow/add", {
						data
					});
				},
				projectDelFollow(projectid) {
					return _.$ajax.post("/api/follow/del", {
						data: { projectid }
					});
				},
				group_del_member(data) {
					return _.$ajax.post("/api/group/del_member", {
						data
					});
				},
				groupDeleteGroup(data) {
					return _.$ajax.post("/api/group/del", {
						data
					});
				},
				groupAddMember(data) {
					return _.$ajax.post("/api/group/add_member", {
						data
					});
				},
				groupChangeMemberRole(data) {
					return _.$ajax.post("/api/group/change_member_role", {
						data
					});
				},
				groupAddGroup(data) {
					return _.$ajax.post("/api/group/add", {
						data
					});
				},
				groupUpdateGroup(data) {
					return _.$ajax.post("/api/group/up", {
						data
					});
				},
				groupMine() {
					return _.$ajax.get("/api/group/mine");
				},
				groupGetMyGroupBy(group_id) {
					let id;
					try {
						id = Number(group_id);
					} catch (error) {
						return;
					}

					return _.$ajax.get("/api/group/get", {
						data: {
							id
						}
					});
				},
				groupGetMemberListBy(group_id) {
					return _.$ajax.get("/api/group/get_member_list", {
						data: {
							id: Number(group_id)
						}
					});
				},
				/* user */
				async uploadAvatar(data) {
					return _.$ajax.post(`/api/user/upload_avatar`, {
						data
					});
				},
				async userById(id) {
					return _.$ajax.get(`/api/user/find`, {
						data: {
							id: Number(id)
						}
					});
				},
				/**
				 *  根据id删除一个用户
				 */
				async userDel(data) {
					return _.$ajax.post(`/user/del`, { data });
				},
				async userSearch(data) {
					return _.$ajax.get(`/api/user/search`, { data });
				},
				/**
				 *  修改用户密码
				 */
				async userChangePassword(data) {
					return _.$ajax.post(`/api/user/change_password`, { data });
				},
				/**
				 * 获取用户状态
				 */
				async userStatus() {
					return _.$ajax.get(`/api/user/status`);
				},
				async userLogin(data) {
					return _.$ajax.post(`/api/user/login`, {
						data
					});
				},
				async userLogout() {
					return _.$ajax.post(`/api/user/logout`);
				},
				async postNewVarifyCode(email) {
					return _.$ajax.post(`/api/user/new_varify_code`, {
						data: {
							email
						}
					});
				},
				async userReg(data) {
					return _.$ajax.post(`/api/user/reg`, {
						data
					});
				},
				/**
				 *  用户角色,只有管理员有权限修改
				 */
				async userUpdate(data) {
					return _.$ajax.post(`/api/user/update`, { data });
				},
				/**
				 *  添加项目相关的git仓库地址
				 */
				async apiCicdGitAddressAdd(data) {
					return _.$ajax.post(`/api/cicd/git_address_add`, { data });
				},
				/**
				 *  获取项目相关的git仓库地址
				 */
				async apiCicdGitAddressList(data) {
					return _.$ajax.get(`/api/cicd/git_address_list`, { data });
				},
				/**
				 *  根据git仓库地址初始化git仓库（clone repo到服务器特定地址，方便后续操作）
				 */
				async apiCicdGitInitRepo(data) {
					return _.$ajax.post(`/api/cicd/git_init_repo`, { data });
				},
				/**
				 *  添加CICD下的任务
				 */
				async apiCicdTaskRun(data) {
					return _.$ajax.post(`/api/cicd/task_run`, { data });
				},
				async apiCicdTaskAdd(data) {
					return _.$ajax.post(`/api/cicd/task_add`, { data });
				},
				async apiCicdTaskList(data) {
					return _.$ajax.get(`/api/cicd/task_list`, { data });
				},
				async apiCicdGitBranchInfo(data) {
					return _.$ajax.get(`/api/cicd/git_branch_info`, { data });
				},
				async apiCicdJobList(data) {
					return _.$ajax.get(`/api/cicd/job_list`, { data });
				},
				/**
				 *  保存桌面工作空间状态
				 */
				async saveDesktopState(data) {
					return _.$ajax.post(`/api/desktop/save_state`, { data });
				},
				/**
				 *  加载桌面工作空间状态
				 */
				async loadDesktopState() {
					return _.$ajax.get(`/api/desktop/load_state`);
				}
			};
		})();
	}

	return window._api.xspace;
}
</script>
