# Deep Interview Spec: QQ 机器人集成

## Metadata
- Interview ID: qq-bot-integration-001
- Rounds: 7
- Final Ambiguity Score: 15%
- Type: brownfield
- Generated: 2026-03-25
- Threshold: 20%
- Status: PASSED

## Clarity Breakdown
| Dimension | Score | Weight | Weighted |
|-----------|-------|--------|----------|
| Goal Clarity | 0.90 | 0.35 | 0.32 |
| Constraint Clarity | 0.90 | 0.25 | 0.23 |
| Success Criteria | 0.85 | 0.25 | 0.21 |
| Context Clarity | 0.85 | 0.15 | 0.13 |
| **Total Clarity** | | | **0.88** |
| **Ambiguity** | | | **12%** |

## Goal

集成官方 QQ 机器人到现有 QQ 临时会话工具，实现网页生成 QQ 号列表/链接后，调用机器人 API 将消息发送给机器人的好友。

## Constraints

1. **官方机器人限制**：无法向陌生人发消息，只能发送给已添加机器人的好友
2. **架构要求**：前端 + 后端服务，不能纯前端调用
3. **目标接收方**：仅限机器人好友
4. **保留双轨方案**：官方机器人 + 网页工具并行使用

## Non-Goals

- 不实现向陌生人发消息（官方 API 限制）
- 不实现 QQ 群群发（需要额外的入群和权限配置）
- 不替换现有网页工具的核心功能

## Acceptance Criteria

- [ ] 后端服务提供机器人 API 接口封装
- [ ] 网页工具可以调用后端发送消息给机器人好友
- [ ] 支持批量发送链接给多个好友
- [ ] 发送结果有状态反馈（成功/失败）
- [ ] 用户可配置机器人 API 凭证

## Assumptions Exposed & Resolved

| Assumption | Challenge | Resolution |
|------------|-----------|------------|
| 机器人可向任意 QQ 号发消息 | 官方 API 限制，只能向好友发消息 | 接受限制，仅支持机器人好友 |
| 可以纯前端调用机器人 | 需要 AppID/AppSecret，不能暴露在前端 | 采用前端 + 后端架构 |
| 机器人可实现临时会话 | 官方机器人不支持主动发起临时会话 | 改为转发链接，让好友点击打开 |

## Technical Context

**现有代码**：`qq-session-tool.html` 单文件应用
- 链接生成：`https://wpa.qq.com/msgrd?v=3&uin=xxx`
- 群发功能：依次打开多个新标签页

**需要新增**：
1. 后端服务（Node.js/Python）封装 QQ 开放平台 API
2. 前端调用后端的发送接口
3. 机器人好友列表管理
4. 消息发送状态追踪

## Ontology (Key Entities)

| Entity | Type | Fields | Relationships |
|--------|------|--------|---------------|
| QQNumber | core domain | value (5-12位数字) | 关联到 SessionLink, BotFriend |
| SessionLink | core domain | url (wpa.qq.com格式), qq | 由 QQNumber 生成 |
| BotFriend | core domain | openid, nickname, avatar | 机器人的好友 |
| Message | supporting | content, target, status | 发送给好友的消息 |
| BotConfig | supporting | appid, appsecret, token | 机器人配置信息 |

## Interview Transcript
<details>
<summary>Full Q&A (7 rounds)</summary>

### Round 1
**Q:** 你现有的 QQ 机器人是什么类型？使用的是什么框架或协议？
**A:** 官方 QQ 机器人
**Ambiguity:** 80%

### Round 2
**Q:** 官方 QQ 机器人有一些限制（如不能主动向陌生人发消息）。你希望实现的具体场景是什么？
**A:** 陌生人临时会话 + QQ 群群发
**Ambiguity:** 60%

### Round 3
**Q:** 官方 QQ 机器人不支持向陌生人发消息和 QQ 群群发。你是否考虑改用第三方协议？
**A:** 保留官方机器人 + 网页工具
**Ambiguity:** 45%

### Round 4
**Q:** 你希望 QQ 机器人与网页工具如何协同工作？
**A:** 网页调用机器人
**Ambiguity:** 35%

### Round 5
**Q:** 由于官方机器人无法向陌生人发消息，你希望机器人在这个流程中扮演什么角色？
**A:** 机器人转发链接/消息
**Ambiguity:** 25%

### Round 6
**Q:** 你希望如何连接网页和机器人？是否已有后端服务？
**A:** 前端 + 后端服务
**Ambiguity:** 18%

### Round 7
**Q:** 机器人发送链接/消息的目标对象是什么？
**A:** 机器人好友
**Ambiguity:** 15%
</details>