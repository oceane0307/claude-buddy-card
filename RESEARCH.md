# Claude Code 源码泄露事件研究 — Buddy 宠物系统

## 事件概述（2026年3月31日）

Anthropic 在发布 Claude Code v2.1.88 到 npm 时，意外地将一个 **59.8MB 的 source map 文件**（.map）包含在发布包中。该文件将混淆后的 JS 代码映射回原始 TypeScript 源码，导致约 **512,000 行代码、1,900+ 个文件** 全部暴露。

安全研究员 **Chaofan Shou**（@Fried_rice）最先在 X 上公开了发现，帖子获得近千万浏览。源码随后被镜像到 GitHub，获得超过 41,500 次 fork。

Anthropic 回应称："这是人为错误导致的打包问题，不涉及客户数据或凭证泄露。"

---

## Buddy 宠物系统（最有趣的发现）

泄露代码中最令人惊喜的发现之一是一个完整的 **Tamagotchi 风格虚拟宠物系统**，内部代号 **"Buddy"**。

### 核心机制

| 特性 | 详情 |
|------|------|
| **物种数量** | 18 种（鸭子、龙、六角恐龙、水豚、蘑菇、幽灵等） |
| **稀有度分层** | 从 Common 到 Legendary（1%），最稀有为 Shiny Legendary |
| **五维属性** | DEBUGGING（调试）、PATIENCE（耐心）、CHAOS（混乱）、WISDOM（智慧）、SNARK（毒舌） |
| **外观系统** | 帽子装饰、闪光（Shiny）变体 |
| **视觉渲染** | 5行高×12字符宽的 ASCII 艺术，含多帧动画（待机动画、反应动画） |
| **显示位置** | 坐在你的输入提示符旁边 |

### 分配机制

- **确定性抽卡（Deterministic Gacha）**：使用用户 ID 作为伪随机种子，同一用户永远获得同一个 Buddy
- **超稀有概率**：Shiny Legendary Nebulynx（星云猞猁）的出现概率仅为 **0.01%**
- **个性描述**：首次孵化时由 Claude 生成独特的性格描述
- **互动能力**：Buddy 不仅是装饰，它有自己的个性，可以在被叫名字时做出回应

### 计划发布时间

- 2026年4月1日至7日：预告/彩蛋期
- 2026年5月：正式上线

---

## 其他重要发现

### 1. KAIROS 自主守护模式
一个让 Claude Code 在用户空闲时继续运行的后台模式，执行"记忆整合"——合并观察结果、消除矛盾、将模糊洞察转化为确切事实。

### 2. 反蒸馏措施（Anti-Distillation）
`claude.ts` 中有一个 `ANTI_DISTILLATION_CC` 标志。启用后，API 请求会注入伪造的工具定义到系统提示中，污染任何试图通过录制 API 流量来训练竞争模型的数据。

### 3. "卧底模式"（Undercover Mode）
一个防止 AI 在参与开源项目时意外泄露 Anthropic 内部代号和项目名称的子系统。系统提示中写道："Do not blow your cover."（不要暴露身份。）

### 4. API 调用 DRM
`system.ts` 中的 API 请求包含一个哈希占位符，在请求发出前由 Bun 的原生 HTTP 层计算并替换，服务端验证该哈希以确认请求来自真正的 Claude Code 二进制文件。

### 5. 44 个隐藏功能标志
包括 ULTRAPLAN（30分钟远程规划）、coordinator mode（协调模式）、agent swarms（代理集群）、workflow scripts 等，全部已编译但通过 feature flag 关闭。

### 6. 内部模型代号
- Claude 4.6 变体代号：**Capybara**
- Opus 4.6 代号：**Fennec**
- 未发布模型：**Numbat**（仍在测试中）

---

## 安全警告

### npm 供应链攻击
2026年3月31日 00:21–03:29 UTC 期间通过 npm 安装或更新 Claude Code 的用户，可能下载到了包含远程访问木马（RAT）的恶意 axios 版本（1.14.1 或 0.30.4）。Anthropic 建议使用官方安装器而非 npm。

---

## 讽刺之处

Anthropic 专门构建了"卧底模式"来防止 AI 泄露内部信息，结果他们自己把全部源码塞进了 .map 文件发布了出去。

---

## 参考来源

- [VentureBeat: Claude Code's source code appears to have leaked](https://venturebeat.com/technology/claude-codes-source-code-appears-to-have-leaked-heres-what-we-know)
- [The Register: Anthropic accidentally exposes Claude Code source code](https://www.theregister.com/2026/03/31/anthropic_claude_code_source_code/)
- [Fortune: Anthropic leaks its own AI coding tool's source code](https://fortune.com/2026/03/31/anthropic-source-code-claude-code-data-leak-second-security-lapse-days-after-absolutely-revealing-mythos/)
- [Alex Kim's Blog: The Claude Code Source Leak](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/)
- [Cybernews: Full source code for Anthropic's Claude Code leaks](https://cybernews.com/security/anthropic-claude-code-source-leak/)
- [Matt Paige: Claude Code's Entire Source Code Just Leaked](https://mattpaige68.substack.com/p/claude-codes-entire-source-code-just)
- [Kuber Studio: Claude Code Source Code Breakdown](https://kuber.studio/blog/AI/Claude-Code's-Entire-Source-Code-Got-Leaked-via-a-Sourcemap-in-npm,-Let's-Talk-About-it)
- [GitHub Mirror: Kuberwastaken/claude-code](https://github.com/Kuberwastaken/claude-code)
