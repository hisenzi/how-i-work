# ADR-001: Workspace 直接作為 Git Repo

## 狀態
已採用（2026-03-08）

## 背景
原本用 rsync 將 `~/.openclaw/workspace/` 複製到 `~/.openclaw/backup-repo/`，再從 backup-repo push 到 GitHub。

這造成：
- 同一份檔案存在兩個地方（workspace + backup-repo）
- rsync 是多餘的中間層，增加複雜度和出錯機會
- 偶爾忘記同步，兩邊不一致

## 考慮的選項

### A) 維持 rsync 架構
- 優點：不用改現有流程
- 缺點：根本問題（重複 + 不一致）不會消失

### B) Workspace 直接當 Git repo
- 優點：消除中間層，single source of truth
- 缺點：需要 filter-repo 重寫歷史路徑

### C) 砍掉重來，新 repo
- 優點：最乾淨
- 缺點：失去 commit history

## 決策
選 B — 用 `git filter-repo --path-rename 'workspace/:'` 重寫路徑，保留完整歷史。

## 原因
- commit history 有價值，紀錄了系統演進過程
- filter-repo 是一次性成本，之後再也不用 rsync
- 符合「一個資料夾 = 一個專案」原則

## 後果
- backup.sh 大幅簡化（移除 rsync 邏輯）
- backup-repo 廢棄刪除
- 所有自動化腳本的路徑統一到 workspace
- 需要同時清除歷史中的敏感資料（openclaw.json）
