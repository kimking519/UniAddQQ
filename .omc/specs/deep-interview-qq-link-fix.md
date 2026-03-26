# Deep Interview Spec: QQ 临时会话链接修复

## Metadata
- Interview ID: qq-link-risk-001
- Rounds: 8
- Final Ambiguity Score: 14%
- Type: brownfield
- Generated: 2026-03-25
- Threshold: 20%
- Status: PASSED

## Clarity Breakdown
| Dimension | Score | Weight | Weighted |
|-----------|-------|--------|----------|
| Goal Clarity | 0.95 | 0.35 | 0.33 |
| Constraint Clarity | 0.90 | 0.25 | 0.23 |
| Success Criteria | 0.90 | 0.25 | 0.23 |
| Context Clarity | 0.90 | 0.15 | 0.14 |
| **Total Clarity** | | | **0.92** |
| **Ambiguity** | | | **8%** |

## Goal

将 QQ 临时会话工具的所有链接从 `tencent://` 协议改为 `https://wpa.qq.com/msgrd` 格式，解决腾讯客户端拦截陌生人临时会话请求的问题。

## Constraints

1. **平台限制**：wpa.qq.com 方案同样需要对方开启临时会话权限，不能保证 100% 成功
2. **操作方式**：改为"只生成链接，手动操作"，不自动触发跳转
3. **修改范围**：全部三个功能（单个会话、批量会话、群发）
4. **兼容性**：保持现有 UI 布局和交互流程不变

## Non-Goals

- 不解决腾讯平台本身的临时会话限制
- 不实现自动化批量发送（受平台限制）
- 不添加绕过平台限制的功能

## Acceptance Criteria

- [ ] 单个会话功能生成 wpa.qq.com 格式链接
- [ ] 批量会话功能生成 wpa.qq.com 格式链接
- [ ] 群发功能改为生成链接列表（不自动打开）
- [ ] 所有"打开"按钮改为 `window.open()` 在新标签页打开
- [ ] 用户可以复制链接手动使用

## Assumptions Exposed & Resolved

| Assumption | Challenge | Resolution |
|------------|-----------|------------|
| 浏览器拦截了链接 | 调查发现是 QQ 客户端弹窗 | 确认是腾讯安全策略限制非好友临时会话 |
| `tencent://` 协议无法使用 | 是客户端拒绝，非协议问题 | 改用 wpa.qq.com 可能有更高成功率 |
| 自动打开链接更方便 | 群发自动打开会触发多次拦截 | 用户选择改为手动操作模式 |

## Technical Context

**当前实现** (`qq-session-tool.html`):
- `makeURL(qq)` 函数返回 `tencent://message/?uin=${qq}&Site=&Menu=yes`
- 通过 `window.location.href = url` 触发跳转
- 批量会话和群发功能使用相同的链接生成逻辑

**修改方案**:
```javascript
function makeURL(qq) {
  return `https://wpa.qq.com/msgrd?v=3&uin=${qq}&site=qq&menu=yes`;
}
```

**行为变更**:
- 单个会话：生成链接后不自动跳转，用户点击"打开"按钮
- 批量会话：每个"打开"按钮使用 `window.open(url, '_blank')`
- 群发功能：取消自动打开逻辑，改为生成链接列表供用户手动操作

## Ontology (Key Entities)

| Entity | Type | Fields | Relationships |
|--------|------|--------|---------------|
| QQNumber | core domain | value (5-12位数字) | 关联到 SessionLink |
| SessionLink | core domain | url (wpa.qq.com格式), qq | 由 QQNumber 生成 |
| UserAction | supporting | type (打开/复制), target | 作用于 SessionLink |

## Interview Transcript
<details>
<summary>Full Q&A (8 rounds)</summary>

### Round 1
**Q:** 你在什么环境下使用这个工具？这个错误提示来自哪个浏览器或应用？
**A:** 桌面浏览器（Chrome/Edge等）
**Ambiguity:** 80% (Goal: 0.6, Context: 0.5, Criteria: 0.5, Constraints: 0.5)

### Round 2
**Q:** 具体是在哪个操作步骤遇到这个错误？
**A:** 点击列表中的「打开」按钮
**Ambiguity:** 60% (Goal: 0.7, Context: 0.7, Criteria: 0.6, Constraints: 0.5)

### Round 3
**Q:** 「链接有潜在的风险，暂不支持跳转」这个错误提示是从哪里弹出的？
**A:** QQ 客户端弹窗
**Ambiguity:** 40% (Goal: 0.8, Context: 0.9, Criteria: 0.7, Constraints: 0.7)

### Round 4
**Q:** 你尝试发起会话的 QQ 号，和你的 QQ 是好友关系吗？
**A:** 非好友，想发起临时会话
**Ambiguity:** 30% (Goal: 0.9, Context: 0.9, Criteria: 0.8, Constraints: 0.8)

### Round 5
**Q:** 你使用这个工具的实际业务场景是什么？
**A:** 批量营销/通知
**Ambiguity:** 25% (Goal: 0.9, Context: 0.9, Criteria: 0.8, Constraints: 0.85)

### Round 6
**Q:** 腾讯已限制陌生人临时会话，你希望如何处理这个工具？
**A:** 尝试 wpa.qq.com 方案
**Ambiguity:** 20% (Goal: 0.9, Context: 0.9, Criteria: 0.85, Constraints: 0.85)

### Round 7
**Q:** wpa.qq.com 方案同样有平台限制（对方需开启临时会话权限）。你对成功率的预期是什么？
**A:** 只生成链接，手动操作
**Ambiguity:** 15% (Goal: 0.9, Context: 0.9, Criteria: 0.9, Constraints: 0.85)

### Round 8
**Q:** 哪些功能需要改为 wpa.qq.com 链接格式？
**A:** 全部功能（单个会话、批量会话、群发）
**Ambiguity:** 14% (Goal: 0.95, Context: 0.9, Criteria: 0.9, Constraints: 0.9)
</details>