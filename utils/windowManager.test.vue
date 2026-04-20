export default async function ({ PRIVATE_GLOBAL }) {
  /**
   * windowManager 单元测试脚本
   * 覆盖核心逻辑、边界情况、状态流转
   */
  const windowManager = await _.$importVue('@/utils/windowManager.vue');
  const results = [];
  let testCount = 0;
  let passCount = 0;

  const it = (desc, fn) => {
    testCount++;
    try {
      fn();
      passCount++;
      results.push({ desc, status: 'pass' });
      console.log(`✅ [PASS] ${desc}`);
    } catch (e) {
      results.push({ desc, status: 'fail', error: e.message });
      console.error(`❌ [FAIL] ${desc}: ${e.message}`);
    }
  };

  const expect = (actual) => ({
    toBe(expected) {
      if (actual !== expected) throw new Error(`Expected ${expected}, but got ${actual}`);
    },
    toBeTruthy() {
      if (!actual) throw new Error(`Expected truthy, but got ${actual}`);
    },
    toBeFalsy() {
      if (actual) throw new Error(`Expected falsy, but got ${actual}`);
    },
    toHaveLength(len) {
      if (actual.length !== len) throw new Error(`Expected length ${len}, but got ${actual.length}`);
    }
  });

  // 清理初始状态
  windowManager.state.windows = [];
  windowManager.state.activeId = null;

  // --- 测试用例 ---

  it('1. 基础打开窗口功能', () => {
    const id = windowManager.open({ appId: 'app1', title: 'App 1', component: { name: 'Comp1' } });
    expect(windowManager.state.windows).toHaveLength(1);
    expect(windowManager.state.activeId).toBe(id);
    expect(windowManager.state.windows[0].appId).toBe('app1');
    expect(windowManager.state.windows[0].isFocused).toBeTruthy();
  });

  it('2. 单例窗口逻辑 (singleton: true)', () => {
    const id1 = windowManager.state.windows[0].id;
    const id2 = windowManager.open({ appId: 'app1', title: 'App 1 Reopen', component: {}, singleton: true });
    expect(id1).toBe(id2);
    expect(windowManager.state.windows).toHaveLength(1);
  });

  it('3. 多窗口并发打开与层级调度', () => {
    const id2 = windowManager.open({ appId: 'app2', title: 'App 2', component: {} });
    const id3 = windowManager.open({ appId: 'app3', title: 'App 3', component: {} });
    expect(windowManager.state.windows).toHaveLength(3);
    expect(windowManager.state.activeId).toBe(id3);
    
    // 检查 zIndex 递增
    const win2 = windowManager.state.windows.find(w => w.id === id2);
    const win3 = windowManager.state.windows.find(w => w.id === id3);
    expect(win3.zIndex > win2.zIndex).toBeTruthy();
  });

  it('4. 聚焦切换 (Focus)', () => {
    const windows = windowManager.state.windows;
    const id1 = windows[0].id;
    const id2 = windows[1].id;
    
    windowManager.focus(id1);
    expect(windowManager.state.activeId).toBe(id1);
    expect(windows.find(w => w.id === id1).isFocused).toBeTruthy();
    expect(windows.find(w => w.id === id2).isFocused).toBeFalsy();
  });

  it('5. 最小化与焦点自动转移 (Minimize & AutoFocus)', () => {
    const id1 = windowManager.state.activeId;
    const idNext = windowManager.state.windows[2].id; // 应该是 id3，因为它是之前 zIndex 最高的
    
    windowManager.minimize(id1);
    expect(windowManager.state.windows.find(w => w.id === id1).isMinimized).toBeTruthy();
    expect(windowManager.state.activeId).toBe(idNext);
  });

  it('6. 恢复最小化窗口 (Restore)', () => {
    const windows = windowManager.state.windows;
    const id1 = windows.find(w => w.isMinimized).id;
    
    windowManager.restore(id1);
    expect(windowManager.state.windows.find(w => w.id === id1).isMinimized).toBeFalsy();
    expect(windowManager.state.activeId).toBe(id1);
  });

  it('7. 最大化与恢复之前的尺寸 (Maximize & Restore Rect)', () => {
    const win = windowManager.state.windows[0];
    const initialRect = { x: win.x, y: win.y, width: win.width, height: win.height };
    
    windowManager.toggleMaximize(win.id);
    expect(win.isMaximized).toBeTruthy();
    
    windowManager.toggleMaximize(win.id);
    expect(win.isMaximized).toBeFalsy();
    expect(win.x).toBe(initialRect.x);
    expect(win.width).toBe(initialRect.width);
  });

  it('8. 边界情况：尝试操作不存在的窗口', () => {
    const initialCount = windowManager.state.windows.length;
    windowManager.close('non-existent-id');
    expect(windowManager.state.windows).toHaveLength(initialCount);
    
    windowManager.focus('non-existent-id');
    // activeId 不应改变
    expect(windowManager.state.activeId).toBeTruthy();
  });

  it('9. 销毁窗口与资源清理 (Close)', () => {
    const idToClose = windowManager.state.activeId;
    windowManager.close(idToClose);
    expect(windowManager.state.windows.find(w => w.id === idToClose)).toBeFalsy();
    expect(windowManager.state.activeId !== idToClose).toBeTruthy();
  });

  it('10. 边界测试：关闭所有窗口', () => {
    const ids = windowManager.state.windows.map(w => w.id);
    ids.forEach(id => windowManager.close(id));
    expect(windowManager.state.windows).toHaveLength(0);
    expect(windowManager.state.activeId).toBe(null);
  });

  console.log(`\n--- 测试结果 ---`);
  console.log(`总计: ${testCount}, 通过: ${passCount}, 失败: ${testCount - passCount}`);
  
  return {
    results,
    summary: { total: testCount, passed: passCount, failed: testCount - passCount }
  };
}
