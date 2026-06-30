# 设计：继续打开会话时自动收集并关闭

## 元信息
- 项目：UniAddQQ（qq-session-tool.html）
- 类型：brownfield（现有功能增强）
- 日期：2026-07-01
- 状态：已确认，待实现

## 背景与动机

当前群发工作流是半自动的：用户点击"继续打开会话"打开下一批窗口后，需要**手动**点击"收集失败"按钮（把仍打开的窗口对应的 QQ 累加进失败列表）和"关闭所有窗口"按钮（关闭上一批窗口）。每批之间两次手动操作，影响效率。

用户希望在点击"继续打开会话"时，自动完成这两步清理，且默认不开启（保守，不改变现有用户行为）。

## 平台限制（不可突破）

- `wpa.qq.com` 只能打开聊天窗口，**无法自动发送消息**
- 无给陌生人自动发消息的 web 协议
- 因此"效率"优化限于**打开/管理窗口流程**，不涉及自动发送

## 目标

当"自动收集并关闭"开关开启时，用户点击"继续打开会话"→ 确认弹窗后，在打开下一批窗口**之前**，自动执行收集失败 + 关闭窗口。开关默认关闭。

## 功能行为

### 触发点
`confirmBroadcast()` 函数开头（用户在确认弹窗点确认之后、`openBatch()` 之前）。

### 执行顺序
1. **收集失败**：调用现有 `collectFailedQQs()` —— 把仍打开的窗口对应的 QQ 累加进 `failedQQs[]`，更新失败列表 UI 和队列状态
2. **关闭窗口**：调用现有 `closeAllOpenedWindows()` —— 关闭所有跟踪的窗口，清空 `openedWindows[]`
3. **打开下一批**：照常 `openBatch(thisBatch)`

### 守卫条件（三个全满足才执行）
- `autoCollectCloseEnabled` —— 开关开启
- `bcIndex > 0` —— 非首批（首批无历史窗口可清理）
- `openedWindows.length > 0` —— 确实有窗口可清理（避免无意义调用）

### 开关关闭时（默认）
行为完全不变，用户仍需手动点"收集失败"和"关闭所有窗口"。

## UI 变更

在"群发设置"卡片现有的 checkbox 行（`bc-auto-close` 旁）新增一个 checkbox：

```html
<div class="checkbox-row">
  <input type="checkbox" id="bc-auto-collect-close">
  <label for="bc-auto-collect-close" class="checkbox-label">继续时自动收集并关闭</label>
</div>
```

**不加 `checked` 属性**（默认关闭），与 `bc-auto-close`（默认开启）形成对比。

## 状态与持久化

新增状态变量，模式与现有 `autoCloseEnabled` 完全一致：

```js
let autoCollectCloseEnabled = localStorage.getItem('bc_auto_collect_close') === 'true'; // 默认关闭
```

初始化同步 checkbox：
```js
document.getElementById('bc-auto-collect-close').checked = autoCollectCloseEnabled;
```

监听 change 事件持久化：
```js
document.getElementById('bc-auto-collect-close').addEventListener('change', function() {
  autoCollectCloseEnabled = this.checked;
  localStorage.setItem('bc_auto_collect_close', autoCollectCloseEnabled);
});
```

## 核心代码改动

### `confirmBroadcast()` 开头新增（约 line 1058-1064 之后）

```js
function confirmBroadcast() {
  closePreviewModal();

  bcBatchSize = Math.max(1, parseInt(document.getElementById('bc-batch-size').value) || 10);
  bcStayTime = Math.max(1, parseInt(document.getElementById('bc-stay-time').value) || 20);
  bcDelay = Math.max(100, parseInt(document.getElementById('bc-delay').value) || 1000);
  localStorage.setItem('bc_delay', bcDelay);

  // ▼ 新增：继续时自动收集失败 + 关闭窗口（首批跳过）
  if (autoCollectCloseEnabled && bcIndex > 0 && openedWindows.length > 0) {
    collectFailedQQs();
    closeAllOpenedWindows();
  }

  // 计算本批次 ...（原逻辑不变）
}
```

## 不改动的部分（YAGNI）

- `collectFailedQQs()` / `closeAllOpenedWindows()` 函数体**完全不动**，只是被自动调用
- 不改 `openBatch()`、不改失败列表 UI 逻辑、不改任务标签
- 不新增按钮、不新增弹窗确认（开关本身即用户选择）
- 不改 `makeURL()`、不改 QQ 校验

## 验收标准

- [ ] 开关默认关闭，刷新页面后状态保持
- [ ] 开关关闭时：点击"继续打开会话"行为与现在完全一致（不自动收集/关闭）
- [ ] 开关开启时：点击"继续打开会话"→ 确认后，先自动收集失败 QQ（失败列表更新），再关闭所有窗口，再打开下一批
- [ ] 首批（`bcIndex === 0`）即使开关开启也不执行收集/关闭
- [ ] 收集逻辑与手动点"收集失败"结果一致（仍打开的窗口 = 失败）
- [ ] "添加好友"模式（`bcMode === 'add'`）下同样生效
- [ ] `openedWindows` 为空时不调用收集/关闭（避免无意义 toast）

## 风险与边界

- **首批守卫**：`bcIndex > 0` 确保首次打开不会误调收集（此时 `openedWindows` 本就为空，双重保险）
- **空窗口守卫**：`openedWindows.length > 0` 避免连续点击继续时产生"已关闭 0 个窗口"的无意义 toast
- **与"自动关闭窗口"开关的关系**：两者独立。若 `autoCloseEnabled` 开启，窗口会在停留时间后自动关闭，此时"收集失败"会收集到空列表（因为窗口已关）——这是预期行为，符合现有手动流程的语义

## 涉及文件

- `qq-session-tool.html`（唯一文件）
  - HTML：群发设置区新增 checkbox（约 line 835-838 附近）
  - JS 状态：新增 `autoCollectCloseEnabled` 变量（约 line 943 附近）
  - JS 逻辑：`confirmBroadcast()` 开头新增自动清理块（约 line 1064 附近）
  - JS 初始化：同步 checkbox 状态 + 监听 change（约 line 1307-1312 附近）
