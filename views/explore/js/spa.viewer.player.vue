<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  window.spa = window.spa || {};
  window.spa.viewer = window.spa.viewer || {};

  window.spa.viewer.player = (function () {
    "use strict";

    var formatTime = function (seconds) {
      var min = Math.floor(seconds / 60);
      var sec = Math.floor(seconds % 60);
      return (
        (min < 10 ? "0" + min : min) + ":" + (sec < 10 ? "0" + sec : sec)
      );
    };

    var create = function ($media, type, handlers) {
      var $player = $(
        [
          '<div class="viewer-player">',
          '<div class="viewer-player__controls">',
          '<div class="viewer-player__progress">',
          '<span class="viewer-player__time player-time-current">00:00</span>',
          '<div class="viewer-player__progress-container">',
          '<div class="viewer-player__progress-bar">',
          '<div class="viewer-player__progress-fill"></div>',
          "</div>",
          "</div>",
          '<span class="viewer-player__time player-time-duration">00:00</span>',
          "</div>",
          '<div class="viewer-player__main-controls">',
          '<button class="viewer-player__btn player-prev">',
          spa.util.getSvg("skip-back"),
          "</button>",
          '<button class="viewer-player__btn viewer-player__btn--play player-play">',
          spa.util.getSvg("play"),
          "</button>",
          '<button class="viewer-player__btn player-next">',
          spa.util.getSvg("skip-forward"),
          "</button>",
          "</div>",
          '<div class="viewer-player__bottom-controls">',
          '<div class="viewer-player__setting">',
          '<button class="viewer-player__btn player-volume-btn">',
          spa.util.getSvg("volume-2"),
          "</button>",
          '<input type="range" class="viewer-player__volume-slider player-volume-slider" min="0" max="1" step="0.1" value="1">',
          "</div>",
          '<div class="viewer-player__setting">',
          '<span class="viewer-player__speed-label">Speed</span>',
          '<span class="viewer-player__speed-value player-speed-value">1x</span>',
          "</div>",
          '<div class="viewer-player__setting">',
          '<button class="viewer-player__btn player-fullscreen">',
          spa.util.getSvg("maximize"),
          "</button>",
          "</div>",
          "</div>",
          "</div>",
          "</div>",
        ].join("")
      );

      var media = $media[0];
      var $playBtn = $player.find(".player-play");
      var $progress = $player.find(".viewer-player__progress-container");
      var $progressFill = $player.find(".viewer-player__progress-fill");
      var $timeCurrent = $player.find(".player-time-current");
      var $timeDuration = $player.find(".player-time-duration");

      var hideTimer = null;
      var showControls = function () {
        $player.removeClass("viewer-player--hidden");
        if (hideTimer) {
          clearTimeout(hideTimer);
          hideTimer = null;
        }
        if (type === "video" && !media.paused) {
          hideTimer = setTimeout(function () {
            if (!media.paused) $player.addClass("viewer-player--hidden");
          }, 3000);
        }
      };

      media.addEventListener("ended", handlers.onNext);

      $playBtn.on("click", function () {
        if (media.paused) {
          media.play();
        } else {
          media.pause();
        }
      });

      $player.find(".player-prev").on("click", handlers.onPrev);
      $player.find(".player-next").on("click", handlers.onNext);

      var $volumeBtn = $player.find(".player-volume-btn");
      var $volumeSlider = $player.find(".player-volume-slider");
      var lastVolume = 1;

      $volumeBtn.on("click", function () {
        if (media.muted || media.volume === 0) {
          media.muted = false;
          media.volume = lastVolume || 1;
          $volumeSlider.val(media.volume);
          $volumeBtn.html(spa.util.getSvg("volume-2"));
        } else {
          lastVolume = media.volume;
          media.muted = true;
          media.volume = 0;
          $volumeSlider.val(0);
          $volumeBtn.html(spa.util.getSvg("volume-x"));
        }
      });

      $volumeSlider.on("input", function () {
        var val = parseFloat($(this).val());
        media.volume = val;
        media.muted = val === 0;
        if (val === 0) {
          $volumeBtn.html(spa.util.getSvg("volume-x"));
        } else if (val < 0.5) {
          $volumeBtn.html(spa.util.getSvg("volume-1"));
        } else {
          $volumeBtn.html(spa.util.getSvg("volume-2"));
        }
      });

      var $speedValue = $player.find(".player-speed-value");
      $player.find(".viewer-player__setting").on("click", ".player-speed-value", function() {
        var speeds = [0.5, 1.0, 1.25, 1.5, 2.0];
        var current = media.playbackRate;
        var nextIdx = (speeds.indexOf(current) + 1) % speeds.length;
        var nextSpeed = speeds[nextIdx];
        media.playbackRate = nextSpeed;
        $speedValue.text(nextSpeed + "x");
      });

      $player.find(".player-fullscreen").on("click", function () {
        if (!document.fullscreenElement) {
          if (media.requestFullscreen) {
            media.requestFullscreen();
          } else if (media.webkitRequestFullscreen) { /* Safari */
            media.webkitRequestFullscreen();
          } else if (media.msRequestFullscreen) { /* IE11 */
            media.msRequestFullscreen();
          }
        } else {
          if (document.exitFullscreen) {
            document.exitFullscreen();
          } else if (document.webkitExitFullscreen) { /* Safari */
            document.webkitExitFullscreen();
          } else if (document.msExitFullscreen) { /* IE11 */
            document.msExitFullscreen();
          }
        }
      });

      $media.on("play", function () {
        $playBtn.html(spa.util.getSvg("pause"));
        if (type === "audio") $(".viewer__record").addClass("viewer__record--playing");
        handlers.onPlay();
        showControls();
      });

      $media.on("pause", function () {
        $playBtn.html(spa.util.getSvg("play"));
        if (type === "audio") $(".viewer__record").removeClass("viewer__record--playing");
        showControls();
      });

      $media.on("timeupdate", function () {
        var percent = (media.currentTime / media.duration) * 100;
        $progressFill.css("width", percent + "%");
        $timeCurrent.text(formatTime(media.currentTime));
        $timeDuration.text(formatTime(media.duration || 0));
      });

      $media.on("click touchstart mousemove", showControls);
      $player.on("click touchstart mousemove", showControls);

      $progress.on("click", function (e) {
        if (!isFinite(media.duration)) return;
        var rect = this.getBoundingClientRect();
        var x = e.clientX - rect.left;
        var percent = x / rect.width;
        media.currentTime = percent * media.duration;
        showControls();
      });

      // Remove redundant listener or keep it with check
      $player.find(".viewer-player__progress-container").off("click").on("click", function (e) {
        if (!isFinite(media.duration)) return;
        var rect = this.getBoundingClientRect();
        media.currentTime = ((e.pageX - rect.left) / rect.width) * media.duration;
        showControls();
      });

      if (type === "video") {
        $player.find(".player-fullscreen").on("click", function () {
          if (media.requestFullscreen) media.requestFullscreen();
        });
      }

      showControls();
      return $player;
    };

    return { create: create };
  })();

  return window.spa.viewer.player;
}
</script>
