# System Architecture

> AI Agent workspace 的完整架構。
> 這是從我的實際工作環境摘取的結構說明。

---

## Workspace 結構

| 檔案/資料夾 | 功用 |
|------------|------|
| `AGENTS.md` | Agent 行為準則（每次 session 都讀） |
| `SOUL.md` | Agent 人格定義 |
| `USER.md` | 使用者資訊（私有） |
| `CORE.md` | 核心認知 L0（紅線、偏好，≤80行） |
| `MEMORY.md` | 長期記憶 L1（活躍專案，≤150行） |
| `HEARTBEAT.md` | 定期心跳檢查清單 |
| `TOOLS.md` | 工具設定筆記 |
| `memory/` | 每日工作記憶（YYYY-MM-DD.md） |
| `scripts/` | 所有自動化腳本 |
| `skills/` | 自製 Agent Skills |
| `secrets/` | API Keys（.gitignore 保護） |
| `milestones/projects/` | 專案定義（project.json + tasks.json） |

## 記憶系統（雙軌）

```
L0: CORE.md        ← 核心規則，≤80行，僅手動修改
L1: MEMORY.md      ← 活躍工作區，≤150行，janitor 自動維護
L2: archive/       ← 歸檔（過期 P2 項目、月度洞察）
    memory/        ← 每日筆記（原始紀錄）
```

Agent 每次啟動：讀 CORE.md → 讀 MEMORY.md → 讀最近 2 天的 memory/。
memory-janitor.py 每天 22:00 自動清理過期項目、驗證行數上限。

## 備份系統（三層）

```
Layer 1: Git         ← workspace repo，每日 22:05 自動 push（私有）
Layer 2: Vault       ← secrets + SSH + 設定檔 → AES-256 加密 → iCloud
Layer 3: Cron        ← 排程 jobs.json 快照，跟著 Git 走
```

詳見 [BACKUP-RESTORE.md](BACKUP-RESTORE.md)。

## 每日自動化流程

```
21:55  auto-daily-log.py    → 產生當日記憶骨架
22:00  memory-janitor.py    → 記憶清理 + README 里程碑更新
22:05  backup.sh            → git commit + push + sync Obsidian
22:10  check-repos.py       → 檢查外部 repo 狀態 → heartbeat 回報
22:30  heptabase-to-obsidian → 知識庫同步
09:00  每日摘要              → Agent 整理昨日重點
09:01  generate-dashboard    → 儀表板 HTML 更新
```

## 設計原則

1. **一個資料夾 = 一個專案** — 不在 repo 之間複製檔案
2. **Single source of truth** — 每份資料只有一個權威來源
3. **先跑通，再自動化** — 手動驗證 OK 才寫成腳本 / cron
4. **文件即記憶** — Agent 不靠「心裡記住」，寫下來才算數
5. **安全分層** — 敏感資料永遠不進 git，用 vault 加密異地備份
