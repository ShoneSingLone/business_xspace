<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  window.spa = window.spa || {};

  window.spa.browser = (function () {
    "use strict";

    var SORT_STORAGE_KEY = "VIEW_EXPLORE_SORT_CONFIG";
    var PATH_STACK_STORAGE_KEY = "VIEW_EXPLORE_PATH_STACK";
    var defaultSortConfig = [
      { field: "type", order: "asc" },
      { field: "name", order: "asc" },
    ];

    var stateMap = {
      $container: null,
      currentPath: [],
      filter: "all",
      searchQuery: "",
      renderToken: 0,
      sortConfig: defaultSortConfig.slice(),
      isLoading: false,
    };

    var loadSortConfig = function () {
      try {
        var saved = _.$lStorage && _.$lStorage[SORT_STORAGE_KEY];
        if (!saved) return defaultSortConfig.slice();
        if (_.isArray(saved)) return saved;
        if (typeof saved === "string") {
          var parsed = JSON.parse(saved);
          if (_.isArray(parsed)) return parsed;
        }
        return defaultSortConfig.slice();
      } catch (error) {
        return defaultSortConfig.slice();
      }
    };

    var saveSortConfig = function () {
      try {
        if (_.$lStorage) _.$lStorage[SORT_STORAGE_KEY] = stateMap.sortConfig;
      } catch (error) {}
    };

    var normalizeSortConfig = function (sortConfig) {
      if (!_.isArray(sortConfig) || sortConfig.length === 0) return defaultSortConfig.slice();
      var filtered = sortConfig
        .filter(function (x) {
          return x && x.field;
        })
        .map(function (x) {
          return { field: x.field, order: x.order === "desc" ? "desc" : "asc" };
        });
      if (filtered.length === 0) return defaultSortConfig.slice();
      return filtered.slice(0, 2);
    };

    var toggleSortField = function (field) {
      var idx = stateMap.sortConfig.findIndex(function (x) {
        return x.field === field;
      });

      if (idx === -1) {
        stateMap.sortConfig.push({ field: field, order: "asc" });
      } else {
        stateMap.sortConfig[idx].order = stateMap.sortConfig[idx].order === "asc" ? "desc" : "asc";
        if (stateMap.sortConfig.length > 1 && idx !== 0) {
          var moved = stateMap.sortConfig.splice(idx, 1)[0];
          stateMap.sortConfig.unshift(moved);
        }
      }

      if (stateMap.sortConfig.length > 2) stateMap.sortConfig.pop();
      saveSortConfig();
    };

    var VIEW_MODE_STORAGE_KEY = "VIEW_EXPLORE_MODE";

    var getIcon = function (type, file) {
      var fallback = spa.util.getSvg(type || "file", "icon-md");
      if (file && (type === "image" || type === "video")) {
        var previewUrl = file.previewUrl;
        return '<img class="file-item__preview" src="' + previewUrl + '" loading="lazy" onerror="$(this).hide().next().show();">' + 
               '<div style="display:none">' + fallback + '</div>';
      }
      return fallback;
    };

    var getSortValue = function (file, field) {
      if (field === "type") return file.rawType || file.type || "";
      if (field === "mtime") return file.mtime || file.date || "";
      return file[field] || "";
    };

    var compareNaturalName = function (aName, bName) {
      var aParts = String(aName || "").split(/(\d+)/);
      var bParts = String(bName || "").split(/(\d+)/);
      for (var i = 0; i < Math.min(aParts.length, bParts.length); i++) {
        var aPart = aParts[i];
        var bPart = bParts[i];
        if (/^\d+$/.test(aPart) && /^\d+$/.test(bPart)) {
          var aInt = parseInt(aPart, 10);
          var bInt = parseInt(bPart, 10);
          if (aInt !== bInt) return aInt - bInt;
        } else {
          var cmp = aPart.localeCompare(bPart);
          if (cmp !== 0) return cmp;
        }
      }
      return aParts.length - bParts.length;
    };

    var compareBySortConfig = function (a, b) {
      for (var i = 0; i < stateMap.sortConfig.length; i++) {
        var field = stateMap.sortConfig[i].field;
        var order = stateMap.sortConfig[i].order;
        var compareResult = 0;

        if (field === "name") {
          compareResult = compareNaturalName(a.name, b.name);
          compareResult = order === "asc" ? compareResult : -compareResult;
        } else {
          var aValue = getSortValue(a, field);
          var bValue = getSortValue(b, field);
          if (aValue !== bValue) {
            compareResult = aValue > bValue ? 1 : -1;
            compareResult = order === "asc" ? compareResult : -compareResult;
          }
        }

        if (compareResult !== 0) return compareResult;
      }
      return 0;
    };

    var render = async function (targetMode) {
      var mode = targetMode || (_.$lStorage && _.$lStorage[VIEW_MODE_STORAGE_KEY]) || "list";
      stateMap.$container.removeClass("file-browser--list file-browser--grid").addClass("file-browser--" + mode);

      var token = ++stateMap.renderToken;
      var path = spa.model.getPath(stateMap.currentPath);
      var breadcrumbsHtml = "";

      path.forEach(function (folder, index) {
        if (index > 0) {
          breadcrumbsHtml += '<span class="breadcrumb-separator">/</span>';
        }
        breadcrumbsHtml += [
          '<button class="breadcrumb-item" data-path="',
          encodeURIComponent(JSON.stringify(folder.path || [])),
          '">',
          folder.name,
          "</button>",
        ].join("");
      });

      $("#spa-shell-breadcrumb-bar").html(breadcrumbsHtml);

      var showLoading = function() {
        var skeletonHtml = '<div class="file-browser__list">';
        for (var i = 0; i < 5; i++) {
          skeletonHtml += [
            '<div class="file-item file-item--skeleton" style="opacity: 0.5; pointer-events: none;">',
            '<div class="file-item__icon-wrap" style="background: #eee;"></div>',
            '<div class="file-item__info">',
            '<div class="file-item__name" style="width: 60%; height: 16px; background: #eee; border-radius: 4px; margin-bottom: 8px;"></div>',
            '<div class="file-item__meta" style="width: 40%; height: 12px; background: #eee; border-radius: 4px;"></div>',
            "</div>",
            "</div>",
          ].join("");
        }
        skeletonHtml += "</div>";
        stateMap.$container.html(skeletonHtml);
      };

      stateMap.isLoading = true;
      showLoading();

      var files = [];
      try {
        files = await spa.model.getFiles(stateMap.currentPath, stateMap.searchQuery);
      } finally {
        stateMap.isLoading = false;
      }
      
      if (token !== stateMap.renderToken) return;

      if (stateMap.filter !== "all") {
        files = files.filter(function (f) {
          return f.type === stateMap.filter;
        });
      }

      files = files.slice().sort(compareBySortConfig);

      var html = "";

      if (stateMap.searchQuery) {
        html += [
          '<div class="file-browser__filter">',
          '<button class="filter-btn" data-type="all">All</button>',
          '<button class="filter-btn" data-type="image">Images</button>',
          '<button class="filter-btn" data-type="video">Videos</button>',
          '<button class="filter-btn" data-type="audio">Audio</button>',
          '<button class="filter-btn" data-type="file">Documents</button>',
          "</div>",
        ].join("");
      }

      html += '<div class="file-browser__list">';

      if (files.length === 0) {
        html += '<div class="file-browser__empty">No files found</div>';
      } else {
        files.forEach(function (file) {
          html += [
          '<div class="file-item" data-id="',
            file.id,
            '">',
            '<div class="file-item__icon-wrap">',
            getIcon(file.type, file),
            "</div>",
            '<div class="file-item__info">',
            '<div class="file-item__name">',
            file.name,
            "</div>",
            '<div class="file-item__meta">',
            file.date,
            " • ",
            file.size,
            "</div>",
            "</div>",
            '<button class="file-item__action file-menu-btn" data-id="',
            file.id,
            '">',
            spa.util.getSvg("more-vertical"),
            "</button>",
            "</div>",
          ].join("");
        });
      }

      html += "</div>";
      stateMap.$container.html(html);

      if (window.lucide) window.lucide.createIcons();
    };

    var initModule = function ($container) {
      stateMap.$container = $container;
      stateMap.sortConfig = normalizeSortConfig(loadSortConfig());

      $container.on("click", ".file-item", function (e) {
        if ($(e.target).closest(".file-menu-btn").length) return;

        var id = String($(this).attr("data-id"));
        var file = spa.model.getFileById(id);
        if (!file) return;
        var files = spa.model.getCurrentFiles();

        if (file.type === "folder") {
          $(document).trigger("spa-folder-open", [file]);
        } else {
          $(document).trigger("spa-file-open", [{
            file: file,
            list: files.filter(function (f) {
              return f.type !== "folder" && f.type === file.type;
            }),
          }]);
        }
      });

      $container.on("click", ".filter-btn", function () {
        stateMap.filter = $(this).attr("data-type") || "all";
        render();
      });

      $container.on("click", ".file-menu-btn", function (e) {
        e.stopPropagation();
        var id = String($(this).attr("data-id"));
        $(document).trigger("spa-file-menu", [id]);
      });

      $(document).on("spa-navigate", function (e, path) {
        stateMap.currentPath = _.isArray(path) ? path : [];
        stateMap.searchQuery = "";
        stateMap.filter = "all";
        try {
          if (_.$lStorage) _.$lStorage[PATH_STACK_STORAGE_KEY] = stateMap.currentPath;
        } catch (error) {}
        render();
      });

      $(document).on("spa-search", function (e, query) {
        stateMap.searchQuery = query || "";
        stateMap.filter = "all";
        render();
      });

      $(document).on("spa-sort", function (e, sort) {
        if (sort) toggleSortField(sort);
        render();
      });

      $(document).on("spa-model-update", render);

      $(document).on("spa-view-mode-change", function(e, mode) {
        stateMap.$container.removeClass("file-browser--list file-browser--grid").addClass("file-browser--" + mode);
        render(mode);
      });

      $(document).on("spa-add-folder", function (e, data) {
        spa.model.addFolder(data.name, data.parentId);
      });
    };

    return { initModule: initModule };
  })();

  return window.spa.browser;
}
</script>
