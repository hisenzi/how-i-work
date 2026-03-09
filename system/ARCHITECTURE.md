# HiSenzi 系統架構

> 最後更新：2026-03-08（開案流程改造：內部/產品分類，ai2line + marketing-agent-pack 回補）
> Source of truth：`workspace/ARCHITECTURE.md` → 每日同步到 Obsidian `HiSenzi/系統架構.md`

---

## workspace/（HiSenzi 工作區）

| 檔案/資料夾 | 功用 |
|------------|------|
| `AGENTS.md` | 行為準則（每次都讀） |
| `SOUL.md` | HiSenzi 人格定義 |
| `USER.md` | Vincent 資訊 |
| `IDENTITY.md` | HiSenzi 身份（名字、Email） |
| `CORE.md` | 核心認知 L0（紅線、偏好，≤80行） |
| `MEMORY.md` | 長期記憶 L1（活躍專案，≤150行） |
| `HEARTBEAT.md` | 心跳檢查清單 |
| `TOOLS.md` | 工具設定筆記（日曆、Obsidian、專案管理指令） |
| `ARCHITECTURE.md` | 本檔案，系統架構速查 |
| `memory/` | 每日工作記憶（YYYY-MM-DD.md） |
| `scripts/` | 所有自動化腳本 |
| `skills/` | 所有 Skills + dist/ 歸檔 |
| `secrets/` | API Keys（.gitignore 保護，勿動） |
| `logs/` | backup.log 等執行紀錄 |
| `config/` | 系統設定（repos.json 等） |
| `milestones/projects/` | 專案定義（project.json + tasks.json） |

## HiSenzi/（Agent 產出區）

| 檔案/資料夾 | 功用 |
|------------|------|
| `dashboard.html` | 儀表板（每日 09:01 自動更新） |
| `backup-status.json` | 最近一次備份狀態 |
| `work-logs.json` | janitor 工作日誌 |
| `token-usage.json` | Token 用量統計 |
| `reviews/` | 月復盤報告（YYYY-MM-review.md） |

## cron/（排程系統）

| 檔案 | 功用 |
|------|------|
| `jobs.json` | 所有排程定義（schedule-cron Skill 管理） |
| `runs/` | 各 job 執行紀錄（自動產生） |

---

## Repo 清單

| Repo | 用途 | 管理方式 |
|------|------|---------|
| hisenzi/hisenzi-backup | 本地備份 + 里程碑 | backup.sh 22:05 自動 push |
| hisenzi/nsg-system | NSG 學習系統（產品，Zeabur） | check-repos.py 22:10 通知 |
| hisenzi/ai2line | ai2line 智慧業務預約系統（產品，Zeabur） | check-repos.py 22:10 通知 |
| hisenzi/openclaw-marketing-agent-pack | 行銷人的 AI Agent（產品，Zeabur） | check-repos.py 22:10 通知 |
| hisenzi/how-i-work | 公開 Digital Garden（內部，public） | 手動更新，不進 repos.json |

> backup-repo 由 backup.sh 獨立管理，不在 `config/repos.json` 中。
> repos.json 只管需要額外監控的 repo。

## 資料流

```
22:00 janitor        → 記憶清理 + README 里程碑
22:05 backup.sh      → push backup-repo + sync Obsidian
22:10 check-repos.py → 檢查 repos.json 裡的 repo
      → /tmp/hisenzi-repo-status.json → heartbeat 回報
```

## Repo 規則

- `deploy_target` 有值 → `auto_push: false`（只通知，不自動推）
- `deploy_target` 為 null → 可設 `auto_push: true`
- 新增 repo → 更新 `config/repos.json` + 本檔案
- backup-repo 不放 repos.json（backup.sh 已管理）

## 可清理

| 資料夾 | 說明 |
|--------|------|
| `HiHiSenzi_backup/` | 2/16 舊快照，已有 GitHub 備份，可刪 |
| `edu-system_closed/` | 會考系統早期原型，已歸檔（部署骨架已移入 nsg-system） |

## README.md 里程碑規則

- 每個獨立 repo：README.md 只顯示**自己的**專案里程碑
- hisenzi-backup：README.md 顯示**全部**專案里程碑（總覽）
- janitor 22:00 自動更新所有 repo 的 README.md
- 條件：repos.json 有登記 + README.md 有 `MILESTONES_START/END` 標記
- 開案時 `new-project.py --repo-path` 自動建含標記的 README.md

## 何時讀這個檔案

- 涉及 repo / 部署 / 架構 / Zeabur 時
- 開案時（project-init skill 會引導）
- 不需每次 session 啟動都讀
