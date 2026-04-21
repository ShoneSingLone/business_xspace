<style lang="less">
/* 
 * index.css 
 * Apple-style Glassmorphism Design using BEM
 * 纯原生标准CSS版本
 */
:root {
  --color-bg: #f5f5f7;
  --color-text: #1d1d1f;
  --color-text-secondary: #86868b;
  --color-primary: #0071e3;
  --color-white: #ffffff;
  --glass-bg: rgba(255, 255, 255, 0.7);
  --glass-border: rgba(255, 255, 255, 0.3);
  --glass-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.07);
  --blur: 20px;
  --radius-lg: 20px;
  --radius-md: 12px;
  --transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  -webkit-tap-highlight-color: transparent;
}

html,
body,
#spa {
  height: 100%;
  width: 100%;
  overflow: hidden;
}

body {
  font-family:
    -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial,
    sans-serif;
  background-color: var(--color-bg);
  color: var(--color-text);
  line-height: 1.5;
}

/* BEM: spa-shell */
.spa-shell {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
}

.spa-shell__header {
  background: var(--glass-bg);
  backdrop-filter: blur(var(--blur));
  -webkit-backdrop-filter: blur(var(--blur));
  border-bottom: 1px solid var(--glass-border);
  padding: 16px 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  z-index: 50;
}

.spa-shell__title {
  font-size: 20px;
  font-weight: 600;
  letter-spacing: -0.02em;
}

.spa-shell__actions {
  display: flex;
  gap: 8px;
}

.spa-shell__btn {
  background: transparent;
  border: none;
  padding: 8px;
  border-radius: 50%;
  cursor: pointer;
  color: var(--color-text);
  display: flex;
  align-items: center;
  justify-content: center;
  transition: var(--transition);
}

.spa-shell__btn:hover {
  background: rgba(0, 0, 0, 0.05);
}

.spa-shell__btn--primary {
  background: var(--color-primary);
  color: var(--color-white);
}

.spa-shell__btn--primary:hover {
  background: #0077ed;
}

.spa-shell__btn--viewer {
  color: #fff !important;
}

.spa-shell__btn--viewer:hover {
  background: rgba(255, 255, 255, 0.1) !important;
}

/* BEM: breadcrumb-bar */
.breadcrumb-bar {
  background: var(--glass-bg);
  backdrop-filter: blur(var(--blur));
  -webkit-backdrop-filter: blur(var(--blur));
  border-bottom: 1px solid var(--glass-border);
  padding: 8px 20px;
  display: flex;
  align-items: center;
  gap: 4px;
  overflow-x: auto;
  white-space: nowrap;
  scrollbar-width: none;
}

.breadcrumb-bar::-webkit-scrollbar {
  display: none;
}

.breadcrumb-item {
  background: transparent;
  border: none;
  font-size: 14px;
  color: var(--color-text-secondary);
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 6px;
  transition: var(--transition);
}

.breadcrumb-item:hover {
  background: rgba(0, 0, 0, 0.05);
  color: var(--color-text);
}

.breadcrumb-separator {
  color: var(--color-text-secondary);
  opacity: 0.5;
  font-size: 12px;
}

/* BEM: file-browser */
.file-browser {
  flex: 1;
  overflow-y: auto;
  min-height: 0;
  -webkit-overflow-scrolling: touch;
  padding-bottom: 80px;
}

.file-list {
  display: flex;
  flex-direction: column;
}

.file-item {
  display: flex;
  align-items: center;
  padding: 12px 20px;
  cursor: pointer;
  transition: var(--transition);
  border-bottom: 1px solid rgba(0, 0, 0, 0.03);
  background: var(--color-white);
  border-radius: var(--radius-lg);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.03);
  animation: fadeIn 0.4s ease-out forwards;
  position: relative;
}

.file-item__preview {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: inherit;
}

.file-item__icon-wrap i {
  width: 24px;
  height: 24px;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.file-item:hover {
  background: rgba(0, 0, 0, 0.02);
}

.file-item__icon-wrap {
  width: 48px;
  height: 48px;
  background: var(--color-white);
  border-radius: var(--radius-md);
  display: flex;
  flex-shrink: 0;
  align-items: center;
  justify-content: center;
  margin-right: 16px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

.file-item__info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.file-item__name {
  font-weight: 500;
  font-size: 15px;
  line-height: 1.4;
  margin-bottom: 4px;
  color: var(--color-text);
  word-break: break-all;
  white-space: normal !important; /* 确保在所有模式下都能换行 */
}

.file-item__meta {
  font-size: 12px;
  line-height: 1.4;
  color: var(--color-text-secondary);
}

.file-item__action {
  padding: 8px;
  border-radius: 50%;
  color: var(--color-text-secondary);
  background: transparent;
  border: none;
  cursor: pointer;
  flex-shrink: 0;
  margin-left: 16px;
}

.file-browser__filter {
  padding: 16px;
  display: flex;
  gap: 8px;
  overflow-x: auto;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  scrollbar-width: none;
}

.file-browser__filter::-webkit-scrollbar {
  display: none;
}

.filter-btn {
  padding: 6px 16px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 500;
  background: rgba(0, 0, 0, 0.05);
  color: var(--color-text);
  border: none;
  cursor: pointer;
  transition: var(--transition);
  white-space: nowrap;
}

.filter-btn:hover {
  background: rgba(0, 0, 0, 0.1);
}

.filter-btn.active {
  background: var(--color-primary);
  color: var(--color-white);
}

.file-browser__list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 16px;
}

.file-browser__empty {
  padding: 48px;
  text-align: center;
  opacity: 0.6;
}

.search-bar {
  padding: 12px 20px;
  background: var(--color-white);
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.search-bar__input {
  width: 100%;
  padding: 10px 16px;
  border-radius: 10px;
  border: none;
  background: rgba(0, 0, 0, 0.05);
  outline: none;
  font-size: 16px;
}

/* BEM: modal */
.modal {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 100;
  padding: 20px;
}

.modal__content {
  background: var(--color-white);
  border-radius: var(--radius-lg);
  width: 100%;
  max-width: 340px;
  padding: 24px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
}

.modal__title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 16px;
  text-align: center;
}

.modal__input {
  width: 100%;
  padding: 12px;
  border: 1px solid rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  margin-bottom: 20px;
  outline: none;
}

.modal__actions {
  display: flex;
  gap: 12px;
}

.modal__btn {
  flex: 1;
  padding: 12px;
  border-radius: 12px;
  border: none;
  font-weight: 600;
  cursor: pointer;
}

.modal__btn--cancel {
  background: rgba(0, 0, 0, 0.05);
}

.modal__desc {
  font-size: 12px;
  opacity: 0.6;
  text-align: center;
  margin-bottom: 12px;
}

.modal__list {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.modal__action {
  width: 100%;
  text-align: left;
  padding: 12px;
  background: transparent;
  border: none;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  cursor: pointer;
  transition: var(--transition);
}

.modal__action:hover {
  background: rgba(0, 0, 0, 0.05);
}

.modal__action-label {
  display: flex;
  align-items: center;
  gap: 12px;
}

.modal__action-indicator {
  font-size: 12px;
  opacity: 0.6;
  font-family: monospace;
}

.modal__info-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  font-size: 14px;
}

.modal__info-item {
  display: flex;
  justify-content: space-between;
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  padding-bottom: 8px;
}

.modal__info-item--block {
  flex-direction: column;
  gap: 4px;
}

.modal__info-label {
  opacity: 0.6;
}

.modal__info-value {
  font-weight: 500;
}

.modal__info-value--break {
  word-break: break-all;
  font-size: 13px;
  opacity: 0.8;
}

.modal__close-btn {
  width: 100%;
  margin-top: 24px;
  padding: 12px;
  background: rgba(0, 0, 0, 0.05);
  border: none;
  border-radius: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: var(--transition);
}

.modal__close-btn:hover {
  background: rgba(0, 0, 0, 0.1);
}

.modal__title--truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* BEM: viewer */
.viewer {
  position: fixed;
  inset: 0;
  background: #000;
  z-index: 200;
  display: flex;
  flex-direction: column;
  color: #fff;
}

.viewer__header {
  padding: 16px 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: linear-gradient(to bottom, rgba(0, 0, 0, 0.8), transparent);
  position: absolute;
  top: 0;
  width: 100%;
  z-index: 50;
}

.viewer__content {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  overflow: hidden;
}

.viewer__header-left {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
  min-width: 0;
}

.viewer__header-right {
  display: flex;
  align-items: center;
  gap: 8px;
}

.viewer__title-wrapper {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 0;
  flex: 1;
}

.viewer__title {
  font-size: 16px;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.viewer__index {
  font-size: 12px;
  opacity: 0.5;
  font-family: monospace;
  white-space: nowrap;
  flex-shrink: 0;
}

.viewer__fallback {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  color: rgba(255, 255, 255, 0.6);
}

.viewer__fallback-icon {
  width: 64px;
  height: 64px;
}

.viewer__fallback-text {
  font-size: 14px;
}

.viewer__btn {
  background: transparent;
  border: none;
  padding: 8px;
  border-radius: 50%;
  cursor: pointer;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: var(--transition);
}

.viewer__btn:hover {
  background: rgba(255, 255, 255, 0.1);
}

.viewer__btn--primary {
  padding: 8px 24px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 999px;
  color: #fff;
}

.viewer__btn--primary:hover {
  background: rgba(255, 255, 255, 0.2);
}
.viewer-image {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: grab;
}

.viewer-image:active {
  cursor: grabbing;
}

.viewer-image__img {
  max-width: 90%;
  max-height: 90%;
  object-fit: contain;
  transition: transform 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
}

.viewer__media {
  display: block;
  max-width: 90%;
  max-height: 80%;
  width: auto;
  height: auto;
  object-fit: contain;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
  border-radius: 12px;
}

.viewer-seek-tip {
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  padding: 10px 14px;
  background: rgba(0, 0, 0, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 12px;
  color: #fff;
  font-size: 14px;
  font-weight: 500;
  z-index: 70;
  pointer-events: none;
  white-space: nowrap;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}

@media (max-width: 768px) {
  .viewer-seek-tip {
    font-size: 13px;
    padding: 8px 12px;
  }
}

@media (max-width: 768px) {
  .viewer--video .viewer__media {
    width: 100%;
    max-width: 100%;
    max-height: calc(100vh - 180px);
  }
}

.viewer-image__controls {
  position: absolute;
  bottom: 100px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  padding: 8px 16px;
  border-radius: 30px;
  display: flex;
  align-items: center;
  gap: 16px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  z-index: 40;
}

.viewer-image__btn {
  background: transparent;
  border: none;
  padding: 8px;
  border-radius: 50%;
  cursor: pointer;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: var(--transition);
}

.viewer-image__btn:hover {
  background: rgba(255, 255, 255, 0.1);
}

.viewer-image__level {
  font-size: 14px;
  font-weight: 500;
  min-width: 40px;
  text-align: center;
}

.viewer-image__divider {
  width: 1px;
  height: 16px;
  background: rgba(255, 255, 255, 0.2);
}

/* Audio Specific */
.viewer__audio-player {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 40px;
  width: 100%;
  max-width: 400px;
}

.viewer__record-wrap {
  position: relative;
  width: 280px;
  height: 280px;
}

.viewer__record {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: #111;
  background-image: radial-gradient(
    circle at center,
    #333 0%,
    #111 5%,
    #333 10%,
    #111 15%,
    #333 20%,
    #111 25%,
    #333 30%,
    #111 35%,
    #333 40%,
    #111 45%,
    #333 50%,
    #111 55%,
    #333 60%,
    #111 65%,
    #333 70%,
    #111 75%,
    #333 80%,
    #111 85%,
    #333 90%,
    #111 95%,
    #333 100%
  );
  border: 8px solid #222;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  animation: spin 10s linear infinite;
  animation-play-state: paused;
}

.viewer__record--playing {
  animation-play-state: running;
}

.viewer__record-center {
  width: 80px;
  height: 80px;
  background: var(--color-primary);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
  border: 4px solid #111;
}

.viewer__audio-controls {
  width: 100%;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  padding: 24px;
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.viewer__nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.3);
  color: #fff;
  padding: 28px;
  border-radius: 50%;
  cursor: pointer;
  backdrop-filter: blur(20px);
  transition: var(--transition);
  z-index: 30;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

.viewer__nav:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.25);
  transform: translateY(-50%) scale(1.1);
  border-color: rgba(255, 255, 255, 0.5);
}

.viewer__nav:disabled {
  opacity: 0.2;
  cursor: not-allowed;
  background: rgba(255, 255, 255, 0.02);
}

.viewer__nav i {
  width: 28px;
  height: 28px;
}

/* Navigation visibility */
.viewer__nav {
  display: none !important;
}

.viewer-image {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  touch-action: none;
}

.viewer-image__img {
  max-width: 90%;
  max-height: 90%;
  object-fit: contain;
  transition: transform 0.1s ease-out;
  cursor: grab;
  user-select: none;
  -webkit-user-drag: none;
}

.viewer-image__img:active {
  cursor: grabbing;
}

.viewer-image__img.zoom-mode-1-1 {
  max-width: none;
  max-height: none;
}

.viewer__index {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.5);
  font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
  background: rgba(255, 255, 255, 0.1);
  padding: 2px 8px;
  border-radius: 4px;
}

.viewer__drawer {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  width: 320px;
  background: rgba(20, 20, 20, 0.95);
  backdrop-filter: blur(20px);
  border-left: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  flex-direction: column;
  z-index: 100;
  transform: translateX(0);
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: -10px 0 30px rgba(0, 0, 0, 0.5);
}

.viewer__drawer.hidden {
  transform: translateX(100%);
  display: flex !important; /* Override Tailwind hidden to keep animation */
  visibility: hidden;
}

.viewer__drawer-header {
  padding: 16px 20px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.viewer__drawer-title {
  font-size: 16px;
  font-weight: 500;
  color: white;
}

.viewer__drawer-content {
  flex: 1;
  overflow-y: auto;
  padding: 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.viewer-drawer__item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px;
  border-radius: 12px;
  background: transparent;
  border: 1px solid transparent;
  cursor: pointer;
  transition: all 0.2s;
  text-align: left;
  color: white;
}

.viewer-drawer__item:hover {
  background: rgba(255, 255, 255, 0.05);
}

.viewer-drawer__item--active {
  background: rgba(255, 255, 255, 0.1);
  border-color: rgba(255, 255, 255, 0.2);
}

.viewer-drawer__item-media {
  width: 48px;
  height: 48px;
  border-radius: 8px;
  overflow: hidden;
  background: rgba(0, 0, 0, 0.5);
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.viewer-drawer__item-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.viewer-drawer__item-info {
  flex: 1;
  min-width: 0;
}

.viewer-drawer__item-name {
  font-size: 13px;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  opacity: 0.9;
}

.viewer-drawer__item--active .viewer-drawer__item-name {
  color: var(--color-primary, #0077ed);
  opacity: 1;
}

@media (max-width: 768px) {
  .viewer__drawer {
    width: 100%;
  }
}

.no-scrollbar::-webkit-scrollbar {
  display: none;
}
.no-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
}

.viewer__nav--prev {
  left: 30px;
}
.viewer__nav--next {
  right: 30px;
}

.viewer__footer {
  padding: 20px 40px;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.8), transparent);
  position: absolute;
  bottom: 0;
  width: 100%;
  z-index: 50;
}

.viewer__footer-content {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 24px;
}

.viewer__speed-control {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(255, 255, 255, 0.05);
  padding: 4px 12px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.viewer-player {
  width: 90%;
  max-width: 500px;
  background: rgba(20, 20, 20, 0.7);
  backdrop-filter: blur(40px);
  -webkit-backdrop-filter: blur(40px);
  border-radius: 32px;
  padding: 24px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  display: flex;
  flex-direction: column;
  gap: 20px;
  user-select: none;
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
  z-index: 60;
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.viewer-player--hidden {
  opacity: 0;
  pointer-events: none;
  transform: translateX(-50%) translateY(10px);
}

.viewer-player__media {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 200px;
  border-radius: 16px;
  overflow: hidden;
  background: rgba(0, 0, 0, 0.5);
  margin-bottom: 12px;
}

.viewer-player__video {
  width: 100%;
  height: 100%;
  object-fit: contain;
}

.viewer-player__controls {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
}

.viewer-player__progress {
  display: flex;
  align-items: center;
  gap: 12px;
  width: 100%;
}

.viewer-player__progress-container {
  flex: 1;
  height: 4px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 2px;
  position: relative;
  cursor: pointer;
}

.viewer-player__progress-container:hover {
  height: 6px;
}

.viewer-player__progress-bar {
  height: 100%;
  background: transparent;
  width: 100%;
}

.viewer-player__progress-fill {
  height: 100%;
  background: #fff;
  border-radius: 2px;
  width: 0%;
  position: relative;
}

.viewer-player__progress-fill::after {
  content: "";
  position: absolute;
  right: -6px;
  top: 50%;
  transform: translateY(-50%);
  width: 12px;
  height: 12px;
  background: #fff;
  border-radius: 50%;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  opacity: 0;
  transition: opacity 0.2s;
}

.viewer-player__progress-container:hover .viewer-player__progress-fill::after {
  opacity: 1;
}

.viewer-player__time {
  font-size: 11px;
  font-family: monospace;
  opacity: 0.6;
  min-width: 40px;
  text-align: center;
}

.viewer-player__main-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 24px;
}

.viewer-player__bottom-controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 12px;
  border-top: 1px solid rgba(255, 255, 255, 0.05);
}

.viewer-player__setting {
  display: flex;
  align-items: center;
  gap: 8px;
  background: rgba(255, 255, 255, 0.05);
  padding: 6px 12px;
  border-radius: 8px;
}

.viewer-player__btn {
  background: transparent;
  border: none;
  color: white;
  padding: 8px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: transform 0.2s, background 0.2s;
  opacity: 0.7;
}

.viewer-player__btn:hover {
  opacity: 1;
  transform: scale(1.1);
}

.viewer-player__btn--play {
  width: 56px;
  height: 56px;
  background: var(--color-primary, #0077ed);
  opacity: 1;
  box-shadow: 0 8px 20px rgba(0, 113, 227, 0.3);
}

.viewer-player__btn--play:hover {
  background: #0077ed;
  transform: scale(1.05);
}

.viewer-player__volume-slider {
  width: 60px;
  cursor: pointer;
}

.viewer-player__speed-label,
.viewer-player__volume-label {
  font-size: 10px;
  text-transform: uppercase;
  opacity: 0.5;
}

.viewer-player__speed-value {
  font-size: 13px;
  font-weight: 500;
}

.viewer-list__item {
  flex: 0 0 64px;
  height: 64px;
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  border: 2px solid transparent;
  background: transparent;
  transition: all 0.3s;
  opacity: 0.5;
  display: flex;
  align-items: center;
  justify-content: center;
}

.viewer-list__item:hover {
  opacity: 1;
}

.viewer-list__item--active {
  border-color: #fff;
  opacity: 1;
  transform: scale(1.1);
}

.viewer-list__item-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 8px;
}

.viewer-list__item-icon {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
}

.hidden {
  display: none !important;
}

/* Custom Scrollbar */
::-webkit-scrollbar {
  width: 6px;
}
::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.1);
  border-radius: 10px;
}

/* --- Grid / Waterfall View Styles --- */
.file-browser--grid .file-browser__list {
  display: grid !important;
  grid-template-columns: repeat(2, 1fr) !important;
  gap: 12px !important;
  padding: 12px !important;
}

@media (min-width: 600px) {
  .file-browser--grid .file-browser__list {
    grid-template-columns: repeat(3, 1fr) !important;
  }
}

@media (min-width: 900px) {
  .file-browser--grid .file-browser__list {
    grid-template-columns: repeat(4, 1fr) !important;
  }
}

@media (min-width: 1200px) {
  .file-browser--grid .file-browser__list {
    grid-template-columns: repeat(5, 1fr) !important;
  }
}

.file-browser--grid .file-item {
  display: flex !important;
  flex-direction: column !important;
  margin-bottom: 0 !important; /* Grid manages gap */
  padding: 8px !important;
  text-align: center !important;
  gap: 6px !important;
  height: auto !important;
  align-self: start !important; /* 关键：确保项高度自适应且不对齐行高 */
  border: 1px solid rgba(0, 0, 0, 0.05) !important;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.02) !important;
}

.file-browser--grid .file-item__icon-wrap {
  margin-right: 0 !important;
  width: 100% !important;
  height: auto !important;
  min-height: 80px !important;
  background: #f8f8f8 !important;
}

.file-browser--grid .file-item__preview {
  width: 100% !important;
  height: auto !important;
  display: block !important;
  object-fit: contain !important;
  border-radius: 4px !important;
}

.file-browser--grid .file-item__info {
  width: 100% !important;
}

.file-browser--grid .file-item__name {
  white-space: normal !important;
  text-align: center !important;
  word-break: break-all !important;
}

.file-browser--grid .file-item__action {
  position: absolute !important;
  top: 4px !important;
  right: 4px !important;
  background: rgba(255, 255, 255, 0.8) !important;
  opacity: 0 !important;
}

.file-browser--grid .file-item:hover .file-item__action {
  opacity: 1 !important;
}
.viewer-image__bottom-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
  background: rgba(20, 20, 20, 0.7);
  backdrop-filter: blur(40px);
  -webkit-backdrop-filter: blur(40px);
  padding: 12px 24px;
  border-radius: 32px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  position: absolute;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 60;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.viewer-image__bottom-controls--hidden {
  opacity: 0;
  pointer-events: none;
  transform: translateX(-50%) translateY(20px);
}

.viewer-image__group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.viewer-image__divider {
  width: 1px;
  height: 24px;
  background: rgba(255, 255, 255, 0.15);
  margin: 0 4px;
}

.viewer-image__btn {
  background: transparent;
  border: none;
  color: #fff;
  padding: 8px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
  opacity: 0.7;
}

.viewer-image__btn:hover {
  opacity: 1;
  background: rgba(255, 255, 255, 0.1);
  transform: scale(1.1);
}

.viewer-image__btn--active {
  color: var(--color-primary);
  opacity: 1;
  background: rgba(0, 113, 227, 0.1);
}

.viewer-image__level {
  font-size: 13px;
  font-weight: 500;
  min-width: 45px;
  text-align: center;
  color: rgba(255, 255, 255, 0.9);
}

.viewer-image__interval {
  width: 45px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 6px;
  color: #fff;
  padding: 4px 6px;
  font-size: 13px;
  text-align: center;
  outline: none;
}

.viewer-image__unit {
  font-size: 12px;
  opacity: 0.5;
}

.viewer-image__interval::-webkit-inner-spin-button {
  display: none;
}

@media (max-width: 600px) {
  .viewer-image__bottom-controls {
    padding: 8px 16px;
    gap: 8px;
    bottom: 20px;
    width: 95%;
    max-width: 450px;
    flex-wrap: nowrap;
    overflow-x: auto;
    justify-content: flex-start;
  }
  .viewer-image__divider {
    height: 16px;
  }
}
</style>
<template>
	    <div id="spa"></div>
</template>
<script lang="ts">
export default async function ({ PRIVATE_GLOBAL }) {
	await _.$importVue("@/views/explore/js/spa.util.vue");
	await _.$importVue("@/views/explore/js/spa.model.vue");
	await _.$importVue("@/views/explore/js/spa.viewer.list.vue");
	await _.$importVue("@/views/explore/js/spa.viewer.image.vue");
	await _.$importVue("@/views/explore/js/spa.viewer.player.vue");
	await _.$importVue("@/views/explore/js/spa.viewer.vue");
	await _.$importVue("@/views/explore/js/spa.browser.vue");
	await _.$importVue("@/views/explore/js/spa.shell.vue");
	await _.$importVue("@/views/explore/js/spa.vue");

	return {
		mounted(){
			var $ = window.$ || window.jQuery;
			window.spa.initModule($(this.$el));
		}
	};
}
</script>
