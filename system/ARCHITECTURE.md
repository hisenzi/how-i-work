# HiSenzi 系統架構

> 最後更新：2026-03-13（五區塊重構 + Agent 架構 / 記憶系統 / 備份摘要）
> Source of truth：`workspace/ARCHITECTURE.md` → 每日同步到 Obsidian `HiSenzi/系統架構.md`

---

## 一、系統概覽（Agent 架構）

| Agent | Model | 角色 | Telegram |
|-------|-------|------|----------|
| 🌀 main / HiSenzi | Opus | 主腦：感知、規劃、決策 | @HiSenzibot |
| ✊ dosenzi / DoSenzi | Sonnet | 執行者：寫程式、跑 pipeline | @DoSenzi_bot |
| 🔍 whysenzi / WhySenzi | Sonnet | 研究者：調研、分析、比較 | @WhySenzi_bot |

**分工：** main 判斷 + 對話 → dosenzi 明確規格的執行 → whysenzi 查資料分析比較

> 完整細節：`docs/agent-architecture.md`

---

## 二、記憶系統（L0 / L1 / L2 三層）

| 層級 | 檔案 | 用途 | 維護方式 |
|------|------|------|---------|
| **L0** | `CORE.md` | 核心認知：紅線、偏好、系統設定 | 僅手動修改，≤80 行 |
| **L1** | `MEMORY.md` | 活躍工作區：專案 [P1] + 臨時 [P2] | 手動 + janitor 歸檔，≤150 行 |
| **L2** | `archive/` | 長期歸檔：過期的 P2、舊日誌摘要 | janitor 自動搬入 |

**每日記憶：** `memory/YYYY-MM-DD.md` — 當天工作日誌（raw notes）

**讀取規則：**
- 每次 session：讀 CORE.md（L0）
- 主對話 session：加讀 MEMORY.md（L1）+ 今天/昨天日誌
- 群組/共享 session：**不讀** MEMORY.md（隱私保護）

**維護流程：**
- 22:00 `memory-janitor.py` — 清理 L1 過期項目 → L2、更新 README 里程碑
- Heartbeat 定期 — 從日誌提煉洞見更新 MEMORY.md

> 行為規則細節：`CORE.md` / 記憶架構設計：`AGENTS.md`

---

## 三、每日自動化排程

### 每日

| 時間 | 任務 | Agent | 備註 |
|------|------|-------|------|
| 05:25 | 📖 NSG 學習提醒 | default | 週一～五 |
| 09:00 | ✅ 每日 HiSenzi 摘要 | main | isolated → Telegram |
| 11:55 | 📖 NSG 學習提醒 | default | 週一～五 |
| 15:25 | 📖 NSG 學習提醒 | default | 週一～五 |
| 19:55 | 📖 NSG 學習提醒 | default | 週一～五 |
| 20:25 | 📖 NSG 間隔複習提醒 | default | 週一～五 |
| 21:30 | 📋 未完成任務蒐集與排程規劃 | main | isolated → Telegram |
| 21:55 | 📝 記憶骨架 + 升版偵測 | main | |
| 22:00 | 🗂 memory-janitor 維護 | main | 記憶清理 + README 里程碑 |
| 22:01 | 📊 Dashboard 更新 | main | |
| 22:05 | 💾 OpenClaw 備份 + Obsidian 同步 | main | push backup-repo |
| 22:10 | 🔄 Repo 狀態檢查 | main | → /tmp/hisenzi-repo-status.json → heartbeat |
| 22:30 | 🔁 Heptabase → Obsidian 同步 | main | |

### 每週

| 時間 | 任務 | Agent |
|------|------|-------|
| 週日 21:00 | 📊 NSG 每週學習週報 | default |
| 週日 21:00 | 🔍 每週自動化缺口檢查 | main |

### 每月

| 時間 | 任務 | Agent |
|------|------|-------|
| 每月最後一天 23:00 | 📈 月復盤報告 | main |

### 手動觸發

| 任務 | 指令 |
|------|------|
| 🔐 Vault 備份 | `bash scripts/vault-backup.sh` |

### Heartbeat 檢查項（每日早上）

- 昨晚 backup.sh 是否成功（讀 log）
- Repo 狀態通知（/tmp/hisenzi-repo-status.json）
- Vault 備份提醒（敏感檔案更新 vs 上次備份時間）
- how-i-work 同步提醒（/tmp/hisenzi-how-i-work-sync.txt）
- Cadence 到期檢查（兩軌分流 ops task 的 last_done）
- 腳本完成通知（/tmp/hisenzi-annotate-result.json）

> 排程設定：`~/.openclaw/cron/jobs.json`（schedule-cron Skill 管理）
> 完整排程表：Obsidian `HiSenzi/維護規則.md`（自動產生，以此為準）

---

## 四、檔案結構

### workspace/（HiSenzi 工作區）

| 檔案/資料夾 | 功用 |
|------------|------|
| `AGENTS.md` | 行為準則（每次都讀） |
| `SOUL.md` | HiSenzi 人格定義 |
| `USER.md` | Vincent 資訊 |
| `IDENTITY.md` | HiSenzi 身份（名字、Email） |
| `CORE.md` | 核心認知 L0（紅線、偏好，≤80行） |
| `MEMORY.md` | 長期記憶 L1（活躍專案，≤150行） |
| `HEARTBEAT.md` | 心跳檢查清單 |
| `TOOLS.md` | 工具設定筆記 |
| `ARCHITECTURE.md` | 本檔案 |
| `BACKUP-RESTORE.md` | 備份與災難恢復完整指南 |
| `docs/` | 架構文件（agent-architecture.md 等） |
| `memory/` | 每日工作記憶（YYYY-MM-DD.md） |
| `archive/` | L2 長期歸檔 |
| `scripts/` | 所有自動化腳本 |
| `skills/` | 所有 Skills + dist/ 歸檔 |
| `secrets/` | API Keys（.gitignore，勿動） |
| `logs/` | backup.log 等執行紀錄 |
| `config/` | 系統設定（repos.json 等） |
| `milestones/projects/` | 專案定義（project.json + tasks.json） |
| `clients/` | 客戶資料（訓練套件、內部備註） |

### HiSenzi/（Agent 產出區）

| 檔案 | 功用 |
|------|------|
| `dashboard.html` | 儀表板（每日 09:01 自動更新） |
| `backup-status.json` | 最近一次備份狀態 |
| `work-logs.json` | janitor 工作日誌 |
| `token-usage.json` | Token 用量統計 |
| `reviews/` | 月復盤報告 |

### Repo 清單

| Repo | 用途 | 管理方式 |
|------|------|---------|
| hisenzi/hisenzi-backup | workspace 備份 | backup.sh 22:05 自動 push |
| hisenzi/nsg-system | NSG 學習系統 | check-repos.py 22:10 通知 |
| hisenzi/ai2line | 智慧業務預約系統 | check-repos.py 22:10 通知 |
| hisenzi/openclaw-marketing-agent-pack | 行銷人的 AI Agent | check-repos.py 22:10 通知 |
| hisenzi/hisenzi-website | hisenzi.com 靜態網站 | check-repos.py 22:10 通知 |
| hisenzi/hisenzi-dashboard | 儀表板（暫停） | check-repos.py 22:10 通知 |
| hisenzi/openclaw-training-kit | Agent 訓練套件 | check-repos.py 22:10 通知 |
| hisenzi/how-i-work | 公開 Digital Garden | 手動更新 |

### Repo 規則

- `deploy_target` 有值 → `auto_push: false`（只通知，不自動推）
- `deploy_target` 為 null → 可設 `auto_push: true`
- 新增 repo → 更新 `config/repos.json` + 本檔案
- backup-repo 不放 repos.json（backup.sh 已管理）

---

## 五、備份與災難恢復

```
三層備份：
1. OpenClaw 主程式  → npm（不需備份，重裝即可）
2. Workspace 資料   → Git → GitHub private repo（每日 22:05 自動）
3. 敏感資料         → AES-256 加密 → iCloud + 本機（手動觸發）
```

**災難恢復流程（15 分鐘）：**
Apple ID 登入 → iCloud 同步 vault → `vault-restore.sh` 恢復金鑰 → git clone repos → 安裝 OpenClaw → 啟動驗證

**Single Point of Failure：** Apple ID 密碼（唯一需要記住的東西）

> 完整 SOP 與細節：`BACKUP-RESTORE.md`

---

## 何時讀這個檔案

- 涉及 repo / 部署 / 架構 / Zeabur 時
- 開案時（project-init skill 會引導）
- 不需每次 session 啟動都讀
