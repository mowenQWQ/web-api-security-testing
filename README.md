# web-api-security-testing

**Web API 安全测试的轮次化执行流程——测试前 3 件事、测试中手段手册、测试后 5 件事闭环，轮轮可复用。**
**A round-based workflow for Web API security testing — 3 pre-flight checks, an in-flight technique handbook, and a fixed 5-step post-round closure. Reusable round after round.**

[中文](#中文) | [English](#english)

> ⚠️ **AI 使用说明**：本作品使用 AI 辅助生成（生成时间：2026-09-05）。仅供学习与参考使用，发布、引用或投入使用前请人工核实关键信息。

---

## 中文

### 这是什么

把"一次测试轮次"工程化的可复用流程，核心思想是**每轮都站在上一轮的肩膀上**：

- **测试前**：token 有效性验证、JS 变更检测（主 JS 文件名指纹）、读漏洞账本里的到期复查项
- **测试中**：JS 先读后测（参数/端点全集在 chunk 里，比盲测高效 10 倍）、行为学验证额度类修复（精确计数封口）、网络捕获验证幂等、长测试后台化
- **测试后（顺序固定）**：报告存档 → 立即发送 → 更新手段手册 → 更新漏洞账本 → 日记补记
- **铁律**：危险操作前先存档、副作用最小化、同款漏洞模式立即 sweep 全站、sweep 必须读业务码而非 HTTP 状态码

### 踩坑实录（每条都交过学费）

- HTTP 200 陷阱：业务层 403/404 包在 200 里返回，管理端 sweep 的"200"全是拒绝
- 400 响应也计入频率窗口，等待要留足余量
- "可信凭证"免费铸造 = 免检通道：配额校验要对所有获取路径统一生效
- 报告测完不主动发，用户要两次追问——交付纪律是流程的一部分

### 安装

```bash
git clone https://github.com/mowenQWQ/web-api-security-testing.git
cp -r web-api-security-testing /path/to/your/agent/skills/
```

---

## English

### What is this

An engineered, reusable workflow for one testing round, built on the idea that **every round stands on the previous round's shoulders**:

- **Pre-flight**: token validity check, JS change detection (main-bundle filename fingerprint), due retest items from the findings ledger
- **In-flight**: read JS before testing (params & endpoint inventory live in chunks — 10x faster than blind probing), behavioral verification of quota fixes (exact-count capping), network-capture idempotency proof, backgrounded long tests
- **Post-round (fixed order)**: archive report → send immediately → update technique handbook → update findings ledger → journal backfill
- **Iron rules**: archive before destructive ops, minimize side effects, sweep same-pattern vulnerabilities site-wide, read business codes not HTTP status codes

### War stories (each one cost real time)

- HTTP 200 trap: business-layer 403/404 wrapped in 200 responses — every "200" in an admin sweep was a rejection
- 400 responses still count against rate windows; wait with margin
- Freely mintable "trusted credentials" = a bypass lane: quota checks must apply uniformly across all acquisition paths
- Not sending the report proactively means the user asks twice — delivery discipline is part of the workflow

### Install

```bash
git clone https://github.com/mowenQWQ/web-api-security-testing.git
cp -r web-api-security-testing /path/to/your/agent/skills/
```

---

## License

MIT-0（MIT No Attribution）— 详见 [LICENSE](LICENSE)。任何人可自由使用、修改、再分发，无需署名。

MIT No Attribution — see [LICENSE](LICENSE). Free to use, modify, and redistribute with no attribution required.
