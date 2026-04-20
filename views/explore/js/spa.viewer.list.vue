<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  window.spa = window.spa || {};
  window.spa.viewer = window.spa.viewer || {};

  window.spa.viewer.list = (function () {
    "use strict";

    var render = function (fileList, currentFile, handlers) {
      var $list = $("#spa-viewer-list");
      var $listInner = $list.find(".viewer__drawer-content");
      if ($listInner.length === 0) return;

      var html = "";
      fileList.forEach(function (file) {
        var currentClass = file.id === currentFile.id ? "viewer-drawer__item--active" : "";
        var content = "";

        if (file.type === "image") {
          content = [
            '<img src="',
            file.previewUrl || file.url,
            '" class="viewer-drawer__item-img" referrerPolicy="no-referrer">'
          ].join("");
        } else {
          content = [
            '<div class="viewer-drawer__item-icon">',
            spa.util.getSvg(file.type === "video" ? "video" : "music"),
            "</div>",
          ].join("");
        }

        html += [
          '<button class="viewer-drawer__item ',
          currentClass,
          '" data-id="',
          file.id,
          '">',
          '<div class="viewer-drawer__item-media">',
          content,
          "</div>",
          '<div class="viewer-drawer__item-info">',
          '<div class="viewer-drawer__item-name">',
          file.name,
          "</div>",
          "</div>",
          "</button>",
        ].join("");
      });
      $listInner.html(html);

      var $active = $listInner.find(".viewer-drawer__item--active");
      if ($active.length) {
        var scrollLeft =
          $active.position().left +
          $listInner.scrollLeft() -
          $listInner.width() / 2 +
          $active.width() / 2;
        $listInner.animate({ scrollLeft: scrollLeft }, 200);
      }

      $listInner
        .off("click", ".viewer-drawer__item")
        .on("click", ".viewer-drawer__item", function () {
          var id = $(this).data("id");
          var file = fileList.find(function (f) {
            return f.id === id;
          });
          if (file && handlers.onSelect) {
            handlers.onSelect(file);
          }
        });
    };

    return { render: render };
  })();

  return window.spa.viewer.list;
}
</script>
