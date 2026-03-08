# ADR-002: 用 OpenSSL + Keychain 而不是 GPG

## 狀態
已採用（2026-03-08）

## 背景
需要加密備份 secrets/、SSH keys、設定檔。需要選擇加密工具。

## 考慮的選項

### A) GPG
- 業界標準，chezmoi / pass 都用
- macOS 沒有內建，需要安裝 gnupg
- key management 複雜（生成、備份、過期、信任鏈）
- 一個人用太重了

### B) age（現代替代品）
- 比 GPG 簡單很多
- 也需要安裝（`brew install age`）
- 沒有內建密碼管理

### C) OpenSSL + macOS Keychain
- macOS 內建 LibreSSL 3.3.6，不用裝東西
- AES-256-CBC + PBKDF2，夠安全
- 密碼存 Keychain，系統級管理
- Keychain 可以透過 iCloud 同步到新設備

## 決策
選 C — 零安裝、零外部依賴、密碼管理交給 OS。

## 原因
- 一人使用場景，不需要 GPG 的信任鏈
- 災難恢復時新電腦上一定有 openssl，不用先裝工具
- macOS Keychain 是 Apple 生態的一等公民，跟 iCloud 天然整合
- 簡單的事不要用複雜的工具

## 後果
- `vault-backup.sh` 和 `vault-restore.sh` 各約 80 行 shell script
- 密碼只需要設定一次，之後 Keychain 自動處理
- 跟 Linux 不相容（Keychain 是 macOS 限定），但我只用 Mac
- 如果未來需要跨平台，可以考慮遷移到 age

## 踩坑紀錄
- Keychain 密碼存在 `login.keychain-db`（本機），不是 iCloud Keychain
- 這代表換新電腦時密碼**不會自動同步**，需要手動輸入或事先移到 iCloud Keychain
- vault-restore.sh 已處理這個情況：先嘗試 Keychain，失敗就 fallback 到手動輸入
