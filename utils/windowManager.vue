export default async function ({ PRIVATE_GLOBAL }) {
  /**
   * WindowManager - 集中式窗口管理器 (单例模式)
   * 负责所有窗口的生命周期管理、层级调度和事件分发
   */
  const { defineComponent, reactive, markRaw } = Vue;

  // 单例状态存储
  const state = reactive({
    windows: [], // 窗口实例列表
    activeId: null, // 当前获得焦点的窗口 ID
    nextZIndex: 1000, // 初始层级
    config: {
      minWidth: 400,
      minHeight: 300,
      defaultWidth: 800,
      defaultHeight: 600,
      offsetStep: 30, // 新窗口偏移步长
    }
  });

  // 辅助函数：生成唯一 ID
  const generateId = () => Math.random().toString(36).substring(2, 9);

  // 窗口管理器核心方法
  const windowManager = {
    state,

    /**
     * 打开新窗口或聚焦现有窗口
     * @param {Object} options 窗口配置
     * @returns {String} 窗口 ID
     */
    open(options) {
      const { appId, title, component, props = {}, data = {}, singleton = false } = options;

      // 如果是单例窗口，检查是否已存在
      if (singleton) {
        const existing = state.windows.find(w => w.appId === appId);
        if (existing) {
          this.focus(existing.id);
          if (existing.isMinimized) this.restore(existing.id);
          return existing.id;
        }
      }

      // 计算新窗口位置 (阶梯偏移)
      const count = state.windows.length;
      const x = 100 + (count % 10) * state.config.offsetStep;
      const y = 60 + (count % 10) * state.config.offsetStep;

      const id = generateId();
      const newWindow = reactive({
        id,
        appId,
        title: title || 'New Window',
        component: markRaw(component), // 标记为非响应式以优化性能
        props,
        data,
        zIndex: state.nextZIndex++,
        isMinimized: false,
        isMaximized: false,
        isFocused: true,
        x,
        y,
        width: options.width || state.config.defaultWidth,
        height: options.height || state.config.defaultHeight,
        prevRect: null, // 用于恢复最大化前的状态
      });

      state.windows.push(newWindow);
      this.focus(id);
      
      return id;
    },

    /**
     * 关闭窗口
     * @param {String} id 窗口 ID
     */
    close(id) {
      const index = state.windows.findIndex(w => w.id === id);
      if (index === -1) return;

      state.windows.splice(index, 1);
      
      // 如果关闭的是当前激活窗口，激活剩余的最顶层窗口
      if (state.activeId === id) {
        this._autoFocusNext();
      }
    },

    /**
     * 聚焦窗口
     * @param {String} id 窗口 ID
     */
    focus(id) {
      const win = state.windows.find(w => w.id === id);
      if (!win || win.isMinimized) return;

      // 如果已经聚焦，且 zIndex 也是最高的，则不做操作
      if (state.activeId === id && win.zIndex === state.nextZIndex - 1) return;

      state.activeId = id;
      win.zIndex = state.nextZIndex++;
      
      // 更新所有窗口的聚焦状态
      state.windows.forEach(w => {
        w.isFocused = w.id === id;
      });
    },

    /**
     * 最小化窗口
     * @param {String} id 窗口 ID
     */
    minimize(id) {
      const win = state.windows.find(w => w.id === id);
      if (!win) return;

      win.isMinimized = true;
      win.isFocused = false;

      if (state.activeId === id) {
        this._autoFocusNext();
      }
    },

    /**
     * 恢复/取消最小化
     * @param {String} id 窗口 ID
     */
    restore(id) {
      const win = state.windows.find(w => w.id === id);
      if (!win) return;

      win.isMinimized = false;
      this.focus(id);
    },

    /**
     * 最大化/取消最大化
     * @param {String} id 窗口 ID
     */
    toggleMaximize(id) {
      const win = state.windows.find(w => w.id === id);
      if (!win) return;

      if (!win.isMaximized) {
        // 保存当前位置大小
        win.prevRect = { x: win.x, y: win.y, width: win.width, height: win.height };
        win.isMaximized = true;
      } else {
        // 恢复之前位置大小
        if (win.prevRect) {
          const { x, y, width, height } = win.prevRect;
          win.x = x;
          win.y = y;
          win.width = width;
          win.height = height;
        }
        win.isMaximized = false;
      }
      this.focus(id);
    },

    /**
     * 内部方法：自动聚焦下一个最合适的窗口
     */
    _autoFocusNext() {
      const visibleWindows = state.windows
        .filter(w => !w.isMinimized)
        .sort((a, b) => b.zIndex - a.zIndex);

      if (visibleWindows.length > 0) {
        this.focus(visibleWindows[0].id);
      } else {
        state.activeId = null;
      }
    },

    /**
     * 更新窗口位置/大小
     */
    updateRect(id, rect) {
      const win = state.windows.find(w => w.id === id);
      if (!win || win.isMaximized) return;

      if (rect.x !== undefined) win.x = rect.x;
      if (rect.y !== undefined) win.y = rect.y;
      if (rect.width !== undefined) win.width = Math.max(rect.width, state.config.minWidth);
      if (rect.height !== undefined) win.height = Math.max(rect.height, state.config.minHeight);
    }
  };

  return windowManager;
}
