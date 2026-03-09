# ADR 005: 三層記憶架構

## 狀態

已採用（2026-03-09）

## 背景

AI Agent 每次 session 醒來什麼都不記得。需要一個持久化的記憶系統，但面臨兩難：

- 記太少 → 失去脈絡，每次重來
- 記太多 → 檔案膨脹，token 爆炸，啟動變慢

之前只有一個 `MEMORY.md` 塞所有東西，很快就超過管理上限。

## 決策

採用三層記憶架構，每層有明確職責和容量上限：

```
L0  CORE.md      — 核心認知（80 行上限，永不自動修改）
L1  MEMORY.md    — 活躍工作區（150 行上限，索引式）
L2  archive/     — 歸檔（無上限，自動搬運）
```

輔助層：
- `memory/YYYY-MM-DD.md` — 每日 raw log（原始紀錄）
- Obsidian vault — 知識本體（SOP、架構文件、專案詳情）

### 各層規則

**L0 CORE.md**
- 抽象規則、紅線、長期偏好
- 每次 session 都讀（不論主對話、群組、子 agent）
- 只能手動修改，不交給自動化
- 80 行硬上限

**L1 MEMORY.md**
- 活躍專案索引（[P1] 永久 / [P2] 30 天過期）
- 只在主對話載入（安全考量：不在群組洩漏）
- 150 行硬上限
- 詳細內容指向 Obsidian（「詳見 Obsidian Projects/xxx.md」）

**L2 archive/**
- 過期的 P2 項目自動搬到這裡
- memory-janitor.py 每天 22:00 執行
- 洞察提取 → archive/insights-YYYY-MM-DD.md

### 流向

```
每日工作 → memory/YYYY-MM-DD.md (raw log)
                ↓ 萃取
         MEMORY.md (活躍索引)
                ↓ 過期
         archive/ (歸檔)

決策/SOP → Obsidian (知識本體)
```

## 為什麼不用其他方案

### 方案 B：單一大檔案
所有記憶塞一個檔案。問題：token 成本線性成長，超過 500 行就不實用。

### 方案 C：向量資料庫 RAG
用 embedding 搜尋記憶。問題：目前 archive 累積量不夠，ROI 太低。等 L2 累積到一定量再考慮（Phase 3b）。

### 方案 D：每次都搜不存
完全不維護記憶檔，每次用 `memory_search` 搜尋。問題：搜尋有延遲，且無法保證找到關鍵脈絡。

## 結果

- MEMORY.md 從無上限壓到 150 行，啟動 token 穩定
- 詳細內容指向 Obsidian，不重複儲存
- memory-janitor 自動清理，不需人工維護
- 安全隔離：群組對話不載入 MEMORY.md

## 風險

- Obsidian 需要開著才能存取（離線時退化為純 MEMORY.md）
- 三層之間的同步靠紀律，不是自動化（可能漏更新）

---

*2026-03-09*
