---
{"dg-publish":true,"permalink":"/2-cards/claude-code-source-of-truth/","tags":["主題/AI與知識工作流","source-of-truth"],"dg-note-properties":{"created":"2026-04-18","updated":"2026-04-18","tags":["主題/AI與知識工作流","source-of-truth"],"source":"Claude Code 官方文件 https://code.claude.com/docs/en/memory.md + 2026-04-18 Stanley 實測驗證"}}
---


# Claude Code 指令系統 Source of Truth

> 建立於 2026-04-18
>
> **這張卡是什麼**：關於 CLAUDE.md / `@import` / `.claude/rules/` / Auto Memory 的**唯一真理**，每一條都有官方文件原文引用。未來對這套系統有疑問時，讀這張就好。
>
> **取代以下卡片**（不要再直接引用它們，它們基於二手資訊有漏洞）：
> - [[2-cards/CLAUDE.md 架構優化筆記：Joe Njenga Masterclass × 我的系統比對\|CLAUDE.md 架構優化筆記：Joe Njenga Masterclass × 我的系統比對]]（2026-04-02）
> - [[2-cards/Claude 指令架構全解：CLAUDE.md 重構指南\|Claude 指令架構全解：CLAUDE.md 重構指南]]（2026-04-04）
> - [[0-inbox/CLAUDE-md 三層架構拆分待決\|CLAUDE-md 三層架構拆分待決]]（2026-04-08）
> - [[2-cards/Debug：釐清 CLAUDE.md\|Debug：釐清 CLAUDE.md]]（2026-04-02）
>
> **驗證狀態**：Claude Code CLI + Cowork 三項核心機制（`@import` / `.claude/rules/` paths / lazy load）已實測 PASS；claude.ai web app 尚未驗證。

---

## 0. 為什麼需要 Source of Truth

2026-04-02 到 2026-04-08 之間 Stanley 跟 Claude 討論過四輪 CLAUDE.md 架構，產出四張卡片，但：
- 當時的 Claude 沒查官方文件，引用的是 Joe Njenga Masterclass、雷蒙教學、HumanLayer 研究——全是二手資訊
- 四張卡片主張部分重複、部分衝突，導致 2026-04-18 重新討論時迷糊
- 最大漏洞：**`@import` 語法完全沒提到**（官方核心機制之一）
- 「200 行上限」、「100-150 條指令」、「80% 法則」——被當成社群傳言，但其實 200 行是**官方明確建議**

這張卡**只寫官方文件能查證的事實**，加上 2026-04-18 的實測結果。

---

## 1. 指令載入路徑（完整六層）

### 官方原文

> CLAUDE.md files can live in several locations, each with a different scope.
>
> | Scope | Location |
> | --- | --- |
> | Managed policy | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md` · Linux/WSL: `/etc/claude-code/CLAUDE.md` · Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` |
> | Project instructions | `./CLAUDE.md` or `./.claude/CLAUDE.md` |
> | User instructions | `~/.claude/CLAUDE.md` |
> | Local instructions | `./CLAUDE.local.md` |

來源：https://code.claude.com/docs/en/memory.md

### 載入順序（由外而內、合併而非覆蓋）

> Claude Code reads CLAUDE.md files by walking up the directory tree from your current working directory, checking each directory along the way for `CLAUDE.md` and `CLAUDE.local.md` files. This means if you run Claude Code in `foo/bar/`, it loads instructions from `foo/bar/CLAUDE.md`, `foo/CLAUDE.md`, and any `CLAUDE.local.md` files alongside them. **All discovered files are concatenated into context rather than overriding each other**. Within each directory, `CLAUDE.local.md` is appended after `CLAUDE.md`, so when instructions conflict, your personal notes are the last thing Claude reads at that level.

### 六層對照表

| # | 層級 | 路徑 | 何時載入 | 跨環境一致嗎 |
|---|------|------|---------|-------------|
| 1 | Managed Policy | `/Library/Application Support/ClaudeCode/CLAUDE.md`（macOS） | 一定載入（無法排除） | — |
| 2 | User Instructions | `~/.claude/CLAUDE.md` | 一定載入 | ⚠️ 只在 CLI（Cowork/web app 讀不到本地 `~/`） |
| 3 | Project CLAUDE.md | `./CLAUDE.md` or `./.claude/CLAUDE.md` | 啟動時（從 cwd 向上遞迴） | ✅（Dropbox sync） |
| 4 | Local CLAUDE.md | `./CLAUDE.local.md` | 啟動時（緊接 CLAUDE.md 之後） | ✅ |
| 5 | `.claude/rules/*.md` | 專案內 `.claude/rules/` | 有 `paths:` 則 lazy；無 `paths:` 則啟動載入 | ✅ |
| 6 | 子目錄 CLAUDE.md | 專案子目錄內 | **Lazy**（Claude 讀子目錄檔案時才載入） | ✅ |

**衝突時的處理**：「last read wins」——後載入的勝過先載入的。所以 Local > Project > User > Managed。

---

## 2. `@import` 語法

### 官方原文

> CLAUDE.md files can import additional files using `@path/to/import` syntax. **Imported files are expanded and loaded into context at launch** alongside the CLAUDE.md that references them.
>
> Both relative and absolute paths are allowed. Relative paths resolve relative to the file containing the import, not the working directory. Imported files can recursively import other files, with a **maximum depth of five hops**.

官方範例：
```markdown
See @README for project overview and @package.json for available npm commands.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

### 關鍵特性

| 特性 | 說明 |
|------|------|
| 展開時機 | **啟動時全部展開** — 等同內容寫在 CLAUDE.md 本體 |
| 省不省 token | ❌ **不省 token**（啟動時全載入） |
| 省什麼 | ✅ 省**維護成本與可讀性**（CLAUDE.md 骨幹乾淨） |
| 相對路徑基準 | 引入檔案**所在目錄**（不是 cwd） |
| 絕對路徑 | 支援（例：`@~/.claude/my-instructions.md`） |
| 遞迴深度 | 最多 5 層（hops） |

### 實測結果（2026-04-18）

| 環境 | 測試 | 結果 |
|------|------|------|
| Claude Code CLI | CLAUDE.md 加 `@.claude/test/pilot.md`，pilot.md 含口令「紫色大象」，新 session 直接問口令 | ✅ 答出「紫色大象」 |
| Cowork | 同上 | ✅ 答出「紫色大象」 |
| web app | — | 未驗證 |

---

## 3. `.claude/rules/*.md` 條件加載

### 官方原文

> For larger projects, you can organize instructions into multiple files using the `.claude/rules/` directory. This keeps instructions modular and easier for teams to maintain. **Rules can also be scoped to specific file paths, so they only load into context when Claude works with matching files, reducing noise and saving context space.**
>
> Place markdown files in your project's `.claude/rules/` directory. Each file should cover one topic, with a descriptive filename like `testing.md` or `api-design.md`. All `.md` files are discovered recursively, so you can organize rules into subdirectories.

### 官方範例（`paths:` frontmatter）

```markdown
---
paths:
  - "src/api/**/*.ts"
---

# API Development Rules
```

### 關鍵特性

| 特性 | 說明 |
|------|------|
| 無 `paths:` | 啟動時全部載入（跟 CLAUDE.md 同級） |
| 有 `paths:` | **Lazy load**：只在 Claude 讀匹配的檔案時才載入 |
| Glob 支援 | ✅（如 `"stanley-vault/test-trigger/**"`） |
| 省 token | ✅ **這是真正省啟動 context 的機制**（對比 `@import` 不省） |
| 多檔組織 | 遞迴 discovery，可放子目錄 |

### 實測結果（2026-04-18）

| 環境 | 測試 A（觸發後） | 測試 B（未觸發） | 測試 C（限制 tool scope） |
|------|----------------|----------------|------------------------|
| Claude Code CLI | ✅ 答出 paths 口令 | ✅ 答不出 paths 口令（確認 lazy） | ✅ 仍答出（系統自動載入非 Claude 自讀） |
| Cowork | ✅ 答出 paths 口令 | ✅ 答不出 paths 口令，Claude 自己詮釋「rule 只在讀取符合條件的路徑時才會載入，不會常駐」 | — |
| web app | — | — | 未驗證 |

### ⚠️ 重要補充：Pointer-driven vs Lazy load 的區分（2026-04-18 Stage 1 驗收揭露）

本卡初期描述 `.claude/rules/` 時，沒清楚區分兩種不同機制。Stage 1 實際重構並驗收後，這個區分變得關鍵：

**機制 1：Lazy load（本卡 Pilot A/B/C 驗證的）**
- **觸發條件**：Claude 先 Read 了匹配 `paths:` glob 的檔案
- **行為**：rule 內容自動進 context，Claude 不需再 Read rule 檔
- **狀態**：CLI + Cowork 已驗證（2026-04-18 上午 pilot）

**機制 2：Pointer-driven read（Stage 1 驗收揭露的日常實際行為）**
- **觸發條件**：CLAUDE.md 骨幹寫 pointer（「完整規則見 `.claude/rules/xxx.md`」），Claude 被問到相關問題時**主動** Read rule 檔
- **行為**：Claude 用 Read 工具讀 rule 檔，答題時會 cite source「根據 `.claude/rules/xxx.md`」
- **狀態**：CLI Stage 1 三題驗證（2026-04-18 晚上）

### 對使用者的實務意義

| 面向 | Lazy load | Pointer-driven |
|------|-----------|----------------|
| 啟動 context 佔用 | cwd 匹配就整個進來 | 零佔用 |
| Response latency | 直接答（不用 Read） | 多一個 Read tool call（+0.5-1 秒） |
| Cite source | 不會（資訊已混在 context） | ✅ 會（Claude 明確 cite「根據 xxx.md」） |

**實務結論**：Stanley 的 rule 設計（配合 CLAUDE.md 骨幹 pointer），實際運作以 **Pointer-driven 為主**——反而更省 token、更有追溯性。Lazy load 是「Claude 已經讀過 matching file 後」的 fallback 機制，補強 context 覆蓋率。

**尚未驗證**：paths 的 lazy load 精確觸發邊界（cwd 在 matching paths 底下會不會觸發？Grep / Glob / Bash 等其他檔案操作呢？）——見 Section 16 待驗證 #5。

---

## 4. CLAUDE.local.md

### 官方原文

> For private per-project preferences that shouldn't be checked into version control, create a `CLAUDE.local.md` at the project root. It loads alongside `CLAUDE.md` and is treated the same way. **Add `CLAUDE.local.md` to your `.gitignore` so it isn't committed; running `/init` and choosing the personal option does this for you**.
>
> Within each directory, `CLAUDE.local.md` is appended after `CLAUDE.md`, so when instructions conflict, your personal notes are the last thing Claude reads at that level.

### 關鍵特性

| 特性 | 說明 |
|------|------|
| 自動進 git 嗎 | ❌ **不會**，要手動加 `.gitignore`（或跑 `/init` 讓工具加） |
| 載入順序 | 同層級內：`CLAUDE.md` 先 → `CLAUDE.local.md` 後 |
| 衝突時 | `CLAUDE.local.md` 勝（後讀的優先） |
| 用途 | 個人偏好、敏感資訊、易腐資訊（當前焦點、完整通訊錄） |

---

## 5. 子目錄 CLAUDE.md

### 官方原文

> Claude also discovers `CLAUDE.md` and `CLAUDE.local.md` files in subdirectories under your current working directory. **Instead of loading them at launch, they are included when Claude reads files in those subdirectories.**

### 關鍵特性

- **Lazy load**：Claude 讀該目錄內檔案時才載入
- 官方文件**未精確定義**「讀取」的邊界（Read / Edit / Grep 是否都觸發）——實測確認
- 跟 `.claude/rules/*.md` with `paths:` 效果類似，但路徑範圍是「整個子目錄」而非 glob pattern

---

## 6. Auto Memory（MEMORY.md）

### 官方原文

> Auto memory lets Claude accumulate knowledge across sessions without you writing anything. Claude saves notes for itself as it works: build commands, debugging insights, architecture notes, code style preferences, and workflow habits.
>
> **The first 200 lines of `MEMORY.md`, or the first 25KB, whichever comes first, are loaded at the start of every conversation.**

### Auto Memory vs CLAUDE.md

| 面向 | CLAUDE.md | MEMORY.md |
|------|-----------|-----------|
| 誰寫 | 使用者 | Claude（自動） |
| 載入量 | 全部 | 前 200 行 or 25KB（先到為準） |
| 存儲位置 | `./CLAUDE.md` 等多層 | `~/.claude/projects/<project>/memory/` |
| 主要用途 | 規則與脈絡 | Claude 累積的模式與踩坑心得 |

### Stanley 的 symlink 設計

Stanley 把 `~/.claude/projects/.../memory/` 做成 symlink 指向 `$cowork-workspace/memory/`——這樣 MEMORY.md 的 auto-load magic 生效，同時 source of truth 在 Dropbox 跨機同步。（詳見 CLAUDE.md 的「Memory 系統」章節）

---

## 7. Managed Policy（企業級）

### 官方原文

> Organizations can deploy a centrally managed CLAUDE.md that applies to all users on a machine. **This file cannot be excluded by individual settings.**
>
> Create the file at the managed policy location:
> - macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`
> - Linux and WSL: `/etc/claude-code/CLAUDE.md`
> - Windows: `C:\Program Files\ClaudeCode\CLAUDE.md`

### Stanley 現況

✅ 不相關——Stanley 不是企業部署。此層級存在但他這輩子可能用不到，記錄備查。

---

## 8. CLAUDE.md 長度建議（官方）

### 官方原文

> **Size: target under 200 lines per CLAUDE.md file**. Longer files consume more context and reduce adherence. If your instructions are growing large, split them using imports or `.claude/rules/` files.

### 關鍵結論

- **200 行是官方建議，不是社群傳言**
- 超過 → 兩個副作用：(1) 吃 context；(2) **合規性下降**（Claude 對指令的遵守率會降低）
- **超過時的官方解法**：拆到 `@import` 或 `.claude/rules/`

---

## 9. 跨環境支援矩陣

| 機制 | Claude Code CLI | Cowork | claude.ai web app |
|------|----------------|--------|-------------------|
| Managed Policy CLAUDE.md | ✅ 官方明述 | ❓ 未驗證 | ❓ 未驗證 |
| User Instructions `~/.claude/CLAUDE.md` | ✅ 官方明述 | ❌ 讀不到本地 `~/` | ❌ |
| Project CLAUDE.md（向上遞迴） | ✅ 官方明述 | ✅ 實測（Dropbox 掛載） | ❓ |
| CLAUDE.local.md | ✅ 官方明述 | ✅ 實測 | ❓ |
| `@import` | ✅ **實測 PASS**（2026-04-18） | ✅ **實測 PASS**（2026-04-18） | ❓ 未驗證 |
| `.claude/rules/` + `paths:` 條件加載 | ✅ **實測 PASS**（2026-04-18） | ✅ **實測 PASS**（2026-04-18） | ❓ 未驗證 |
| 子目錄 CLAUDE.md lazy load | ✅ 官方明述 | ❓ 未驗證 | ❓ |
| Auto Memory MEMORY.md | ✅ 官方明述 | ✅ Stanley symlink 設定 | ❌ 無檔案系統 |

**官方立場**（overview）：

> Each surface connects to the same underlying Claude Code engine, so **your CLAUDE.md files, settings, and MCP servers work across all of them**.

—— 但具體同步機制官方未詳述，因此實測矩陣上仍有 `❓`。

---

## 10. 核心設計原則

### 原則 1：Context window 是稀缺資源

每次對話載入的內容都有成本——**不相關的內容不只沒幫助，還會稀釋真正重要的指令權重**。

- 能放 `.claude/rules/*.md` + `paths:` 的不放 CLAUDE.md
- 能放 Skill 的不放 CLAUDE.md
- 能讓 Auto Memory 自己學的不放 CLAUDE.md

### 原則 2：80% 法則

CLAUDE.md 只留「**80% 以上的對話都會用到**」的指令。其他的：
- 按情境觸發 → `.claude/rules/*.md` with `paths:`
- 只有特定任務用 → Skill
- Claude 自己學 → Auto Memory

### 原則 3：消除矛盾比搶優先順序重要

Claude 是語言模型，不是 if-else 程式。當規則矛盾時，它傾向聽「更具體、更後出現」的，但不保證 100%。**與其設計精密的優先順序，不如讓規則不矛盾**。

### 原則 4：頻率 × 專屬性兩軸分流

```
                   跨 workspace 需要        只這個 workspace 需要
每次對話都用到    → ~/.claude/CLAUDE.md    → ./CLAUDE.md
特定情境才用到    → (罕見，用 rules)       → .claude/rules/ with paths
個人 / 敏感       → (N/A，CLI 本來就私)    → CLAUDE.local.md
Claude 自己學     → —                      → Auto Memory
按需執行          → Skills（user or project）
```

### 原則 5：真實省 token 的唯一機制是條件加載

```
        啟動時全載入            條件載入（真省 token）
        ──────────              ──────────────────
        CLAUDE.md               .claude/rules/*.md
        @import                 with paths: frontmatter
        rules without paths     子目錄 CLAUDE.md（lazy）
```

`@import` 是維護工具，**不是省 token 工具**。不要以為拆成多檔 `@import` 就省了 context——啟動時還是全展開。

---

## 11. Stanley 的系統現況診斷（2026-04-18）

### 檔案清單

| 檔案 | 存在？ | 行數 | 狀態 |
|------|--------|------|------|
| `~/.claude/CLAUDE.md`（user-level） | ❌ 不存在 | — | 4/8 卡片建議建但未執行 |
| `~/.claude/settings.json` | ✅ | — | — |
| Managed Policy CLAUDE.md | ❌ 不存在 | — | 不相關（非企業部署） |
| `$cowork-workspace/CLAUDE.md` | ✅ | **118**（Stage 1 後） | ✅ 符合官方 ≤200 行建議 |
| `$cowork-workspace/CLAUDE.local.md` | ✅ | 54 | ✅ 合格（目錄非 git repo，用 Dropbox 版本控制 + CHANGELOG 代替） |
| `$cowork-workspace/.claude/rules/vault-operations.md` | ✅（Stage 1 建立） | 78 | paths: `stanley-vault/**`，lazy load |
| `$cowork-workspace/.claude/rules/skill-management.md` | ✅（Stage 1 建立） | 76 | paths: `Developer/skills/**`、`.claude/skills/**`、`_dist/**` |
| `$cowork-workspace/memory/README.md` | ✅（Stage 1 建立） | 102 | 架構文件，Claude 主動 Read 查閱（不 auto-load） |
| `$cowork-workspace/.claude/settings.local.json` | ✅ | — | — |
| `$cowork-workspace/memory/MEMORY.md` | ✅ | — | Symlink 設計已 work |

### 271 行 CLAUDE.md 內容結構分析

| 章節 | 行數 | 每次都要？ | 建議去向 |
|------|------|-----------|---------|
| `# Me / ## People / ## Terms / ## Teams` | ~38 | ✅ 80%+ 都用到 | **保留 CLAUDE.md** |
| `## Obsidian Vault`（路徑 / 結構 / frontmatter / skill 對照 / 檔名） | 52 | ⚠️ 部分（硬性規則要留，細節可搬） | 硬性規則留；細節搬 `.claude/rules/vault-operations.md` with `paths: ["stanley-vault/**"]` |
| `## Skill 管理規則`（三環境 / 檔案規格 / 打包紀律 / 更新流程 / 版號） | 66 | ❌ 只有改 skill 時用 | 搬 `.claude/rules/skill-management.md` with `paths: ["Developer/skills/**", ".claude/skills/**"]` |
| `## Memory 系統`（架構 / why symlink / 設置命令 / 結構 / 格式 / 何時寫讀） | 88 | ⚠️ 判斷標準要留；設置指令不用 | 判斷標準留；架構與設置搬 `memory/README.md`（可被 Read 查閱不強制載入） |
| `## 本地工具 whisper` | 9 | ❌ 只有音檔轉錄時用 | 搬 `.claude/rules/local-tools.md` with `paths` 或 memory reference |
| `## 系統操作規則`（11 條硬性規則） | 11 | ✅ 每次都要 | **保留 CLAUDE.md** |

### 四張舊卡片的收束

| 舊卡片 | 日期 | 結論 | 此卡相對狀態 |
|--------|------|------|-------------|
| [[2-cards/Debug：釐清 CLAUDE.md\|Debug：釐清 CLAUDE.md]] | 4/2 | 起點記錄，提出原始疑問 | **Superseded** — 疑問在此卡全部回答 |
| [[2-cards/CLAUDE.md 架構優化筆記：Joe Njenga Masterclass × 我的系統比對\|CLAUDE.md 架構優化筆記：Joe Njenga Masterclass × 我的系統比對]] | 4/2 | 大方向對（80% 法則、context 稀缺） | **Superseded** — 方法論此卡有更完整版本 |
| [[2-cards/Claude 指令架構全解：CLAUDE.md 重構指南\|Claude 指令架構全解：CLAUDE.md 重構指南]] | 4/4 | 架構圖大部分對，遺漏 `@import` | **Superseded** — 此卡補上 `@import` 與實測 |
| [[0-inbox/CLAUDE-md 三層架構拆分待決\|CLAUDE-md 三層架構拆分待決]] | 4/8 | park 中，建議搬 user-level | **部分採納** — user-level 機制真實存在，但「條件加載」有更精巧的 `.claude/rules/` 方案 |

---

## 12. 目標架構（重構後）

```
┌─────────────────────────────────────────────────────────┐
│ ~/.claude/CLAUDE.md（user-level，optional）              │
│ ├ 跨 workspace 通用：Me / Terms / 系統操作規則            │
│ └ 決定是否建：需先觀察自己從哪些 cwd 啟動 claude（4/8 park）│
├─────────────────────────────────────────────────────────┤
│ $cowork-workspace/CLAUDE.md（project，≤200 行）           │
│ ├ Me / People / Terms / Teams（~38 行）                  │
│ ├ Vault 硬性規則（路徑、禁止 mnt 捷徑）短版               │
│ ├ Memory 判斷標準（何時寫 / 何時讀）                      │
│ ├ 系統操作規則 11 條（~11 行）                            │
│ └ @import 維護分類（若有需要）                            │
├─────────────────────────────────────────────────────────┤
│ $cowork-workspace/CLAUDE.local.md（個人 + 易腐）          │
│ ├ 當前焦點（關機儀式自動更新）                            │
│ ├ 完整通訊錄                                             │
│ └ 活躍專案表                                             │
├─────────────────────────────────────────────────────────┤
│ $cowork-workspace/.claude/rules/（條件加載）              │
│ ├ skill-management.md — paths: ["Developer/skills/**"]   │
│ ├ vault-operations.md — paths: ["stanley-vault/**"]      │
│ └ local-tools.md（whisper 等） — paths 或無              │
├─────────────────────────────────────────────────────────┤
│ $cowork-workspace/memory/                                │
│ ├ MEMORY.md（auto-load 前 200 行）                       │
│ ├ README.md（memory 架構、why symlink、設置命令）         │
│ └ *.md（領域知識、feedback、reference）                   │
├─────────────────────────────────────────────────────────┤
│ ~/Developer/skills/（按需觸發）                           │
│ └ 40+ skill，source of truth，push-and-pack 工作流        │
└─────────────────────────────────────────────────────────┘
```

---

## 13. 重構執行 Plan（分階段）

### Stage 1：把三大章節搬到 `.claude/rules/`（1-1.5 小時）

**目標**：CLAUDE.md 從 271 行瘦到 ≤200 行（符合官方建議）。

1. `$cowork-workspace/.claude/rules/` 建立
2. 寫三個 rules 檔：
   - `skill-management.md` with `paths: ["Developer/skills/**", ".claude/skills/**"]`
   - `vault-operations.md` with `paths: ["stanley-vault/**"]`
   - `local-tools.md`（whisper——可含 paths 或不含）
3. `memory/README.md` 建立，搬「架構與設置指令」段落
4. CLAUDE.md 瘦身：
   - 保留：Me / People / Terms / Teams、Vault 路徑硬性規則、Memory 判斷標準、系統操作規則 11 條
   - 搬走：Obsidian Vault 細節、Skill 管理規則細節、Memory 系統細節、whisper
5. **備份原 CLAUDE.md** 為 `.bak-YYYYMMDD`（已於 2026-04-18 備份過）
6. 新對話測試三個 rules 是否正確 lazy load

#### 執行記錄 2026-04-18 08:30（Stanley 出門前授權 + Dispatch 討論後）

**完成項**：

1. ✅ 備份 `CLAUDE.md.bak-stage1-20260418-0828`（271 行原檔保留）
2. ✅ `$cowork-workspace/.claude/rules/` 目錄建立
3. ✅ `.claude/rules/vault-operations.md`（78 行）with `paths: ["stanley-vault/**"]` — 搬出原 CLAUDE.md 的資料夾結構、frontmatter schema、skill→資料夾對照、檔名規則、tag 軸線
4. ✅ `.claude/rules/skill-management.md`（76 行）with `paths: ["Developer/skills/**", ".claude/skills/**", "_dist/**"]` — 搬出原 CLAUDE.md 的三環境同步架構、檔案規格、完整更新流程、版號規則
5. ✅ `memory/README.md`（102 行）— 搬出原 CLAUDE.md 的 memory 架構表、symlink 設計哲學、設置命令、檔案格式 schema
6. ✅ CLAUDE.md 瘦身：**271 → 118 行（減 56%）**

**與原計畫的三個 deviation**：

| 項目 | 原計畫 | 實際執行 | 理由 |
|------|-------|---------|------|
| whisper.cpp 章節 | 搬到 `.claude/rules/local-tools.md` | **保留在 CLAUDE.md 骨幹（9 行）** | 太短，lazy load 的 overhead 不划算；偶爾用到也不是重複任務，留骨幹 |
| Skill 管理三條硬性禁令 | 全搬 rules | **骨幹保留三條硬性禁令**（寫 source of truth、禁手動 pack、禁手動 git），完整流程搬 rules | 這三條在任何 cwd 都可能被違反（paths 觸發不到），必須全域守住 |
| Memory 判斷標準（何時寫 / 何時讀） | 搬到 `memory/README.md` | **骨幹保留**，README 只放架構與設置 | 判斷標準是每次對話都可能用到的決策邏輯（80% 法則），不該 lazy；架構與設置是一次性資訊才 lazy |

**新增治理項**：

- 骨幹末尾新增 **「底層架構討論紀律」** 章節（3 行 + pointer），對應 memory `feedback_底層架構必先驗證.md`（本日踩了兩次雷後強化）

**Stanley 回來要做**（5 分鐘）：

1. 開新 Claude Code 對話（CLI 或 Cowork 都要測）
2. 測試 A：問「我要改一個 skill 的版號規則是什麼？」→ 預期 Claude 主動提到 skill-management.md rule 或直接答出 semver 規則
3. 測試 B：問「我要建一張新 vault 卡片，frontmatter schema 是什麼？」→ 預期 Claude 提到 vault-operations.md 或直接答出完整 schema
4. 測試 C：問「你的身份是？」→ 預期 Claude 答出 Stanley Cheng（骨幹 Me 章節仍在）
5. 若三題都 OK → Stage 1 成功
6. 若有問題 → 執行 rollback：`cp CLAUDE.md.bak-stage1-20260418-0828 CLAUDE.md`

#### 驗收結果 2026-04-18 晚上

Stanley 實測三題（CLI 新對話，cwd 為 `stanley-vault/`），**三題全 PASS**：

| # | 題目 | 結果 | 機制 | Bonus 觀察 |
|---|------|------|------|----------|
| A | vault 檔名規則？ | ✅ Claude 答出完整表格（卡片 / Journal / 週記 / 會議卡 / Email 草稿） | **Pointer-driven**：Claude 主動 Read `vault-operations.md` | Cite source「根據 .claude/rules/vault-operations.md」 |
| B | skill 版號規則？ | ✅ 答出 patch / minor / major + 寫入位置 + CHANGELOG 配套 | **Pointer-driven**：Claude 主動 Read `skill-management.md` | 補「版號放 `metadata:` 下否則 pack.py 驗證失敗」（骨幹硬性禁令也活著） |
| C | Peter 是誰？ | ✅ 秒答「Peter Chu / 直屬主管 / 一寸鮮主管 / Slack ID U04KHFCAZ1B」 | 骨幹直接可用，無需 Read | **主動應用**「Slack mention 用 `<@USER_ID>` 格式」規則，建議「要 @ 他用 `<@U04KHFCAZ1B>`」 |

**驗收洞察**：

1. **Pointer-driven 模式確認可行且優於預期**——詳見 Section 3 重要補充。Claude 看到骨幹 pointer 後主動 Read rule 檔，反而比 lazy load 更省 token（零啟動 context 佔用，按需讀取）
2. **骨幹「系統操作規則」活著**——測試 C 的 bonus 證實 Claude 不只「讀到」規則，還「主動應用」作為行為準則。這是瘦身後最重要的驗證
3. **Cite source 是 bonus**——Claude 主動說「根據 .claude/rules/xxx.md」，未來 debug 可直接追規則來源

**結論**：Stage 1 **✅ 蓋章完成**。備份 `CLAUDE.md.bak-stage1-20260418-0828` 留一週觀察期後可刪。Cowork 環境的驗收尚未執行——但 2026-04-18 上午 pilot 已確認 Cowork 跟 CLI 三個機制行為一致，預期 pointer-driven 模式也同步。

### Stage 2：user-level 分離（選做，先 park）

**前提**：先觀察一週自己從哪些 cwd 啟動 `claude`（在 CLAUDE.md 無法載入的 cwd，跨 workspace 通用資訊會遺失）。

若決定做：
1. `~/.claude/CLAUDE.md` 建立
2. 搬「跨 workspace 通用」內容（Me / Terms / 系統操作規則）
3. project-level CLAUDE.md 只留該 workspace 專屬

### Stage 3（選做）：CLAUDE.local.md 進 `.gitignore`

目前 `$cowork-workspace` 不是 git repo（2026-04-17 Stanley 決定用 Dropbox 版本控制 + CHANGELOG 代替 git）——本條 N/A，但記錄於此備查。若未來改用 git，記得加 `.gitignore`。

---

## 14. 官方文件來源（全部）

| 主題 | URL |
|------|-----|
| Memory / CLAUDE.md 完整文件 | https://code.claude.com/docs/en/memory.md |
| Skills | https://code.claude.com/docs/en/skills.md |
| Settings | https://code.claude.com/docs/en/settings.md |
| Permissions | https://code.claude.com/docs/en/permissions.md |
| Hooks | https://code.claude.com/docs/en/hooks.md |
| Overview（跨環境） | https://code.claude.com/docs/en/overview.md |

---

## 15. 實測記錄

### 2026-04-18 Pilot 測試

**材料**：
- `$cowork-workspace/.claude/test/pilot.md`（@import 口令：「紫色大象」）
- `$cowork-workspace/.claude/rules/test-paths-loading.md` with `paths: ["stanley-vault/test-trigger/**"]`（paths 口令：「橘色長頸鹿」）
- `$cowork-workspace/stanley-vault/test-trigger/trigger.md`（觸發檔）
- CLAUDE.md 底部加 `@.claude/test/pilot.md`

**結果**：

| 環境 | Test A（直接問 @import 口令） | Test B（Read trigger → 問 paths 口令） | Test C（無 Read 直接問 paths 口令） |
|------|---------------------------|-----------------------------------|--------------------------------|
| Claude Code CLI（`claude -p`） | ✅ 紫色大象 | ✅ 橘色長頸鹿 | ✅ 答不出 paths 口令（誤答成 @import 口令，確認 lazy） |
| Claude Code CLI with `--allowedTools Read(stanley-vault/test-trigger/**)` | — | ✅ 橘色長頸鹿（系統自動載入，非 Claude 自讀 rule） | — |
| Cowork（claude.ai） | ✅ 紫色大象 | ✅ 橘色長頸鹿 + Claude 自己詮釋機制 | ✅「rule 沒有被觸發，我的 context 裡沒有這個口令」 |

**結論**：CLI 跟 Cowork 對 `@import` / `.claude/rules/` + `paths:` / lazy load 三項機制**100% 一致**。官方文件在這兩個環境成立。

測試後已全部清理，CLAUDE.md 回到原始 271 行。

---

## 16. 待驗證（未來的 pilot）

1. **claude.ai web app**：是否支援 `@import` / `.claude/rules/`？（web app 可能根本讀不到本地檔案，需先釐清其檔案掛載機制）
2. **子目錄 CLAUDE.md lazy load 的精確觸發邊界**：Read / Edit / Grep / Glob 是否都觸發？還是只有某些 tool 會觸發？
3. **Cowork 對 user-level `~/.claude/CLAUDE.md` 的支援**：Cowork 是雲端沙盒，應該讀不到本地 `~/.claude/`——但官方 overview 說「files work across surfaces」，實際行為待實測
4. **Managed Policy 在 Cowork 的行為**：N/A，因為 Stanley 不是企業部署
5. **`.claude/rules/` paths lazy load 精確觸發邊界**（2026-04-18 Stage 1 揭露）：官方原文「rules only load into context when Claude works with matching files」——實測發現：
   - ✅ Claude 主動 Read 一個 matching path 的檔案後，rule 自動進 context（上午 pilot 驗證）
   - ❌ cwd 在 matching paths 底下但未 Read 任何 matching file，rule **不自動進 context**（Stage 1 驗收揭露——Claude 需要靠 CLAUDE.md 骨幹 pointer 才會主動讀 rule 檔）
   - ❓ 其他 file 操作（Grep / Glob / Bash cat）是否觸發 lazy load？尚未測
   - ❓ 一個 session 中 Read 了 matching file 後，rule 留在 context 多久？下次對話重置嗎？尚未測
   - 這個邊界不影響日常使用（pointer-driven 已 work），但若未來要設計更自動化的 rule triggering，需要先釐清

---

## 17. 相關卡片

- [[2-cards/Claude Code 四大模組 — Context × Rules × Reach × Operation\|Claude Code 四大模組 — Context × Rules × Reach × Operation]]（MKT1 方法論視角）
- [[2-cards/綠藤 AI 系統四層架構\|綠藤 AI 系統四層架構]]（組織級四層 vs 個人級六層配置）
- [[2-cards/How to build marketing systems in Claude Code\|How to build marketing systems in Claude Code]]（marketing-specific 應用）
- [[2-cards/Karpathy LLM Wiki — AI 全權維護的第二大腦\|Karpathy LLM Wiki — AI 全權維護的第二大腦]]（Context 層的極致）
- [[2-cards/Orchestrator — 把零件變成產品的最後一層\|Orchestrator — 把零件變成產品的最後一層]]（Operation 層的極致）

## 18. 長期永續治理原則

> 這四條原則是底層架構**隨時間演化**的護欄。跟第 10 節（核心使用原則）互補：第 10 節講「怎麼使用」，本節講「怎麼治理」。

### 原則 A：不變性金字塔（Pyramid of Immutability）

配置分層的第一原則——**越底層越穩定**。各層按「變化速度」排序：

```
        慢變 ←→ 快變
┌──────────────────────────────────────┐
│ 身分與永恆偏好（年級變化）              │ ← ~/.claude/CLAUDE.md
├──────────────────────────────────────┤
│ Workspace 結構與骨幹規則                │ ← $cowork/CLAUDE.md
├──────────────────────────────────────┤
│ 情境化操作紀律（vault / skill 等）      │ ← .claude/rules/ + paths:
├──────────────────────────────────────┤
│ 易腐狀態（當前焦點、通訊錄、活躍專案）    │ ← CLAUDE.local.md
├──────────────────────────────────────┤
│ Claude 累積學到的（踩坑、模式、偏好）     │ ← Auto Memory (MEMORY.md)
└──────────────────────────────────────┘
```

判斷一條規則該放哪層，問自己：**這條規則預期多久會變？**
- 十年後還成立 → user-level
- 這個 workspace 存在期間成立 → project-level
- 特定情境才適用 → `.claude/rules/` + paths
- 這週成立 → local.md
- 不確定 / Claude 自己發現 → 讓 auto memory 累積

### 原則 B：單一責任（Single Responsibility）

每層只負責一個面向，層間不重疊：

| 層級 | 唯一責任 | 不該放 |
|------|----------|--------|
| `~/.claude/CLAUDE.md` | 「我是誰」+ 永恆行為規則 | Workspace 術語、同事 People、易腐狀態 |
| `$cowork/CLAUDE.md` | 這個 workspace 的結構與骨幹紀律 | 跨 workspace 通用身分、個人偏好 |
| `.claude/rules/*.md` | 情境觸發的操作紀律 | 全場景規則 |
| `CLAUDE.local.md` | 個人 + 易腐 + 敏感 | 跨人分享也有意義的規則 |
| `memory/` | Claude 的跨 session 學習 | 使用者主動維護的結構化資料（放 local.md） |
| Skills | 按需執行的完整任務流程 | 每次都需要的規則 |

**反面測試**：如果要跟同事（如 Dora）分享整個 workspace，哪些層要抽掉？**只有 `CLAUDE.local.md` 應該被抽掉**，其他層照共享無妨。若發現別層也要抽掉才能分享，表示內容放錯層了。

### 原則 C：演化友善（Evolution-Friendly）

設計時預想三個未來情境，每次加規則前都問一次：

1. **個人使用 → 跟同事共享 workspace**：`CLAUDE.local.md` 保持私有，其他層可共用——這條規則放這層後，「共享情境」還合理嗎？
2. **單一 workspace → 多個 workspace**（例如加 `~/Developer/my-blog/`）：這條規則若在 user-level，其他 workspace 會受影響嗎？合理嗎？
3. **CLI → Cowork → web app**：這條規則在各環境行為一致嗎？（已驗證 `@import` / rules 在 CLI + Cowork 一致；web app 待驗）

**規則**：不要設計綁死在「當下 Stanley 的工作模式」的配置。

### 原則 D：事實驅動變更（Fact-Driven Changes）

底層改動前三步驟，缺一不可：

1. **查官方文件**（此卡第 14 節的 URL 是起點）
2. **實測驗證**（類似 2026-04-18 的三環境 pilot）
3. **寫入此卡**（Section 20 版本歷史；若涉及機制發現，補對應 section）

**拒絕**靠「Claude 在某對話裡說⋯」做底層決策——Claude 會過時、會記錯、會引用二手資訊。這就是 4/2-4/8 那四張舊卡片的毛病：寫得自信，但沒查官方，導致 `@import` 機制整個遺漏。

**遇到不確定時的 default action**：讀此卡 → 仍不確定 → 查官方 → 仍不確定 → 實測 pilot → 仍不確定 → 不動，park 起來等更多資訊。

**配套 memory**：`memory/feedback_底層架構必先驗證.md` — 這條紀律的 operational 版本（觸發主題清單、驗證方式優先順序、絕對禁止項目）。2026-04-18 Stanley 明確要求「所有底層架構的討論都先做確認」後建立。

---

## 19. 變更管理 SOP

**適用範圍**：任何對以下檔案的**結構性**變更——
- `~/.claude/CLAUDE.md`
- `$cowork/CLAUDE.md`
- `$cowork/CLAUDE.local.md` 的結構（非「當前焦點」更新）
- `.claude/rules/*.md`
- `memory/README.md`（memory 架構規則）

**不適用**：
- Skill 內容變更（走 `~/Developer/skills/` 的 push-and-pack 流程）
- memory 新增 / 更新內容（日常累積，不算結構變更）
- CLAUDE.local.md「當前焦點」段落（關機儀式自動處理）
- 修 typo / 單純改措辭（見下方「簡化 SOP」）

### 完整 SOP（8 步驟）

1. **動機記錄**：一句話——為什麼要改？（像寫 git commit message）
2. **影響評估**：這次變更會不會影響現有 skill / rules / memory / 其他 workspace？跟原則 A-C 有沒有衝突？
3. **讀此卡**：確認機制理解正確——特別是跨環境矩陣（Section 9）
4. **備份**：`cp <file> <file>.bak-YYYYMMDD`
5. **改動**：執行修改
6. **實測**：開新對話驗證（跨環境重要改動至少 CLI + Cowork 都測）
7. **CHANGELOG**：在 `$cowork-workspace/CHANGELOG.md` 記錄 `date | file | reason`
8. **若涉及機制發現**：更新此卡對應 section（不只 Section 20，對應機制也要補）

### 簡化 SOP（低風險變更）

- **修 typo / 改措辭**：改 + 寫一行 CHANGELOG，跳過實測
- **新增 memory 檔**：正常累積流程
- **`.claude/rules/` 檔內容小調**（paths 沒變，只改內文）：改 + 簡短實測 + CHANGELOG

### 強制走完整 SOP 的情境

- 新增 / 刪除 `.claude/rules/` 檔案
- 改 `paths:` frontmatter
- 改 `~/.claude/CLAUDE.md`（影響所有 workspace）
- 改檔案載入機制（例如首次引入某個 `@import` 結構）
- 任何實測未驗證的機制首次採用

### 誰啟動 SOP

- Stanley 自己改：照走
- Claude 代改：Claude 必須在動手前明確列出 SOP 對應步驟給 Stanley 看，取得同意後執行
- 對於「高風險」（例如改 user-level CLAUDE.md），**Claude 不主動動手**——只給建議 + diff，Stanley 自己執行

---

## 20. Context 注入光譜（Context Injection Spectrum）

> 這個光譜描述 Claude 在 runtime 被注入脈絡的方式，從完全靜態到完全自動化。Stanley 的工作流實際上跨越多個位置。

### 四格光譜

```
靜態              半動態              動態手動            自動化
(static)        (path-triggered)   (human-injected)   (agentic)
│                  │                   │                  │
CLAUDE.md +      .claude/rules/       Obsidian          MCP resources
 @import          + paths:            路徑貼 Terminal    (@mention 顯式選)
 (launch 展開)    (lazy load)          (Stanley 貼)       RAG pipeline
                                                         Research Agent
每次載入         按 path 匹配         按 Stanley 判斷     按系統觸發
                  觸發                                    （semantic / agentic）
```

### 每格的職責

| 格 | 機制 | 觸發 | 誰決定注入 |
|---|------|------|-----------|
| 1. 靜態 | CLAUDE.md + `@import` | 對話啟動 | 使用者事前寫好 |
| 2. 半動態 | `.claude/rules/*.md` with `paths:` | Claude 讀匹配 path 的檔案 | 使用者事前定義 paths glob |
| 3. 動態手動 | Obsidian → Terminal 貼路徑 | Stanley 判斷相關性 | 人每次判斷 |
| 4. 自動化 | MCP resources `@mention` / RAG / Agent | 依機制：`@mention` 是人選、RAG 是語意 match、Agent 是自主決策 | 系統（或人 + 系統混合） |

### Stanley 的現況（2026-04-18）

**已用**：1 + 3
- 1：CLAUDE.md / CLAUDE.local.md
- 3：Obsidian → Terminal 工作流（每天在用）

**規劃中**：2
- `.claude/rules/` Stage 1 重構已規劃（見 Section 13），尚未執行

**未用**：4
- MCP resources（`@mention` 顯式選擇）：Claude Code 支援，Stanley 尚未用這個 pattern 管理 vault 資源
- RAG：尚未架設（vault 2,663 張已超過手動管理閾值，但現狀 work，未必要急著上——詳見 Section 21）
- Research Agent：有用 Claude Code 內建 subagent（Explore / claude-code-guide），但沒專門的「vault research agent」

### 設計洞察：Graph × Tree 雙軌

Stanley 的工作流同時跑兩套結構：

```
┌─────────────────────────┐         ┌─────────────────────────┐
│  Knowledge = Graph      │         │  Config = Tree          │
│  Obsidian vault         │         │  CLAUDE.md 載入機制      │
│  - 雙向連結              │   ←→    │  - user-level           │
│  - MOC / tag            │         │  - project-level        │
│  - 語意關聯              │         │  - subdirectory lazy    │
│  - 2,663 檔案           │         │                         │
└─────────────────────────┘         └─────────────────────────┘
           ↑                                   ↑
           └─────── 手動橋接 ──────────────────┘
              （貼路徑到 Terminal）
```

**知識的組織邏輯是 graph**（不適合被 tree 結構綁死）——Obsidian 負責。
**配置與規則的組織邏輯是 tree**（階層、繼承）——CLAUDE.md 負責。

兩者透過人工貼路徑橋接。這是**有意義的設計選擇**，不是暫時 workaround。未來若要自動化（用 vault-rag MCP 取代手動橋接），兩層結構仍分離。

### MCP resources 的實際行為（重要修正）

本卡初期討論曾提「MCP server 可以透過 `@mcp://...` 自動拉資源進 context」——**這是錯誤資訊**（2026-04-18 驗證後修正）。

**官方原文**：

> Resources in MCP are designed to be **application-driven**, with host applications determining how to incorporate context based on their needs. Applications could: Expose resources through UI elements for explicit selection... Allow the user to search through and filter... Implement automatic context inclusion, based on heuristics or the AI model's selection.

來源：https://modelcontextprotocol.io/docs/concepts/resources/

**結論**：
- Claude Code 目前是**顯式 `@mention` 選擇模式**——不是 CLAUDE.md 自動拉取
- 官方有提到未來「automatic context inclusion」的可能性，但目前未實作
- 要做「自動化脈絡注入」，當前唯一路徑是**自建 RAG pipeline**（見 Section 21），不是指望 MCP resources 自動機制

---

## 21. RAG / Contextual Retrieval 官方方法論

> 當 Stanley 的 vault 成長超過手動管理的規模（當前 2,663 檔案），「自動化 context 注入」變得有價值。本節記錄 Anthropic 官方 RAG 做法，作為未來決策基礎。

### 大前提：Anthropic 沒有 embedding API

**官方原文**：

> Anthropic does not offer its own embedding model. One embeddings provider that has a wide variety of options and capabilities encompassing all of the above considerations is Voyage AI.

來源：https://platform.claude.com/docs/en/build-with-claude/embeddings

**結論**：Claude 負責「理解 + 回答」；embedding 必須用第三方或本地 model。

### Voyage AI 當前推薦 model（官方）

**官方原文**：

> For general-purpose embedding, the recommended models are: `voyage-3-large`: Best quality; `voyage-3.5-lite`: Lowest latency and cost; `voyage-3.5`: Balanced performance with superior retrieval quality at a competitive price point.

來源：https://platform.claude.com/docs/en/build-with-claude/embeddings

| Model | 定位 | 適用情境 |
|-------|------|---------|
| `voyage-3-large` | 最佳品質 | 成本不是問題、追求最高 retrieval 精準度 |
| `voyage-3.5` | 平衡 | **當前最推薦均衡選擇** |
| `voyage-3.5-lite` | 最低成本 / 延遲 | 大量 query 或 latency 敏感 |

### 其他 embedding 路徑

| 路徑 | 優缺 |
|------|------|
| **Voyage**（Anthropic 推薦） | 品質最高、為 retrieval 量身打造；需 API key + 資料送 Voyage 伺服器 |
| **OpenAI** `text-embedding-3-*` | 便宜、成熟；需 OpenAI API key + 跟 Claude 生態非對口 |
| **本地** bge-m3 / jina-embeddings-v3 | 免費、完全本地、隱私最高；需 Ollama / Python 環境 + 品質略低 |
| **Cohere** embed-multilingual-v3.0 | 強多語言、商業級；又一個 API key |

### Contextual Retrieval（Anthropic 官方 RAG pipeline）

Anthropic 2024/9 推出的方法論，**至 2026-04 仍為當前推薦**。

來源：https://www.anthropic.com/news/contextual-retrieval

#### 為什麼需要「contextual」

傳統 RAG 把 document 切成 chunk 後 embedding。問題：chunk 切出去就失去上下文——例如 Stanley vault 的「質靈×三日苗 Pipeline 續接卡」第三段單獨被 retrieve 出來，Claude 根本不知道這段在講什麼。

#### 四步法

```
Step 1 — 預處理
  每個 chunk 先用 Claude 生成「這個 chunk 的 context 描述」
  例：「這段是 PAPERSKY 編輯哲學卡片的第三段，討論地方做夢空間概念⋯」

Step 2 — 雙索引
  把 context prefix + chunk 合併後
  ├─ 算 embedding（向量索引）
  └─ 建 BM25（關鍵字索引）

Step 3 — 執行時融合
  Query 同時打兩路，融合結果

Step 4 — 可選 rerank
  用 Voyage Rerank 2 或 Cohere Rerank 3 精選 top-K
```

#### 官方效果數據（原文）

> Reducing retrieval failures by 49% alone, and by 67% when combined with reranking.

- 單獨用 contextual embedding：retrieval 失敗率減少 **49%**
- 加 rerank：減少 **67%**

### 對 Stanley vault 的 pipeline 建議（成本不限版）

```
Query
  ↓
Stage 1 — 粗檢索（撈 top 50）
  ├─ voyage-3-large embedding 向量搜尋（with Contextual prefix）
  └─ BM25 關鍵字搜尋
  ↓
Stage 2 — Re-ranking（撈出最佳 top 5）
  └─ Voyage Rerank 2
  ↓
注入 Claude context
```

### 成本估計（成本不限版）

| 項目 | 單價 | 對 2,663 檔案的一年成本 |
|------|------|----------------------|
| Voyage embedding（index） | $0.18/1M tokens | 初次 ~$2-3 + 週更新 ~$0.10 |
| Voyage embedding（query） | 同上 | 日查 10 次 ≈ $0.01 |
| Voyage Rerank | $0.05/1K queries | 日查 10 次 ≈ 年 $2 |
| Contextual prefix（Claude 幫寫） | Claude API pricing | 初次 ~$10-30（一次性） |
| **總年成本** | | **約 $20-40 美金** |

### 實作路徑選擇

1. **快速 pilot**（驗證 RAG 對你 vault 的價值）
   - Obsidian Smart Connections 外掛：30 分鐘裝好，在 Obsidian 側邊欄看相關卡片推薦
   
2. **長期方案**：自建 vault-rag MCP server
   - **本地版**：LanceDB + Ollama 跑 bge-m3（零成本、純本地、隱私最高）
   - **雲端最佳版**：Voyage `voyage-3-large` + Contextual Retrieval + Voyage Rerank（品質最高）

3. **整合進 skill**：morning-briefing / weekly-reflection / shutdown-ritual 自動查 vault

### 待決策點（未來做這題時回來看）

1. **隱私 vs 品質**：要不要讓 vault 內容送到 Voyage 伺服器算 embedding？本地 bge-m3 差距有多大，需要實測對比才知道
2. **觸發門檻**：現在做還是再等？vault 2,663 張已超過手動管理閾值，但現狀還 work，未必要現在動
3. **MCP server spec**：若要自建，先寫 `vault-rag MCP` 規格卡落地討論，不要直接動手寫 code

---

## 22. 版本歷史

| 日期 | 變更 |
|------|------|
| 2026-04-18 | 初版建立。合併 4 張舊卡片 + 官方文件 + Pilot 實測 |
| 2026-04-18 | 新增 Section 18（長期永續治理原則 A/B/C/D）+ Section 19（變更管理 SOP） |
| 2026-04-18 | Section 18 原則 D 加 pointer 指向 `memory/feedback_底層架構必先驗證.md` |
| 2026-04-18 | 新增 Section 20（Context 注入光譜）+ Section 21（RAG / Contextual Retrieval 含 Voyage 當前推薦模型 + Anthropic Contextual Retrieval 四步法）。修正 MCP resources 行為（app-driven / 非自動載入）。版本歷史從 Section 20 挪到 Section 22 |
| 2026-04-18 | **Stage 1 執行完成**：CLAUDE.md **271→118 行（減 56%）**。建 `.claude/rules/vault-operations.md` + `skill-management.md` + `memory/README.md`。三處 deviation（whisper / skill 硬性禁令 / memory 判斷標準留骨幹）已記錄。Section 11 現況快照更新。強化 memory `feedback_底層架構必先驗證.md`（Dispatch 踩第二次雷後 hard checkpoint） |
| 2026-04-18 | **Stage 1 驗收完成**：CLI 三題測試全 PASS。揭露實際日常機制是 **Pointer-driven Read**（CLAUDE.md 骨幹 pointer → Claude 主動 Read rule 檔）非 lazy load auto-trigger。Pointer-driven 反而更省啟動 context，且 Claude 會 cite source。Section 3 補「Pointer-driven vs Lazy load」區分。Section 13 加驗收結果表。Section 16 加 #5（paths lazy load 精確觸發邊界待驗證） |
| 2026-04-18 | 另建 [[2-cards/vault-rag MCP server 規格卡\|vault-rag MCP server 規格卡]]（2-cards/）作為 Stage 2 任務的交接文件，含技術決策樹 + Phase 1 setup guide 兩版本（Voyage 雲端 / bge-m3 本地） |
