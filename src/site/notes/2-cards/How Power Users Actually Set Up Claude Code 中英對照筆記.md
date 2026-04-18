---
{"dg-publish":true,"permalink":"/2-cards/how-power-users-actually-set-up-claude-code/","tags":["主題/AI與知識工作流","中英對照","clipping-notes"],"dg-note-properties":{"created":"2026-04-18","updated":"2026-04-18","tags":["主題/AI與知識工作流","中英對照","clipping-notes"],"source":"Kieran Flanagan 2026-04-17 — https://www.kieranflanagan.io/p/how-power-users-actually-set-up-claude"}}
---


# How Power Users Actually Set Up Claude Code — 中英對照筆記

> Kieran Flanagan 2026-04-17 的 Substack 專欄,提出 Claude Code 配置的四大模組：**Context / Rules / Reach / Operation**。
>
> 本卡逐段中英對照整理,保留原文語氣與 code 範例,方便未來 cite 與跨語言對照使用。比對 SOT 卡的差異分析見 [[2-cards/Kieran Flanagan 四大模組 × Source of Truth 比對分析\|Kieran Flanagan 四大模組 × Source of Truth 比對分析]]。
>
> 原 clipping：[[4-clippings/How Power Users Actually Set Up Claude Code\|How Power Users Actually Set Up Claude Code]](4-clippings/)

---

## § 0 NUMMI 故事 — 基礎設施比人選重要

### [EN] Original

In 1982, GM shut down its factory in Fremont, California. The workers there were, by their own union's admission, the **worst** in the American auto industry. Absenteeism ran at 20%. Cars rolled off the line with missing parts. Workers left empty bottles inside door panels to rattle and annoy future owners. GM closed it down.

Two years later, Toyota and GM reopened the exact same factory as a joint venture called NUMMI. Same building. Same equipment. They rehired over 85% of the original workforce, including many of the people specifically cited for the strikes and the sabotage.

Within one year, the plant matched Toyota's best factory in Japan for quality. Absenteeism fell from 20% to 2%. The same workers GM had written off were producing some of the best cars in North America.

The only thing that changed was what Toyota put around them before they started work. Toyota sent workers to Japan for three weeks before the factory opened. Not to teach them to work harder, to give them context, rules, tools, and a system for how work actually gets done. GM had given them none of that. Toyota gave them everything.

That's the right approach to turn Claude Code into a powerhouse of a tool for your work. To do it you need a starter pack that includes *Context, Rules, Reach, Operations*.

### [中] 翻譯

1982 年,GM 關掉了位於加州弗里蒙特的工廠。工會自己承認,那裡的工人是美國汽車業**最糟的**。缺勤率 20%。出廠的車缺零件。工人故意把空瓶子塞進門板,讓未來車主聽到喀啦喀啦的雜音。GM 關門了事。

兩年後,Toyota 與 GM 合資,以「NUMMI」之名重啟**同一棟廠房、同一批設備**。他們重新僱用了超過 85% 的原班人馬,包括那些罷工、搞破壞時被點名的工人。

一年之內,這間工廠的品質追上 Toyota 在日本的最佳工廠。缺勤率從 20% 降到 2%。同一群被 GM 放棄的工人,產出北美最好的車。

唯一改變的,是 Toyota 在他們開工前為他們建好的一切。Toyota 送工人到日本三週,不是教他們「更努力工作」,而是給他們 Context(上下文)、Rules(規則)、Tools(工具),以及「工作究竟怎麼完成」的 System(系統)。GM 什麼都沒給。Toyota 給了全部。

把 Claude Code 變成工作利器,正該是這個路徑。你需要一份 starter pack,包含 **Context、Rules、Reach、Operations** 四模組。

---

## § 1 Context — Claude 需要知道什麼

### [EN] Original

Foundation files are `.md` files that live in a `/foundation/` folder in your project. Claude reads them before any task runs.

These are different from CLAUDE.md — that's the next section. Foundation files are the business intelligence layer. What Claude knows about your audience, your voice, your market, and how your customers buy.

**Audience Delight Profile.** How your audience actually talks. The words they use in Reddit threads and sales calls. When Claude knows your audience says "slop" not "low-quality output," and "ship it" not "deploy the solution," everything it writes sounds like a peer instead of a vendor.

**Creator Style.** Extracted from your best-performing content. Patterns, not adjectives. Not "we're conversational" — but "short sentences lead, fragments are welcome for punch, always lead with what the reader can do." Something Claude can check a draft against.

**Market Positioning Map.** A living file. What territory you own, what's contested, what you've ceded. Updated monthly. Every skill that touches messaging reads it automatically.

**Customer Journey Intelligence.** Where people actually stall and convert. If your nurture sequence doesn't know the specific moment people drop off, it writes emails that address the wrong problem.

One thing worth knowing about how this works: skills don't load all four files every time. Each skill reads the file headers and loads only what's relevant. A LinkedIn skill loads Audience Delight and Creator Style. A launch messaging skill loads Market Positioning and Customer Journey.

One thing the foundation post doesn't cover: these files decay. Your positioning shifts, a competitor reframes their messaging, your audience picks up new vocabulary. A foundation file that's six months out of date is actively misleading Claude. Build a scheduled skill that audits them monthly.

### [中] 翻譯

Foundation files 是放在專案 `/foundation/` 資料夾下的 `.md` 檔。Claude 在任何任務執行前都會先讀它們。

這跟 CLAUDE.md 不同(CLAUDE.md 是下一節的主題)。Foundation 檔案是**商業情報層**——Claude 對你的受眾、你的聲音、你的市場、客戶購買路徑的理解。

**Audience Delight Profile(受眾樣貌)**:受眾實際怎麼說話。他們在 Reddit 串、業務電話裡用的字眼。當 Claude 知道你受眾說「slop(糟糕貨)」而不是「low-quality output」、說「ship it(出貨吧)」而不是「deploy the solution」,寫出來的東西才像同溫層的同伴,不像賣你東西的廠商。

**Creator Style(創作者風格)**:從你最高績效的內容抽出來。是**模式,不是形容詞**。不是「我們很口語」,而是「短句開頭、允許片段增強衝擊力、永遠從讀者能做的事切入」。是 Claude 可以拿來比對草稿的東西。

**Market Positioning Map(市場定位地圖)**:動態檔案。你佔據的領域、競爭中的領域、已放棄的領域。每月更新。每一支碰到 messaging 的 skill 都自動讀。

**Customer Journey Intelligence(客戶旅程情報)**:人們實際卡關與轉化的節點。如果你的 nurture 序列不知道流失發生在哪個具體時刻,寫出來的 email 就在處理錯的問題。

關於運作機制:skills 不是每次都載入四支檔案。每個 skill 讀檔頭,只載相關的。LinkedIn skill 載 Audience Delight 與 Creator Style。產品發表 skill 載 Market Positioning 與 Customer Journey。

有一件事原文 foundation post 沒提:**這些檔案會腐敗**。你的定位會變、對手 reframe、受眾開始用新詞彙。一支六個月沒更新的 foundation 檔案,正在積極誤導 Claude。要建一支排程 skill,每月審計。

### [EN] Code Example — Audit Foundation Files Skill

```text
# Skill: Audit Foundation Files

## Steps
1. Read all files in /foundation/
2. For each file, check against live external sources:
   - Audience Delight: scan 3 competitor comment sections.
     Flag any new vocabulary not in the file.
   - Positioning Map: check competitor homepages.
     Flag any new claims that overlap with your owned territory.
   - Creator Style: check your 3 most recent published pieces.
     Flag any patterns that have drifted from the file.
3. Produce a short audit report — what's current, what's stale, what needs updating
4. Save report to /foundation/audit-[date].md
5. Agent to extract patterns, learnings and updates
6. Apply those to .md files in /foundation/
```

---

## § 2 Rules — Claude 應該怎麼行動

### [EN] Original

Two files do this job.

**CLAUDE.md** is your operating manual. The rules Claude follows in every session, what you're working on this quarter, what voice and tone all output defaults to, where the foundation files live, what never to do.

Keep it under 60 lines. [HumanLayer's research](https://humanlayer.dev/blog/writing-a-good-claude-md) on instruction budgets is clear: beyond a certain length, compliance drops. Shorter and specific beats longer and comprehensive.

`settings.json` tells Claude what it cannot touch — your `.env` file, your secrets folder, any bash command that could cause damage. This is particularly important when you start to sync local file systems across team members so everyone has a single context layer for Claude Code.

### [中] 翻譯

兩支檔案負責這件事。

**CLAUDE.md** 是你的 operating manual(操作手冊)——Claude 每個 session 都會跟的規則:你這季在做什麼、所有輸出預設的 voice & tone、foundation 檔案放在哪、絕對不要做什麼。

**守住 60 行以下**。HumanLayer 研究 instruction budget 很清楚:超過某個長度,合規性會下降。**短而具體,打敗長而全面**。

`settings.json` 告訴 Claude 什麼碰不得——你的 `.env`、secrets 資料夾、任何可能造成破壞的 bash 指令。當你開始跨團隊成員同步 local file system、讓大家共享同一層 context 時,這件事特別關鍵。

### [EN] Code Example — CLAUDE.md Template (marketing leader)

```text
# CLAUDE.md

## Who I am
[Name], [role] at [Company]. We [what the company does] for [who you serve].
Our stage: [e.g. Series B, 120 people, $18M ARR, expanding into enterprise].

## My marketing function
Team size: [X people]. Channels we own: [e.g. content, paid, lifecycle, PLG].
What we're responsible for: [pipeline, revenue, brand, product adoption — be specific].

## This quarter's priorities
- [Priority 1 — e.g. "Launch into mid-market segment, first campaign live by May"]
- [Priority 2 — e.g. "Reduce CAC by 20% through better top-of-funnel qualification"]
- [Priority 3 — e.g. "Build a content engine that generates 30% of inbound pipeline"]

## Where to find context
All foundation files live in /foundation/. Before any task, scan file headers
and load only files relevant to the work at hand.
- Audience and ICP context: /foundation/audience-delight.md
- Voice and tone: /foundation/creator-style.md
- Competitive landscape: /foundation/market-positioning.md
- How buyers actually move: /foundation/customer-journey.md

## How I work
- I think in outcomes, not outputs. Always tie recommendations to pipeline or revenue impact.
- I want strategic options, not one answer. Give me 2-3 approaches with tradeoffs.
- Flag assumptions. If you're working with incomplete data, say so.
- Be direct. No preamble, no summaries of what you're about to do — just do it.

## My key metrics
- [e.g. MQLs, SQLs, CAC, pipeline coverage, NRR — list the ones that matter to you]

## Current tools and stack
- CRM: [e.g. HubSpot]
- Analytics: [e.g. GA4, Amplitude]
- Ads: [e.g. Google, LinkedIn]
- Content: [e.g. Webflow, Substack]

## Never
- Recommend tactics without connecting them to a metric
- Give me a single option when tradeoffs exist
- Use corporate filler: "leverage", "synergies", "best-in-class", "move the needle"
- Summarise what you're about to do — just do it
```

### [EN] Code Example — settings.json deny list

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./secrets/**)",
      "Bash(curl *)",
      "Bash(rm -rf *)"
    ]
  }
}
```

### [EN] Folder Structure

```
/foundation/          ← context layer
/campaigns/           ← one folder per campaign
/content/
  /linkedin/
  /newsletter/
  /blog/
/research/
/.claude/
  /skills/
```

### [中] 資料夾結構說明

按工作流程組織,按工作實際在專案裡移動的路徑組織。`/foundation/` 是 context 層、每個 campaign 一個資料夾、內容依通路分、研究產出獨立放、`.claude/skills/` 放技能。

---

## § 3 Reach — Claude 可以連到什麼

### [EN] Original

Without MCPs, Claude only knows what's in the conversation, what's in your terminal, what you've described, copied, or typed. MCPs give Claude access to live data across your stack, which changes the quality of everything it produces.

For your power-user Claude Code setup, there are 2 MCP buckets to connect.

**a. Better marketing decisions, faster by connecting your live marketing stack**

Connect your CRM, analytics, and ad platforms, and Claude stops giving you strategic advice based on your description of the situation. It reads the situation itself.

Three prompts that show what this looks like in practice:

> *"Pull all deals that haven't moved in 21 days and summarise what's stalling by stage and suggest plays to unblock them."* → HubSpot MCP.
>
> *"Which of our active paid campaigns has a CAC above target this month?"* → Google Ads MCP.
>
> *"Go to [competitor]'s pricing page, capture what's there, compare it against the snapshot saved in /research/competitor-snapshots/, and flag anything that's changed."* → Playwright MCP.

Useful MCPs: HubSpot MCP (free for all HubSpot customers), GA4, Google Search Console, Playwright MCP for competitor monitoring, Slack for routing outputs to your team.

**b. Hear what your customers are actually saying**

Most companies are sitting on a goldmine of unstructured customer data they never have time to properly read. Sales call transcripts. Support tickets. Win/loss interviews. NPS responses. G2 and Trustpilot reviews. Every one of those is a customer telling you exactly what they think, in their own words.

Connect Claude Code to those sources and you can turn unstructured customer data into data that can help power your marketing & growth strategy. Instead of reading a sample of calls and hoping it's representative, Claude reads all of them. Instead of guessing on what objections are killing deals, you get a ranked list from the last 90 days of transcripts.

The questions that become answerable: What objections come up most in deals we lose versus deals we win? What do customers say in their first 30 days that predicts whether they'll expand or churn? What language do our happiest customers use that we're not using in our own marketing?

Key connectors: Notion or Google Drive MCP for call transcripts and research docs, Gong or Fireflies if your call recording tool has an MCP, web scraping via Playwright MCP for review sites.

### [中] 翻譯

沒有 MCP,Claude 只知道對話裡的東西、terminal 裡的東西、你描述/複製/打出來的東西。MCP 讓 Claude 能接觸你整個 stack 的即時資料,這會**改變它產出的所有東西的品質**——建議是基於你 business 現在**實際發生的事**,而不是你的描述。

Power-user 的 Claude Code 配置,要連兩桶 MCP:

**a. 連上即時行銷 stack,讓行銷決策更快更準**

連上 CRM、分析、廣告平台後,Claude 不再基於你對情境的描述給策略建議。它**自己讀情境**。

三個實際 prompt 例子:

> *「把過去 21 天沒動的所有 deal 撈出來,按 stage 總結卡在哪、建議怎麼解。」* → HubSpot MCP
>
> *「我們 active 的付費 campaign,這個月哪些 CAC 超過目標?」* → Google Ads MCP
>
> *「去 [對手] 的 pricing 頁面抓快照,跟 `/research/competitor-snapshots/` 裡的舊快照比對,flag 任何改變。」* → Playwright MCP

推薦 MCP:HubSpot MCP(所有 HubSpot 客戶免費)、GA4、Google Search Console、Playwright MCP 做對手監控、Slack 把產出路由給團隊。

**b. 聽顧客實際在說什麼**

大部分公司坐擁一座**非結構化顧客資料金礦,卻從來沒時間好好讀**。銷售電話逐字稿、客服工單、Win/Loss 訪談、NPS 回覆、G2 / Trustpilot 評論——每一個都是顧客用他們自己的話告訴你他們在想什麼。

把 Claude Code 連上那些來源,你就能把非結構化顧客資料轉成行銷 & 成長策略的燃料。不是讀一個 sample 然後希望它 representative——Claude 讀全部。不是猜什麼異議在殺 deal——你拿到過去 90 天逐字稿的排序清單。

變得可回答的問題:「我們輸掉的 deal 跟贏下的 deal 裡,哪些異議最常出現?」「顧客在前 30 天說的哪些話,能預測他之後會擴張還是 churn?」「我們最開心的顧客用什麼語言形容我們,而我們自己的行銷裡沒用到?」

關鍵連接器:Notion / Google Drive MCP 放 call transcript 與研究 doc、Gong / Fireflies(如果你的通話錄音工具有 MCP)、用 Playwright MCP 爬評論網站。

---

## § 4 Operation — 讓它成為利器的關鍵

### [EN] Original

Skills are your team. Each one is a SKILL.md file, a structured instruction set that tells Claude exactly what to do, which foundation files to read, and where to save the output. Where a prompt is a one-off request, a skill is a repeatable hire.

How do you decide what skills to build? **Let Claude surface what to automate.**

The best way to know what to turn into a skill is to look at what you actually do. Two skills make this work together.

The first runs at the end of every session. Before you close Claude Code, run the **End of Session Brief** — it captures what you worked on, what prompts you ran, and what outputs were produced, then saves a timestamped file to `/foundation/briefs/`.

Over time, `/foundation/briefs/` becomes a real record of how you actually use Claude Code. The second skill reads it.

Run **Find Skills** every Friday. In a month you'll have a system that surfaces its own gaps.

### [中] 翻譯

**Skills 是你的團隊**。每一支是一個 `SKILL.md`——結構化的指令集,告訴 Claude 要做什麼、讀哪些 foundation 檔案、把產出存到哪裡。如果說 prompt 是一次性請求,**skill 是可重複僱用的員工**。

怎麼決定建哪些 skill?**讓 Claude 自己浮出該自動化的項目**。

最好的判斷方式是看你實際在做什麼。兩支 skill 合作完成這件事。

第一支在每個 session 結束時跑。關掉 Claude Code 前,跑 **End of Session Brief**——抓下你剛做了什麼、跑了哪些 prompt、產出了哪些東西,存成加時戳的檔案到 `/foundation/briefs/`。

一段時間後,`/foundation/briefs/` 會變成你**實際怎麼用 Claude Code 的真實紀錄**。第二支 skill 讀它。

**Find Skills** 每週五跑。一個月後,你會有一個**自己揪自己缺口**的系統。

### [EN] Code Example — Weekly Pipeline Review Skill

```text
# Skill: Weekly Pipeline Review

## When to use
Run every Monday morning for a clear picture of pipeline health
and where to focus this week.

## Steps
1. Load foundation context
   Read /foundation/audience-delight.md for ICP criteria.
   Read CLAUDE.md for current quarter priorities and target metrics.

2. Pull pipeline data via HubSpot MCP
   - All open deals by stage
   - Deals with no activity in the last 14 days
   - Deals closing this month vs. target
   - Any stage regressions in the last 7 days

3. Produce the review
   - Pipeline coverage vs. target (are we on track?)
   - Deals at risk — stalled, regressed, or closing soon with low activity
   - Recommended actions by deal — what needs to happen this week
   - One-line summary for leadership: are we on track this quarter?

4. Save to /research/pipeline-review-[date].md
   Post summary to #revenue-team in Slack
```

### [EN] Code Example — End of Session Brief Skill

```text
# Skill: End of Session Brief

## When to use
Run before closing Claude Code at the end of any working session.

## Steps
1. Review the current session
   - What tasks were completed?
   - What prompts were run more than once?
   - What outputs were produced and where were they saved?
   - What worked well? What needed multiple attempts?

2. Write a structured brief covering:
   - Session date and duration
   - Tasks completed (one line each)
   - Repeated prompts — exact descriptions of anything run more than once
   - Outputs produced and their file locations
   - Any friction points or tasks that felt manual

3. Save to /foundation/briefs/[YYYY-MM-DD-HH-MM].md
```

### [EN] Code Example — Find Skills Skill

```text
# Skill: Find Skills

## When to use
Run weekly to surface patterns worth turning into skills.

## Steps
1. Read /foundation/briefs/reviewed.md to get the list of
   already-reviewed briefs. If the file doesn't exist, create it.

2. Scan /foundation/briefs/ for any briefs not in reviewed.md

3. For each unreviewed brief, look for:
   - Any prompt or task marked as repeated
   - Any task that required multiple attempts
   - Any output that was manually reformatted after saving

4. Identify patterns across briefs — tasks that appear in
   more than one session are the strongest candidates

5. For each candidate skill:
   - Name the skill
   - Write a first-draft SKILL.md with steps, inputs, and outputs
   - Estimate weekly time saved if automated

6. Present candidates with their draft SKILL.md files
   Append reviewed brief filenames to /foundation/briefs/reviewed.md
```

---

## § 5 結尾 — 基礎設施先於 prompt

### [EN] Original

Toyota didn't send those workers to Japan to motivate them. They sent them to give them everything they needed to do the job well before the job started — the context, the rules, the tools, the system.

When context, rules, reach, and operation are all in place, the ordinary becomes powerful. Everyone who sets up Claude Code rushes into prompts. There is real value in building the scaffolding: a `/foundation/` folder with your `.md` files and a 20-line CLAUDE.md at the project root, default MCPs and your starter skills package.

*Until Next Time, Happy AI'fying* — Kieran

### [中] 翻譯

Toyota 不是送那些工人去日本**激勵**他們。是送他們去拿到「把工作做好的所有條件」——在工作開始之前。Context、Rules、Tools、System。

當 Context、Rules、Reach、Operation 都到位,**平凡變得強大**。每個配置 Claude Code 的人都急著衝 prompt,但真正有價值的是先搭鷹架:一個 `/foundation/` 資料夾裝你的 `.md` 檔、一個專案根目錄 20 行的 CLAUDE.md、預設 MCP、起手技能包。

*下次見,Happy AI'fying* — Kieran

---

## 回應欄位(我的閱讀反應,非原文)

- ++CLAUDE.md 60 行的數字遠低於我 Stage 1 後的 118 行++ → 下一步要驗證:HumanLayer 的研究是基於哪個模型世代?Opus 4.7 的 1M context 是否改變 compliance 曲線?待查
- ++Find Skills 元技能我沒有++ → 關機儀式接近 End of Session Brief,但缺「每週自動提案新 skill」那層。值得建立
- ++`/foundation/` 作為獨立 context 層很乾淨++ → 我的 vault `2-cards/` + `3-MOC/` + `memory/` 分工其實就是 foundation 的超集,但**沒有 Kieran 這麼明確的「商業情報層」分類**(Audience / Style / Positioning / Journey)
- ++「檔案會腐敗」的警告很重要++ → 我的 vault 沒有「6 個月沒更新自動 flag」的機制,館長 skill 可以補

## 相關卡片

- [[2-cards/Claude Code 指令系統 Source of Truth\|Claude Code 指令系統 Source of Truth]] — 我的底層架構 SOT,比對分析見下條
- [[2-cards/Kieran Flanagan 四大模組 × Source of Truth 比對分析\|Kieran Flanagan 四大模組 × Source of Truth 比對分析]] — 本卡的後續分析(本次建立)
- [[2-cards/Claude Code 四大模組 — Context × Rules × Reach × Operation\|Claude Code 四大模組 — Context × Rules × Reach × Operation]] — 2026-04-17 既有的四模組概念卡(MKT1 Get 筆記版),跟本卡互補:那張是摘要 + 現況對照,本卡是逐段中英對照原文
- [[2-cards/Kieran 五原則深度解讀 — 從個人 Skill 走向團隊系統\|Kieran 五原則深度解讀 — 從個人 Skill 走向團隊系統]] — Kieran 其他文章的深度解讀
