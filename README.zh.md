[English](README.md) | [한국어](README.ko.md) | 中文 | [日本語](README.ja.md) | [Español](README.es.md)

# nopal

<p align="center">
  <img src="assets/nopal-hero-01.png" alt="nopal" width="320">
</p>

> **用自然语言驱动的 Google Workspace 编排。**

一句普通的话，就能变成横跨 Gmail、Calendar、Drive、Docs、Sheets、Slides、Meet、Tasks 和 Chat 的协同操作——全程无需离开 Claude Code。

[快速开始](#快速开始) • [为什么选 nopal？](#为什么选-nopal) • [工作原理](#工作原理) • [支持的服务](#支持的服务) • [环境要求](#环境要求)

---

## 快速开始

### 1. 添加市场（仅需一次）

```
/plugin marketplace add https://github.com/fivetaku/gptaku_plugins.git
```

### 2. 安装 nopal

```
/plugin install nopal
```

安装后请重启 Claude Code。

### 3. 配置 gws CLI（仅需一次）

nopal 通过 [gws CLI](https://github.com/googleworkspace/cli) 与 Google Workspace 通信。请先安装：

```bash
npm install -g @googleworkspace/cli
```

然后在终端中运行一次性的 OAuth 设置：

```bash
gws auth setup
```

它会引导你创建 GCP 项目、启用 9 个 Workspace API 并授权你的 Google 账号。设置完成后登录：

```bash
gws auth login
```

登录后导出凭据，让 Claude Code 能在 headless 模式下使用 gws：

```bash
gws auth export --unmasked 2>/dev/null | grep -v '^Using keyring' > ~/.config/gws/credentials.json
chmod 600 ~/.config/gws/credentials.json   # 明文令牌 — 注意保密，切勿提交到仓库
```

### 4. 运行

```
/nopal
```

无需任何参数——nopal 会主动询问你想做什么。也可以直接下指令：

```
/nopal 明天上午10点安排一次团队站会，并把议程邮件发给参会者
/nopal 检查我的未读邮件，总结其中重要的
/nopal 创建一份会议纪要文档，分享给上周的参会者
/nopal 从 Sheets 拉取一季度销售数据，把摘要发到团队聊天
```

---

## 为什么选 nopal？

- **一条命令，任意服务** —— 用平常的语言描述需求，nopal 自行判断该调用哪些服务、按什么顺序执行
- **动态组合** —— 不是固定的工作流库；每次请求都会重新选择并串联服务
- **访谈式交互** —— 信息不足时，nopal 会在执行前先问（而不是执行后）
- **读写分离** —— 只读查询立即执行；写入和修改操作一律先经你确认
- **就在 Claude Code 里** —— 不装新应用、不开浏览器标签页、不切换上下文
- **凭据留在 gws 中** —— OAuth 流程由 gws CLI 负责。headless 场景下它会把令牌导出到本地的 `~/.config/gws/credentials.json`（保持 `chmod 600`，切勿提交到仓库）；nopal 绝不会把凭据嵌入 Claude 本身

---

## 工作原理

```
你: "明天下午2点安排团队会议，并给参会者发邮件"
     │
     ▼
/nopal
     │
     ├─ gws 未安装？ → 尝试自动安装 / 提供设置引导
     │
     └─ gws 就绪 → 开始编排
          │
          ├─ 1. 解析意图      — 需要哪些服务？
          ├─ 2. 访谈          — 拉取实时数据，只问缺失的信息
          ├─ 3. 规划          — 写入操作先确认，只读操作直接跳过
          ├─ 4. 执行          — 按顺序运行 gws 命令
          └─ 5. 汇报          — 总结结果 + 建议后续步骤
```

跨服务请求会自然地被拆解：

- "给明天的会议添加参会者，并把文档发给他们" → Calendar + Drive + Gmail
- "用 Sheets 数据生成一份简报并发送" → Sheets + Gmail
- "写好会议纪要，发到团队 Chat 空间" → Docs + Chat

---

## 支持的服务

| 服务 | nopal 能做什么 | 辅助命令 |
|---------|-------------------|-----------------|
| Gmail | 发送、阅读、分类、监听 | `+send`, `+triage`, `+watch` |
| Calendar | 创建日程、查看安排 | `+insert`, `+agenda` |
| Drive | 上传文件、管理共享 | `+upload` |
| Sheets | 读取/追加表格数据 | `+read`, `+append` |
| Docs | 读写文档 | `+write` |
| Slides | 创建和编辑演示文稿 | — |
| Chat | 向空间发送消息 | `+send` |
| Tasks | 管理待办清单 | — |
| Meet | 创建会议链接、获取参会者与转录文字 | — |

---

## 已知问题

| 问题 | 状态 | 解决办法 |
|-------|--------|------------|
| Gmail trash 411 错误 | gws 0.6.1+ 已修复 | 使用最新版本 |
| `+send` 韩文编码问题 | gws CLI 缺陷 | 已自动改用 raw API 编码 |
| `gws auth export` 日志混入 | `Using keyring backend` 混进 JSON | 已应用 `2>/dev/null \| grep -v '^Using keyring'` 过滤 |

---

## 环境要求

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [gws CLI](https://github.com/googleworkspace/cli) — `npm install -g @googleworkspace/cli`
- Google Workspace 账号 + OAuth 设置（`gws auth setup` + `gws auth login`）
- Node.js 18+

> 首次运行 `/nopal` 时，它会自动检测 gws 是否就绪，并引导你完成设置。

---

## 许可证

MIT

---

<div align="center">

**No Opal needed. —— 不需要 Opal。**

</div>
