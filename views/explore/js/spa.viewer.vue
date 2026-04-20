<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  window.spa = window.spa || {};

  var viewer = window.spa.viewer || {};

  (function () {
    "use strict";

    var stateMap = {
      $container: null,
      currentFile: null,
      fileList: [],
      playbackRate: 1.0,
      isLooping: false,
      imageScale: 1.0,
      isSlideshow: false,
      slideshowTimer: null,
    };

    var configMap = {
      main_html: [
        '<div id="spa-viewer" class="viewer hidden">',
        '<div class="viewer__header">',
        '<div class="viewer__header-left">',
        '<button id="spa-viewer-close" class="spa-shell__btn spa-shell__btn--viewer">',
        spa.util.getSvg("x"),
        "</button>",
        '<div class="viewer__title-wrapper">',
        '<h2 class="viewer__title" id="spa-viewer-title"></h2>',
        '<div class="viewer__index"></div>',
        "</div>",
        "</div>",
        '<div class="viewer__header-right">',
        '<button id="spa-viewer-toggle-list" class="spa-shell__btn spa-shell__btn--viewer" title="Show List">',
        spa.util.getSvg("list"),
        "</button>",
        '<button id="spa-viewer-download" class="spa-shell__btn spa-shell__btn--viewer">',
        spa.util.getSvg("download"),
        "</button>",
        "</div>",
        "</div>",
        '<div id="spa-viewer-content" class="viewer__content">',
        '<button id="spa-viewer-prev" class="viewer__nav viewer__nav--prev">',
        spa.util.getSvg("chevron-left"),
        "</button>",
        '<button id="spa-viewer-next" class="viewer__nav viewer__nav--next">',
        spa.util.getSvg("chevron-right"),
        "</button>",
        "</div>",
        '<div id="spa-viewer-list" class="viewer__drawer hidden">',
        '<div class="viewer__drawer-header">',
        '<h3 class="viewer__drawer-title">播放列表</h3>',
        '<button id="spa-viewer-close-list" class="viewer__btn">',
        spa.util.getSvg("x"),
        "</button>",
        "</div>",
        '<div class="viewer__drawer-content no-scrollbar"></div>',
        "</div>",
        '<footer id="spa-viewer-footer" class="viewer__footer hidden"></footer>',
        "</div>",
      ].join(""),
    };

    var clamp = function (value, min, max) {
      return Math.max(min, Math.min(max, value));
    };

    var formatTime = function (seconds) {
      var safe = Number(seconds) || 0;
      var min = Math.floor(safe / 60);
      var sec = Math.floor(safe % 60);
      return (min < 10 ? "0" + min : "" + min) + ":" + (sec < 10 ? "0" + sec : "" + sec);
    };

    var setupSwipeSeek = function ($media, $content) {
      var media = $media && $media[0];
      if (!media) return;
      if (!("ontouchstart" in window)) return;

      var tipId = "spa-viewer-seek-tip";
      $content.find("#" + tipId).remove();
      var $tip = $('<div id="' + tipId + '" class="viewer-seek-tip hidden"></div>');
      $content.append($tip);

      var startX = 0;
      var startY = 0;
      var startTime = 0;
      var isActive = false;
      var hasMoved = false;
      var targetTime = 0;

      var threshold = 8;

      var onStart = function (e) {
        if (!e.touches || e.touches.length !== 1) return;
        var t = e.touches[0];
        startX = t.clientX;
        startY = t.clientY;
        startTime = media.currentTime || 0;
        isActive = false;
        hasMoved = false;
        targetTime = startTime;
        $tip.addClass("hidden");
      };

      var onMove = function (e) {
        if (!e.touches || e.touches.length !== 1) return;
        var t = e.touches[0];
        var dx = t.clientX - startX;
        var dy = t.clientY - startY;

        if (!isActive) {
          if (Math.abs(dx) < threshold) return;
          if (Math.abs(dx) < Math.abs(dy)) return;
          isActive = true;
        }

        e.preventDefault();

        var duration = media.duration || 0;
        if (!duration) return;

        var rect = media.getBoundingClientRect();
        var width = rect.width || window.innerWidth || 1;
        var ratio = dx / width;
        var offsetSeconds = ratio * duration * 0.6;
        targetTime = clamp(startTime + offsetSeconds, 0, duration);
        hasMoved = true;

        var label = offsetSeconds >= 0 ? "快进" : "快退";
        $tip.removeClass("hidden");
        $tip.text(label + " " + formatTime(startTime) + " → " + formatTime(targetTime));
      };

      var onEnd = function () {
        if (hasMoved && isFinite(targetTime) && isFinite(media.duration)) {
          media.currentTime = targetTime;
        }
        $tip.addClass("hidden");
      };

      media.style.touchAction = "pan-y";
      media.addEventListener("touchstart", onStart, { passive: true });
      media.addEventListener("touchmove", onMove, { passive: false });
      media.addEventListener("touchend", onEnd, { passive: true });
      media.addEventListener("touchcancel", onEnd, { passive: true });
    };

    var nextFile, prevFile, stopSlideshow;

    var updateNavigation = function () {
      var index = stateMap.fileList.indexOf(stateMap.currentFile);
      var $index = $(".viewer__index");
      $index.text(index + 1 + " / " + stateMap.fileList.length);

      spa.viewer.list.render(stateMap.fileList, stateMap.currentFile, {
        onPlay: function () {
          if (stateMap.isSlideshow) stopSlideshow();
        },
        onSelect: function (file) {
          stateMap.currentFile = file;
          $("#spa-viewer-title").text(file.name);
          renderPreview(file);
          // Auto close drawer on select in mobile
          if (window.innerWidth < 768) {
            $("#spa-viewer-list").addClass("hidden");
          }
        },
      });
    };

    var cleanupContent = function () {
      var $content = $("#spa-viewer-content");
      $content.find("video, audio").each(function () {
        if (this._flvPlayer) {
          try {
            this._flvPlayer.destroy();
          } catch (e) {}
          this._flvPlayer = null;
        }
        this.pause();
        this.src = "";
        this.load();
        $(this).remove();
      });

      $content
        .find(
          ".viewer__media, .viewer__media--image, .viewer-image, .viewer-player, .viewer__audio-player, .viewer-image__controls, .viewer__fallback"
        )
        .remove();
    };

    var renderPreview = function (file) {
      var $content = $("#spa-viewer-content");
      var $footer = $("#spa-viewer-footer");
      var $viewer = $("#spa-viewer");

      cleanupContent();

      $footer.addClass("hidden");
      $viewer.removeClass("viewer--image viewer--video viewer--audio");
      $viewer.addClass("viewer--" + file.type);

      var handlers = {
        onNext: function () {
          stopSlideshow();
          nextFile();
        },
        onPrev: function () {
          stopSlideshow();
          prevFile();
        },
        onPlay: function () {
          if (stateMap.isSlideshow) stopSlideshow();
        },
      };

      switch (file.type) {
        case "image":
          spa.viewer.image.render(file, $content, handlers);
          break;
        case "video":
          var isRmvb = file.name.toLowerCase().endsWith(".rmvb") || file.name.toLowerCase().endsWith(".rm") || file.rawType === "rmvb";
          if (isRmvb) {
            $content.append(
              [
                '<div class="viewer__fallback">',
                '<div class="viewer__fallback-icon">',
                spa.util.getSvg("video", "icon-large"),
                "</div>",
                '<div class="viewer__fallback-text">RMVB 格式暂不支持网页直接播放</div>',
                '<button id="spa-viewer-download" class="viewer__btn viewer__btn--primary">',
                "下载文件",
                "</button>",
                "</div>",
              ].join("")
            );
            return;
          }

          $footer.removeClass("hidden");
          var $video = $('<video class="viewer__media max-w-[90%] max-h-[80%] object-contain shadow-2xl" playsinline></video>');
          var $videoPlayer = spa.viewer.player.create($video, "video", handlers);
          $content.append($video).append($videoPlayer);
          setupSwipeSeek($video, $content);

          var isFlv = file.name.toLowerCase().endsWith(".flv") || file.rawType === "flv" || file.rawType === "video/x-flv";
          
          var playVideo = function() {
            var videoEl = $video[0];
            var playPromise = videoEl.play();
            if (playPromise !== undefined) {
              playPromise.catch(function(error) {
                console.log("Auto-play blocked, muted and retrying...", error);
                videoEl.muted = true;
                videoEl.play();
                _.$msg("由于浏览器政策，视频已静音自动播放，请手动开启声音");
              });
            }
          };

          if (isFlv) {
            var loadFlvJs = function() {
              return new Promise(function(resolve, reject) {
                if (window.flvjs) return resolve(window.flvjs);
                var scriptId = "flv-js-script";
                if (document.getElementById(scriptId)) {
                  var timer = setInterval(function() {
                    if (window.flvjs) {
                      clearInterval(timer);
                      resolve(window.flvjs);
                    }
                  }, 100);
                  return;
                }
                var script = document.createElement("script");
                script.id = scriptId;
                script.src = "https://cdn.bootcdn.net/ajax/libs/flv.js/1.6.2/flv.min.js";
                script.onload = function() { resolve(window.flvjs); };
                script.onerror = function() { reject(new Error("Failed to load flv.js")); };
                document.head.appendChild(script);
              });
            };

            loadFlvJs().then(function(flvjs) {
              if (flvjs.isSupported()) {
                var flvPlayer = flvjs.createPlayer({
                  type: "flv",
                  url: file.url,
                  isLive: false,
                  hasAudio: true,
                  hasVideo: true
                }, {
                  enableWorker: true,
                  stashInitialSize: 128,
                  enableStashBuffer: false
                });
                
                flvPlayer.on(flvjs.Events.ERROR, function(errType, errDetail) {
                  console.error("FLV Player Error:", errType, errDetail);
                  _.$msgError("FLV 播放出错: " + errType);
                });

                flvPlayer.attachMediaElement($video[0]);
                flvPlayer.load();
                playVideo();
                $video[0]._flvPlayer = flvPlayer;
              } else {
                _.$msgError("您的浏览器不支持 MSE，无法播放 FLV 视频");
                $video.attr("src", file.url);
                playVideo();
              }
            }).catch(function(err) {
              _.$msgError("加载 flv.js 失败");
              console.error(err);
            });
          } else {
            $video.attr("src", file.url);
            playVideo();
          }
          break;
        case "audio":
          $footer.removeClass("hidden");
          var $audioPlayer = $(
            [
              '<div class="viewer__audio-player">',
              '<div class="viewer__record-wrap">',
              '<div class="viewer__record">',
              '<div class="viewer__record-center">',
              spa.util.getSvg("music", "w-8 h-8"),
              "</div>",
              "</div>",
              "</div>",
              "</div>",
            ].join("")
          );

          var $audio = $("<audio autoplay playsinline></audio>");
          $audio.attr("src", file.url);
          var $audioControls = spa.viewer.player.create($audio, "audio", handlers);
          $audioPlayer.append($audioControls);
          $content.append($audioPlayer).append($audio);

          var audioEl = $audio[0];
          var audioPlayPromise = audioEl.play();
          if (audioPlayPromise !== undefined) {
            audioPlayPromise.catch(function(error) {
              console.log("Auto-play prevented by browser:", error);
            });
          }
          break;
        default:
          $content.append(
            [
              '<div class="viewer__fallback">',
              '<div class="viewer__fallback-icon">',
              spa.util.getSvg(file.type === "video" ? "video" : "music", "icon-large"),
              "</div>",
              '<div class="viewer__fallback-text">Preview not available</div>',
              '<button id="spa-viewer-download" class="viewer__btn viewer__btn--primary">',
              "Download File",
              "</button>",
              "</div>",
            ].join("")
          );
      }

      updateNavigation();
      if (window.lucide) {
        window.lucide.createIcons();
      }
    };

    var initModule = function ($container) {
      stateMap.$container = $container;
      $container.append(configMap.main_html);

      var $viewer = $("#spa-viewer");
      var $title = $("#spa-viewer-title");

      $("#spa-viewer-close").on("click", function () {
        $viewer.addClass("hidden");
        cleanupContent();
        $("#spa-viewer-list").addClass("hidden");
        stopSlideshow();
      });

      $("#spa-viewer-list-toggle").on("click", function () {
        $("#spa-viewer-list").toggleClass("hidden");
        $(this).toggleClass("spa-shell__btn--primary");
        if (!$("#spa-viewer-list").hasClass("hidden")) {
          updateNavigation();
        }
      });

      nextFile = function () {
        var index = stateMap.fileList.indexOf(stateMap.currentFile);
        if (index < stateMap.fileList.length - 1) {
          stateMap.currentFile = stateMap.fileList[index + 1];
          $title.text(stateMap.currentFile.name);
          renderPreview(stateMap.currentFile);
          return true;
        }
        return false;
      };

      prevFile = function () {
        var index = stateMap.fileList.indexOf(stateMap.currentFile);
        if (index > 0) {
          stateMap.currentFile = stateMap.fileList[index - 1];
          $title.text(stateMap.currentFile.name);
          renderPreview(stateMap.currentFile);
          return true;
        }
        return false;
      };

      var isSlideshow = function () {
        return stateMap.isSlideshow;
      };

      var startSlideshow = function (interval) {
        if (stateMap.slideshowTimer) clearInterval(stateMap.slideshowTimer);
        stateMap.isSlideshow = true;
        $("#spa-viewer-slideshow").addClass("spa-shell__btn--primary");
        stateMap.slideshowTimer = setInterval(function () {
          if (!nextFile()) stopSlideshow();
        }, interval || 3000);
      };

      stopSlideshow = function () {
        stateMap.isSlideshow = false;
        $("#spa-viewer-slideshow").removeClass("spa-shell__btn--primary");
        if (stateMap.slideshowTimer) {
          clearInterval(stateMap.slideshowTimer);
          stateMap.slideshowTimer = null;
        }
      };

      viewer.isSlideshow = isSlideshow;
      viewer.startSlideshow = startSlideshow;
      viewer.stopSlideshow = stopSlideshow;

      $("#spa-viewer-prev").on("click", function () {
        stopSlideshow();
        prevFile();
      });

      $("#spa-viewer-next").on("click", function () {
        stopSlideshow();
        nextFile();
      });

      var updateImageTransform = function () {
        $(".viewer__media--image").css("transform", "scale(" + stateMap.imageScale + ")");
        $(".zoom-level").text(Math.round(stateMap.imageScale * 100) + "%");
      };

      $container.on("click", ".zoom-in", function () {
        stateMap.imageScale = Math.min(stateMap.imageScale + 0.2, 3.0);
        updateImageTransform();
      });

      $container.on("click", ".zoom-out", function () {
        stateMap.imageScale = Math.max(stateMap.imageScale - 0.2, 0.5);
        updateImageTransform();
      });

      $container.on("click", ".zoom-reset", function () {
        stateMap.imageScale = 1.0;
        updateImageTransform();
      });

      $("#spa-viewer-info").on("click", function () {
        var file = stateMap.currentFile;
        var infoHtml = [
          '<h3 class="modal__title">File Info</h3>',
          '<div class="modal__info-list">',
          '<div class="modal__info-item">',
          '<span class="modal__info-label">Name</span>',
          '<span class="modal__info-value">',
          file.name,
          "</span>",
          "</div>",
          '<div class="modal__info-item">',
          '<span class="modal__info-label">Type</span>',
          '<span class="modal__info-value">',
          file.type,
          "</span>",
          "</div>",
          '<div class="modal__info-item">',
          '<span class="modal__info-label">Size</span>',
          '<span class="modal__info-value">',
          file.size,
          "</span>",
          "</div>",
          '<div class="modal__info-item">',
          '<span class="modal__info-label">Date</span>',
          '<span class="modal__info-value">',
          file.date,
          "</span>",
          "</div>",
          "</div>",
          '<button class="modal__close-btn">关闭</button>',
        ].join("");
        $(document).trigger("spa-show-modal", [infoHtml]);
      });

      $("#spa-viewer-download").on("click", function () {
        if (stateMap.currentFile && (stateMap.currentFile.downloadUrl || stateMap.currentFile.url)) {
          var link = document.createElement("a");
          link.href = stateMap.currentFile.downloadUrl || stateMap.currentFile.url;
          link.download = stateMap.currentFile.name;
          document.body.appendChild(link);
          link.click();
          document.body.removeChild(link);
        }
      });

      $("#spa-viewer-slideshow").on("click", function () {
        if (stateMap.isSlideshow) {
          stopSlideshow();
        } else {
          startSlideshow();
        }
      });

      $("#spa-viewer-toggle-list").on("click", function () {
        $("#spa-viewer-list").removeClass("hidden");
      });

      $("#spa-viewer-close-list").on("click", function () {
        $("#spa-viewer-list").addClass("hidden");
      });

      $(document).on("spa-file-open", function (e, data) {
        stateMap.currentFile = data.file;
        stateMap.fileList = (data.list || []).filter(function (f) {
          return f.type === data.file.type;
        });
        $title.text(stateMap.currentFile.name);
        renderPreview(stateMap.currentFile);
        $viewer.removeClass("hidden");
      });

      $(document).on("spa-file-menu", function (e, id) {
        var file = spa.model.getFileById(id);
        if (!file) return;

        var menuHtml = [
          '<h3 class="modal__title modal__title--truncate">',
          file.name,
          "</h3>",
          '<div class="modal__info-list">',
          '<div class="modal__info-item">',
          '<span class="modal__info-label">类型</span>',
          '<span class="modal__info-value">',
          file.rawType || file.type,
          "</span></div>",
          '<div class="modal__info-item">',
          '<span class="modal__info-label">大小</span>',
          '<span class="modal__info-value">',
          file.size || "--",
          "</span></div>",
          '<div class="modal__info-item">',
          '<span class="modal__info-label">mtime</span>',
          '<span class="modal__info-value">',
          file.mtime || "--",
          "</span></div>",
          '<div class="modal__info-item modal__info-item--block">',
          '<span class="modal__info-label">path（数组）：</span>',
          '<div class="modal__info-value modal__info-value--break">',
          JSON.stringify(file.path || []),
          "</div></div>",
          '<div class="modal__info-item modal__info-item--block">',
          '<span class="modal__info-label">path（连接）：</span>',
          '<div class="modal__info-value modal__info-value--break">',
          (file.path || []).join("/"),
          "</div></div>",
          "</div>",
          '<button class="modal__close-btn">关闭</button>',
        ].join("");

        $(document).trigger("spa-show-modal", [menuHtml]);
      });

      $(document).on("spa-show-modal", function (e, html) {
        spa.shell.showModal(html);
      });

      $(document).off("click", ".menu-action");
      $(document).off("click", "#rename-confirm");
    };

    viewer.initModule = initModule;
  })();

  window.spa.viewer = viewer;
  return viewer;
}
</script>
