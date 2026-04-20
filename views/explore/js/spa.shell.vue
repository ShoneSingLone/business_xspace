<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  window.spa = window.spa || {};

  window.spa.shell = (function () {
    "use strict";

    var VIEW_MODE_STORAGE_KEY = "VIEW_EXPLORE_MODE";
    var SORT_STORAGE_KEY = "VIEW_EXPLORE_SORT_CONFIG";
    var PATH_STACK_STORAGE_KEY = "VIEW_EXPLORE_PATH_STACK";
    var sortOptions = [
      { label: "名称", value: "name", icon: "type" },
      { label: "类型", value: "type", icon: "folder" },
      { label: "大小", value: "size", icon: "file-text" },
      { label: "修改时间", value: "mtime", icon: "calendar" },
    ];

    var configMap = {
      main_html: [
        '<div class="spa-shell">',
        '<header class="spa-shell__header">',
        '<h1 class="spa-shell__title">Files</h1>',
        '<div class="spa-shell__actions">',
        '<button id="spa-shell-view-toggle" class="spa-shell__btn">',
        spa.util.getSvg("grid"),
        "</button>",
        '<button id="spa-shell-sort" class="spa-shell__btn">',
        spa.util.getSvg("arrow-up-down"),
        "</button>",
        '<button id="spa-shell-search-toggle" class="spa-shell__btn">',
        spa.util.getSvg("search"),
        "</button>",
        '<button id="spa-shell-add-folder" class="spa-shell__btn spa-shell__btn--primary">',
        spa.util.getSvg("plus"),
        "</button>",
        "</div>",
        "</header>",
        '<div id="spa-shell-breadcrumb-bar" class="breadcrumb-bar"></div>',
        '<div id="spa-shell-search-bar" class="search-bar hidden">',
        '<input type="text" id="spa-shell-search-input" class="search-bar__input" placeholder="Search files...">',
        "</div>",
        '<main class="file-browser" id="spa-shell-content"></main>',
        '<div id="spa-shell-modal" class="modal hidden">',
        '<div class="modal__content" id="spa-shell-modal-content"></div>',
        "</div>",
        "</div>",
      ].join(""),
    };

    var stateMap = {
      $container: null,
      currentPath: [],
      history: [],
      jqueryMap: null,
    };

    var setJqueryMap = function () {
      var $container = stateMap.$container;
      stateMap.jqueryMap = {
        $container: $container,
        $content: $container.find("#spa-shell-content"),
        $breadcrumbs: $container.find("#spa-shell-breadcrumb-bar"),
        $searchBar: $container.find("#spa-shell-search-bar"),
        $searchInput: $container.find("#spa-shell-search-input"),
        $modal: $container.find("#spa-shell-modal"),
        $modalContent: $container.find("#spa-shell-modal-content"),
      };
    };

    var toggleSearch = function () {
      stateMap.jqueryMap.$searchBar.toggleClass("hidden");
      if (!stateMap.jqueryMap.$searchBar.hasClass("hidden")) {
        stateMap.jqueryMap.$searchInput.focus();
      } else {
        stateMap.jqueryMap.$searchInput.val("");
        $(document).trigger("spa-search", "");
      }
    };

    var showModal = function (html) {
      stateMap.jqueryMap.$modalContent.html(html);
      stateMap.jqueryMap.$modal.removeClass("hidden");
      if (window.lucide) window.lucide.createIcons();
    };

    var hideModal = function () {
      stateMap.jqueryMap.$modal.addClass("hidden");
    };

    var loadSavedPathStack = function () {
      try {
        var saved = _.$lStorage && _.$lStorage[PATH_STACK_STORAGE_KEY];
        if (!saved) return [];
        if (saved === "undefined") return [];
        if (_.isArray(saved)) return saved;
        if (typeof saved === "string") {
          var parsed = JSON.parse(saved);
          if (_.isArray(parsed)) return parsed;
        }
        return [];
      } catch (error) {
        return [];
      }
    };

    var initModule = function ($container) {
      stateMap.$container = $container;
      $container.html(configMap.main_html);
      setJqueryMap();

      window.spa.browser.initModule(stateMap.jqueryMap.$content);
      window.spa.viewer.initModule($container);

      stateMap.jqueryMap.$breadcrumbs.on("click", ".breadcrumb-item", function () {
        var encodedPath = $(this).attr("data-path") || "";
        var path = [];
        try {
          path = JSON.parse(decodeURIComponent(encodedPath));
        } catch (error) {
          path = [];
        }
        stateMap.currentPath = _.isArray(path) ? path : [];
        $(document).trigger("spa-navigate", [stateMap.currentPath]);
      });

      $container.find("#spa-shell-search-toggle").on("click", toggleSearch);

      stateMap.jqueryMap.$searchInput.on("input", function () {
        $(document).trigger("spa-search", $(this).val());
      });

      $container.find("#spa-shell-add-folder").on("click", function () {
        _.$msg("当前版本暂不支持新建目录");
      });

      var $viewToggleBtn = $container.find("#spa-shell-view-toggle");
      var updateViewToggleIcon = function() {
        var mode = _.$lStorage && _.$lStorage[VIEW_MODE_STORAGE_KEY] || "list";
        $viewToggleBtn.html(spa.util.getSvg(mode === "list" ? "grid" : "list"));
      };
      
      updateViewToggleIcon();

      $viewToggleBtn.on("click", function() {
        var mode = _.$lStorage && _.$lStorage[VIEW_MODE_STORAGE_KEY] || "list";
        var nextMode = mode === "list" ? "grid" : "list";
        if (_.$lStorage) _.$lStorage[VIEW_MODE_STORAGE_KEY] = nextMode;
        updateViewToggleIcon();
        $(document).trigger("spa-view-mode-change", [nextMode]);
      });

      $container.find("#spa-shell-sort").on("click", function () {
        var getSortConfig = function () {
          try {
            var saved = _.$lStorage && _.$lStorage[SORT_STORAGE_KEY];
            if (!saved) return [];
            if (_.isArray(saved)) return saved;
            if (typeof saved === "string") return JSON.parse(saved) || [];
            return [];
          } catch (error) {
            return [];
          }
        };

        var sortConfig = getSortConfig();
        var getIndicator = function (field) {
          var idx = sortConfig.findIndex(function (x) {
            return x && x.field === field;
          });
          if (idx === -1) return "";
          var order = sortConfig[idx].order === "desc" ? "↓" : "↑";
          return " " + order + " (" + (idx + 1) + ")";
        };

        var actionsHtml = sortOptions
          .map(function (opt) {
            var indicator = getIndicator(opt.value);
            return [
              '<button class="modal__action sort-action" data-field="',
              opt.value,
              '">',
              '<span class="modal__action-label">',
              spa.util.getSvg(opt.icon, "icon"),
              " ",
              opt.label,
              "</span>",
              '<span class="modal__action-indicator">',
              indicator,
              "</span>",
              "</button>",
            ].join("");
          })
          .join("");

        showModal(
          [
            '<h3 class="modal__title">排序</h3>',
            '<div class="modal__desc">点击字段切换升/降序；最多支持两个字段组合排序</div>',
            '<div class="modal__list">',
            actionsHtml,
            "</div>",
          ].join("")
        );
      });

      stateMap.jqueryMap.$modal.on("click", ".sort-action", function () {
        var field = $(this).attr("data-field");
        $(document).trigger("spa-sort", [field]);
        hideModal();
      });

      stateMap.jqueryMap.$modal.on("click", function (e) {
        if (e.target === this) {
          hideModal();
        }
      });

      stateMap.jqueryMap.$modal.on("click", "#modal-cancel, .modal__close-btn", hideModal);

      $(document).on("spa-folder-open", function (e, folder) {
        stateMap.currentPath = folder.path || [];
        $(document).trigger("spa-navigate", [stateMap.currentPath]);
      });

      stateMap.currentPath = loadSavedPathStack();
      $(document).trigger("spa-navigate", [stateMap.currentPath]);

      if (window.lucide) {
        window.lucide.createIcons();
      }
    };

    return {
      initModule: initModule,
      showModal: showModal,
      hideModal: hideModal,
    };
  })();

  return window.spa.shell;
}
</script>
