<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
  window.spa = window.spa || {};
  window.spa.viewer = window.spa.viewer || {};

  window.spa.viewer.image = (function () {
    "use strict";

    var stateMap = {
      imageScale: 1.0,
    };

    var updateImageTransform = function () {
      $(".viewer__media--image").css(
        "transform",
        "scale(" + stateMap.imageScale + ")"
      );
      $(".zoom-level").text(Math.round(stateMap.imageScale * 100) + "%");
    };

    var render = function (file, $content, handlers) {
      var $imgContainer = $('<div class="viewer-image"></div>');
      var $img = $(
        '<img src="' +
          file.url +
          '" class="viewer-image__img" referrerPolicy="no-referrer">'
      );
      $imgContainer.append($img);
      $content.append($imgContainer);

      var $footer = $("#spa-viewer-footer");
      $footer.empty().removeClass("hidden");

      var $controls = $(
        [
          '<div class="viewer-image__bottom-controls">',
          '<div class="viewer-image__group">',
          '<button class="viewer-image__btn player-prev" title="上一个">',
          spa.util.getSvg("chevron-left"),
          "</button>",
          '<button class="viewer-image__btn player-next" title="下一个">',
          spa.util.getSvg("chevron-right"),
          "</button>",
          "</div>",
          '<div class="viewer-image__divider"></div>',
          // Zoom group
          '<div class="viewer-image__group zoom-group">',
          '<button class="viewer-image__btn zoom-out" title="缩小">',
          spa.util.getSvg("minus"),
          "</button>",
          '<span class="viewer-image__level zoom-level">100%</span>',
          '<button class="viewer-image__btn zoom-in" title="放大">',
          spa.util.getSvg("plus"),
          "</button>",
          '<div class="viewer-image__divider"></div>',
          '<button class="viewer-image__btn zoom-toggle" title="切换1:1/合适">',
          spa.util.getSvg("maximize"),
          "</button>",
          "</div>",
          // Slideshow interval group (hidden by default)
          '<div class="viewer-image__group interval-group hidden">',
          '<input type="number" class="viewer-image__interval slideshow-interval" value="3" min="1" max="60" title="间隔(秒)">',
          '<span class="viewer-image__unit">s</span>',
          "</div>",
          '<div class="viewer-image__divider"></div>',
          '<div class="viewer-image__group">',
          '<button class="viewer-image__btn slideshow-toggle" title="幻灯片">',
          spa.util.getSvg("play"),
          "</button>",
          "</div>",
          "</div>",
        ].join("")
      );

      $footer.append($controls);

      var hideTimer = null;
      var showControls = function() {
        $controls.removeClass("viewer-image__bottom-controls--hidden");
        if (hideTimer) clearTimeout(hideTimer);
        hideTimer = setTimeout(function() {
          $controls.addClass("viewer-image__bottom-controls--hidden");
        }, 3000);
      };

      $imgContainer.on("click", function() {
        showControls();
      });

      // Show initially
      showControls();

      $controls.find(".player-prev").on("click", function(e) {
        e.stopPropagation();
        handlers.onPrev();
        showControls();
      });
      $controls.find(".player-next").on("click", function(e) {
        e.stopPropagation();
        handlers.onNext();
        showControls();
      });

      var is1to1 = false;
      var posX = 0, posY = 0;
      var lastMouseX = 0, lastMouseY = 0;
      var isDragging = false;

      stateMap.imageScale = 1.0;
      
      var updateImageTransform = function () {
        $img.css({
          "transform": "translate(" + posX + "px, " + posY + "px) scale(" + stateMap.imageScale + ")"
        });
        $controls.find(".zoom-level").text(Math.round(stateMap.imageScale * 100) + "%");
      };

      $controls.find(".zoom-in").on("click", function (e) {
        e.stopPropagation();
        stateMap.imageScale = Math.min(stateMap.imageScale + 0.2, 5.0);
        updateImageTransform();
        showControls();
      });
      $controls.find(".zoom-out").on("click", function (e) {
        e.stopPropagation();
        stateMap.imageScale = Math.max(stateMap.imageScale - 0.2, 0.1);
        updateImageTransform();
        showControls();
      });

      $controls.find(".zoom-toggle").on("click", function (e) {
        e.stopPropagation();
        is1to1 = !is1to1;
        posX = 0; posY = 0;
        if (is1to1) {
          $img.addClass("zoom-mode-1-1");
          $(this).html(spa.util.getSvg("minimize")).addClass("viewer-image__btn--active");
        } else {
          $img.removeClass("zoom-mode-1-1");
          $(this).html(spa.util.getSvg("maximize")).removeClass("viewer-image__btn--active");
        }
        stateMap.imageScale = 1.0;
        updateImageTransform();
        showControls();
      });

      // Mouse/Touch Drag Support
      var startDrag = function(clientX, clientY) {
        isDragging = true;
        lastMouseX = clientX;
        lastMouseY = clientY;
        showControls();
      };

      var moveDrag = function(clientX, clientY) {
        if (!isDragging) return;
        var dx = clientX - lastMouseX;
        var dy = clientY - lastMouseY;
        posX += dx;
        posY += dy;
        lastMouseX = clientX;
        lastMouseY = clientY;
        updateImageTransform();
        showControls();
      };

      var stopDrag = function() {
        isDragging = false;
      };

      $imgContainer.on("mousedown", function(e) {
        e.preventDefault();
        startDrag(e.clientX, e.clientY);
      });

      $(window).on("mousemove.viewer_img", function(e) {
        moveDrag(e.clientX, e.clientY);
      }).on("mouseup.viewer_img", function() {
        stopDrag();
      });

      $imgContainer.on("touchstart", function(e) {
        if (e.originalEvent.touches.length === 1) {
          var touch = e.originalEvent.touches[0];
          startDrag(touch.clientX, touch.clientY);
        }
      });

      $imgContainer.on("touchmove", function(e) {
        if (e.originalEvent.touches.length === 1) {
          var touch = e.originalEvent.touches[0];
          moveDrag(touch.clientX, touch.clientY);
        }
      });

      $imgContainer.on("touchend touchcancel", function() {
        stopDrag();
      });

      // Slideshow logic
      var $slideshowBtn = $controls.find(".slideshow-toggle");
      var updateSlideshowUI = function() {
        var isRunning = spa.viewer.isSlideshow();
        $slideshowBtn.html(spa.util.getSvg(isRunning ? "pause" : "play"));
        $slideshowBtn.toggleClass("viewer-image__btn--active", isRunning);
        
        // Toggle groups visibility
        if (isRunning) {
          $controls.find(".zoom-group").addClass("hidden");
          $controls.find(".interval-group").removeClass("hidden");
        } else {
          $controls.find(".zoom-group").removeClass("hidden");
          $controls.find(".interval-group").addClass("hidden");
        }
      };

      $slideshowBtn.on("click", function(e) {
        e.stopPropagation();
        var interval = parseInt($controls.find(".slideshow-interval").val()) * 1000;
        if (spa.viewer.isSlideshow()) {
          spa.viewer.stopSlideshow();
        } else {
          spa.viewer.startSlideshow(interval);
        }
        updateSlideshowUI();
        showControls();
      });

      $controls.find(".slideshow-interval").on("click change", function(e) {
        e.stopPropagation();
        showControls();
      });

      updateSlideshowUI();
    };

    return { render: render };
  })();

  return window.spa.viewer.image;
}
</script>
