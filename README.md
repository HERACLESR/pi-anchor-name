# pi-anchor-name

> **Session names as anchors, not labels.**
> 会话名字不是标签，是锚点 —— 多年后你在 `/resume` 列表里一眼认出它、瞬间回到当时现场的那根锚。

[English](#english) · [中文](#中文)

---

## English

### The problem

Sessions without a name show your first message in the selector. That first message is rarely what the session actually became: you open it saying *"look at this error"*, and three hours later it was a refactor, a procurement list, and a side quest. Existing auto-naming plugins freeze a title from message #1 — or worse, re-title your session on a timer, whether the topic moved or not.

### How pi-anchor-name is different

| | pi-session-name | pi-autoname | **pi-anchor-name** |
|---|---|---|---|
| Names after the first turn's **actual outcome** | ❌ (first prompt only) | ✅ | ✅ |
| Renames when the topic **drifts** | ❌ | ⚠️ timer-based (every N minutes) | ✅ drift-based (every N turns, model must say the topic moved) |
| Stays put when the topic **hasn't** moved | ✅ | ❌ | ✅ (model answers `KEEP`, nothing happens) |
| Manual names (`/name`) are **never** overwritten | ✅ | ⚠️ off by default | ✅ always, enforced across restarts |
| Date anchor prefix (`YYMMDD-`) | ❌ | ❌ | ✅ (session creation date, stable across renames) |
| Background timers / animations | none | yes | **none** — it only acts at turn boundaries |
| Runtime dependencies | 0 | 0 | **0**, single file |

### Behavior

1. **First naming** — after the first agent run fully settles (retries, compaction and queued messages included), an unnamed session gets a name generated from what actually happened, not just the opening line.
2. **Drift check** — every `recheckEveryTurns` settled turns (default 5), the naming model is asked one question: *has the session's goal clearly moved?* If yes → new name. If no → `KEEP`, silence. Names are stable by default.
3. **Manual names are sacred** — any name this extension did not set (`/name`, `--name`, or a manual `/rename`) is permanently off-limits, remembered across restarts via a tiny local state file. Exception: a name carrying this extension's own title format (`YYMMDD-` / `W-YYMMDD-`) is auto-adopted even if state was lost, so auto-management never dies by accident.
4. **Time anchor** — auto names are prefixed `YYMMDD-` (the session's creation date), so the `/resume` list sorts chronologically and the prefix never changes, even after later renames.

### Install

```bash
pi install git:github.com/HERACLESR/pi-anchor-name
```

Or try it without installing:

```bash
pi -e git:github.com/HERACLESR/pi-anchor-name
```

### Commands

| Command | Effect |
|---|---|
| `/rename` | Regenerate a name from the full conversation, then lock (auto-naming steps back) |
| `/rename <text>` | Set the name directly, then lock |
| `/rename adopt` | Escape hatch: re-take auto-management of the current name (rarely needed — names in our own format are adopted automatically) |
| `/name <text>` | pi's built-in rename — also respected forever |

### Configuration (optional)

Create `~/.pi/agent/extensions/anchor-name.json` — no file means all defaults. Every behavior has its own switch:

```json
{
  "enabled": true,
  "continuous": true,
  "datePrefix": true,
  "workerPrefix": true,
  "log": true,
  "language": "auto",
  "recheckEveryTurns": 5,
  "model": "provider/modelId",
  "maxTitleChars": 60
}
```

| Key | Default | Meaning |
|---|---|---|
| `enabled` | `true` | Master switch |
| `continuous` | `true` | Drift checks after the first naming |
| `datePrefix` | `true` | `YYMMDD-` time-anchor prefix on auto names |
| `workerPrefix` | `true` | Non-interactive runs (subagent workers, one-shot `-p`) get a `W-` marker so they never blend into your main session list |
| `log` | `true` | Black-box log (`anchor-name.log`, self-trimming, hard cap 64 KB) |
| `language` | `"auto"` | `auto` = follow the user's own language; or force one, e.g. `"Chinese"`, `"English"` — hard instruction, immune to English tool/file names in the excerpt |
| `recheckEveryTurns` | `5` | Settled turns between drift checks |
| `model` | *(current model)* | Dedicated naming model, e.g. a small local one |
| `maxTitleChars` | `60` | Title length cap |

### Privacy

The only data that ever leaves your machine is a short excerpt (last few messages, truncated) sent to **the model provider you already use**, via pi's own model registry and auth. No telemetry, no network of its own, no credentials touched. Read the single file — it is the whole product.

### Cost

Naming is nearly free: the excerpt is capped at ~6 KB and the answer at a handful of words. On a local model a naming call completes in well under a second — so light it is easy to miss on a server dashboard. No background timers, no polling: the extension only wakes at turn boundaries.

### Uninstall

```bash
pi remove git:github.com/HERACLESR/pi-anchor-name
```

Names it already wrote stay in your sessions (they're just session names); nothing else to clean up.

### Bonus: backfill old sessions

`scripts/backfill-history.ts` names your existing unnamed sessions in one pass (dry-run by default):

```bash
bun scripts/backfill-history.ts            # preview only
bun scripts/backfill-history.ts --apply    # write
```

Point it at any OpenAI-compatible endpoint via `PI_ANCHOR_API` / `PI_ANCHOR_KEY` / `PI_ANCHOR_MODEL`.

---

## 中文

**一句话**：给 pi 的会话自动起名字，而且只在**话题真的变了**才改名。

很多插件拿你的第一句话起名字，可你的会话往往不是从第一句话长成最后那样的；另一些插件挂个定时器，每十分钟不管你聊没聊新东西都重命名一次。pi-anchor-name 的立场是：**名字默认稳定，只有事实变了才动**。

- 第一轮活**干完**之后才命名（看实际干了什么，不是猜你第一句）；
- 每 5 轮（可调）问一次模型"话题漂移了吗"，没漂移就答 `KEEP`，什么都不发生；
- 你手动起的名字是最高优先级，跨重启永久豁免；
- 自动名字带 `YYMMDD-` 日期前缀，`/resume` 列表天然按时间排序；
- 零依赖、单文件、无后台定时器，官方将来内置该功能时删文件即拆除。

设计动机、与官方内置功能的对比思路，见 README 英文节，逻辑相同。

## License

MIT
