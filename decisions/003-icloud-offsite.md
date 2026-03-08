# ADR-003: 用 iCloud 作為異地加密備份

## 狀態
已採用（2026-03-08）

## 背景
加密後的 vault 需要異地備份。電腦壞了、被偷、硬碟故障時，要能從別的地方拿回來。

## 考慮的選項

### A) 加密後 push 到 GitHub
- chezmoi 的做法：加密檔直接放進 git repo
- 優點：備份流程統一，一個 push 搞定
- 缺點：GitHub 掛了就連公開 repo 和加密備份一起丟

### B) 獨立的雲端儲存（S3、GCS）
- DevOps 標準做法
- 需要額外帳號、API key、付費
- 過度設計

### C) iCloud Drive
- Apple 生態內建
- 登入 Apple ID 就自動同步
- 不需要額外設定、額外帳號、額外費用
- 端到端加密（開啟進階資料保護後）

## 決策
選 C — iCloud Drive。

## 原因
- 我已經在 Apple 生態裡，不需要多一個帳號
- 災難恢復的第一步是「登入 Apple ID」→ iCloud 自動開始同步
- vault-restore.sh 也放一份在 iCloud，解決雞生蛋問題
- 跟 macOS Keychain 天然配合（同一個 Apple ID）

## 災難恢復的 single point of failure

```
Apple ID 密碼
  → 登入新 Mac
  → iCloud 同步 vault-restore.sh + vault-*.enc
  → Keychain 同步密碼（或手動輸入）
  → 解密 → 拿回所有 secrets
  → git clone → 拿回所有程式碼
  → 恢復完成
```

所有雞蛋都在 Apple ID 這個籃子裡。可以接受，因為：
1. Apple ID 有 2FA
2. 不太可能同時丟 Apple ID + 本機
3. 對一個人來說，簡單 > 分散風險帶來的複雜度

## 後果
- iCloud 同步有時候有延遲（幾秒到幾分鐘），但不影響備份
- `du -h` 在加密檔上顯示 `??`（已知的 cosmetic bug，不影響功能）
- 如果 Apple 改變 iCloud Drive 政策，需要重新評估
- 備份檔保留 7 天（自動清理舊的）
