<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  window.spa = window.spa || {};

  window.spa.model = (function () {
    "use strict";

    var PATH_STACK_STORAGE_KEY = "VIEW_EXPLORE_PATH_STACK";
    var RESOURCE_STORAGE_KEY = "VIEW_EXPLORE_RESOURCE";

    var stateMap = {
      currentPath: [],
      currentSearchKey: "",
      resourceList: [],
      resourceMap: {},
      isLoading: false,
      pendingPromise: null,
      pendingPath: null,
      pendingSearchKey: null,
    };

    var getKeyByPath = function (path) {
      return encodeURIComponent(JSON.stringify(path || []));
    };

    var normalizeType = function (type) {
      if (type === "directory" || type === "folder") return "folder";
      if (type === "img") return "image";
      if (type === "video" || type === "flv" || type === "rmvb" || type === "realmedia") return "video";
      if (type === "audio") return "audio";
      return "other";
    };

    var toViewFile = function (item, index) {
      var path = _.isArray(item.path) ? item.path : [];
      var key = getKeyByPath(path);
      var uri = encodeURIComponent(JSON.stringify(path));
      var downloadUrl = Vue._common_utils.appendToken(
        _.$ajax.urlWrapper("/api/resource/get?uri=" + uri)
      );
      var previewUrl = Vue._common_utils.appendToken(
        _.$ajax.urlWrapper("/api/resource/get?uri=" + uri + "&preview=true")
      );
      var videoUrl = Vue._common_utils.appendToken(
        _.$ajax.urlWrapper("/api/resource/video?uri=" + uri)
      );
      var audioUrl = Vue._common_utils.appendToken(
        (window._AJAX_URL_PREFIX || "") + "/api/resource/audio?uri=" + uri
      );

      var type = normalizeType(item.type);
      var mediaUrl = downloadUrl;
      if (type === "video") mediaUrl = videoUrl;
      if (type === "audio") mediaUrl = audioUrl;

      return {
        id: key,
        key: key,
        name: item.name || "",
        type: type,
        rawType: item.type || "",
        size: item.size || "--",
        date: item.mtime || "",
        mtime: item.mtime || "",
        path: path,
        ext: item.ext || "",
        isFolder: type === "folder",
        canPreview: ["image", "video", "audio"].indexOf(type) > -1,
        url: mediaUrl,
        previewUrl: previewUrl,
        downloadUrl: downloadUrl,
        download_uri: downloadUrl,
        uri: videoUrl,
        raw: item,
        index: index,
      };
    };

    var rebuildMap = function (list) {
      var map = {};
      list.forEach(function (item) {
        map[item.id] = item;
      });
      stateMap.resourceMap = map;
    };

    var fetchFiles = async function (path, searchKey) {
      var safePath = _.isArray(path) ? path : [];
      var safeSearchKey = searchKey || "";

      // If we are already fetching the EXACT same thing, return the existing promise
      if (stateMap.isLoading && _.isEqual(safePath, stateMap.pendingPath) && safeSearchKey === stateMap.pendingSearchKey) {
        return stateMap.pendingPromise;
      }

      stateMap.pendingPath = safePath;
      stateMap.pendingSearchKey = safeSearchKey;
      stateMap.isLoading = true;

      stateMap.pendingPromise = (async function () {
        _.$loading(true);
        try {
          var res = await _api.xspace.resourceLs({
            path: safePath,
            search_key: safeSearchKey,
          });
          if (!res.errcode) {
            var list = (res.data || []).map(toViewFile);
            stateMap.currentPath = safePath;
            stateMap.currentSearchKey = safeSearchKey;
            stateMap.resourceList = list;
            rebuildMap(list);
            try {
              if (_.$lStorage) {
                _.$lStorage[PATH_STACK_STORAGE_KEY] = safePath;
                _.$lStorage[RESOURCE_STORAGE_KEY] = res.data || [];
              }
            } catch (error) {}
            return list;
          }
          
          var errmsg = String(res.errmsg || "").toLowerCase();
          if (errmsg !== "not found" && res.errcode !== 404) {
            _.$msgError(res.errmsg || "获取资源失败");
          }
          return [];
        } catch (error) {
          console.error(error);
          var errObj = error || {};
          var errCode = errObj.errcode || errObj.code || (errObj.status);
          var errMsg = String(errObj.errmsg || errObj.message || errObj || "").toLowerCase();
          
          if (errCode !== 404 && errMsg !== "not found" && errMsg.indexOf("404") === -1 && errMsg !== "[object object]") {
            _.$msgError(errObj.errmsg || errObj.message || "获取资源失败");
          }
          return [];
        } finally {
          stateMap.isLoading = false;
          stateMap.pendingPromise = null;
          stateMap.pendingPath = null;
          stateMap.pendingSearchKey = null;
          _.$loading(false);
        }
      })();

      return stateMap.pendingPromise;
    };

    var getFiles = async function (path, searchKey) {
      return fetchFiles(path, searchKey);
    };

    var getCurrentFiles = function () {
      return stateMap.resourceList.slice();
    };

    var getFileById = function (id) {
      return stateMap.resourceMap[id] || null;
    };

    var getPath = function (pathArray) {
      var safePath = _.isArray(pathArray) ? pathArray : [];
      var breadcrumb = [{ id: "root", key: "root", name: "Files", path: [] }];
      safePath.forEach(function (name, index) {
        var currentPath = safePath.slice(0, index + 1);
        breadcrumb.push({
          id: getKeyByPath(currentPath),
          key: getKeyByPath(currentPath),
          name: name,
          path: currentPath,
        });
      });
      return breadcrumb;
    };

    var getCurrentPath = function () {
      return stateMap.currentPath.slice();
    };

    var addFolder = function () {
      _.$msg("当前版本暂不支持新建目录");
    };

    var deleteFile = function () {
      _.$msg("当前版本暂不支持删除资源");
    };

    var renameFile = function () {
      _.$msg("当前版本暂不支持重命名资源");
    };

    return {
      getFiles: getFiles,
      getCurrentFiles: getCurrentFiles,
      addFolder: addFolder,
      deleteFile: deleteFile,
      renameFile: renameFile,
      getFileById: getFileById,
      getPath: getPath,
      getCurrentPath: getCurrentPath,
      getKeyByPath: getKeyByPath,
    };
  })();

  return window.spa.model;
}
</script>
