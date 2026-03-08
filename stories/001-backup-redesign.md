# 備份系統大改造：從 rsync 到三層架構

> 2026-03-08 | 一個非工程師怎麼從零設計備份與災難恢復系統

## 這篇在講什麼

我用 OpenClaw（AI Agent 框架）當日常工作夥伴，所有設定、腳本、記憶都放在 `~/.openclaw/workspace/`。某天我問自己一個問題：

> 如果這台電腦現在壞掉，我要多久才能恢復工作？

答案嚇到我了。這篇紀錄從發現問題到設計解法的完整思考過程。

---

## 背景：原本的備份長什麼樣

```
~/.openclaw/workspace/     ← 工作區（所有檔案在這）
~/.openclaw/backup-repo/   ← rsync 複製過來的副本
                           ← 再 git push 到 GitHub
```

問題：
1. **workspace 和 backup-repo 是兩份一樣的東西** — 違反 single source of truth
2. **rsync 是多餘的中間層** — 為什麼不直接 push？
3. **每次修改都要同步** — 容易漏、容易不一致

但真正的致命問題不在這裡。

---

## 第一個問題：模擬「電腦壞掉」

我跟 AI 做了一次災難恢復模擬：假設電腦現在壞掉，從新的 Mac 恢復。

### 恢復流程

```
新電腦 → git clone backup-repo → 拿回 workspace → 開始工作？
```

### 三個致命缺口

| 缺口 | 說明 | 後果 |
|------|------|------|
| `secrets/` 沒備份 | 6 個 API key/token 只存在本機 | 所有外部服務斷線 |
| `auth-profiles.json` 沒備份 | 4 個 AI provider key | AI Agent 完全無法運作 |
| `.gitignore` 正確排除了 secrets | 但沒有替代的備份機制 | 安全了，但災難恢復時回不來 |

**安全和備份是矛盾的。** secrets 不能進 git（安全），但不備份就會丟（可用性）。需要第三條路。

---

## 思考過程：怎麼解？

### 查了業界做法

| 方案 | 做法 | 我的判斷 |
|------|------|---------|
| **chezmoi** | 用 age/gpg 加密後直接放進 git repo | 好，但多一個工具依賴 |
| **pass** | 每個 secret 一個 gpg 加密檔 + git | 太細碎，我的 secrets 不多 |
| **SOPS** | 欄位級加密 YAML/JSON | 過度設計，我不是 DevOps |
| **macOS Keychain + openssl** | 系統內建，不用裝東西 | ✅ 最簡單 |

### 決策：openssl + macOS Keychain + iCloud

理由：
- **openssl** — macOS 內建（LibreSSL 3.3.6），不用安裝
- **macOS Keychain** — 密碼管理交給系統，不用自己記
- **iCloud** — 異地備份，電腦壞了從 iCloud 拿回來

---

## 設計：三層備份架構

```
Layer 1: Git
├── workspace 直接是 git repo
├── 每日 22:05 自動 commit + push
└── 排除 secrets/（.gitignore）

Layer 2: Vault（加密備份）
├── 收集 secrets/ + SSH keys + 設定檔
├── AES-256-CBC + PBKDF2 加密
├── 存到 iCloud（異地）+ 本機
└── 密碼存 macOS Keychain

Layer 3: Cron Snapshot
├── 排程設定 jobs.json 的快照
└── 跟著 Git 一起 push
```

### 災難恢復流程（設計後）

```
新電腦
  → 登入 Apple ID（iCloud 同步）
  → 從 iCloud 拿 vault-restore.sh（解決雞生蛋問題）
  → bash vault-restore.sh → 恢復 secrets, SSH, 設定
  → git clone → 恢復 workspace
  → openclaw gateway start → 開始工作

Single point of failure: Apple ID 密碼
```

---

## 第二個問題：workspace = git repo

原本：workspace → rsync → backup-repo → push

改成：workspace 直接 push

### 用 git filter-repo 重寫歷史

```bash
# 路徑重寫：workspace/X → X
git filter-repo --path-rename 'workspace/:'

# 清除敏感檔案的歷史
git filter-repo --invert-paths --path openclaw.json
```

這樣保留了完整的 commit history，只是改了路徑結構。比砍掉重來好 — 歷史是有價值的。

### backup.sh 簡化

Before:
```bash
rsync workspace/ → backup-repo/
cd backup-repo && git add -A && git commit && git push
```

After:
```bash
cd workspace && git add -A && git commit && git push
```

少了 rsync，少了一個資料夾，少了一堆可能出錯的地方。

---

## 第三個問題：要不要公開這個 repo？

做完備份改造後，我想把這個案例公開。評估了幾個方案：

### 方案 A-1：工作 repo 直接公開

需要做的事：
- filter-repo 清洗歷史中的 email
- 把 memory/、USER.md 等私有檔案從 git 移到 vault
- 腳本裡的 email 改成環境變數
- 未來每次 commit 都要注意不洩漏

**問題：工作 repo ≠ 展示 repo。** 強行公開會讓日常維護變複雜，而且移走的 memory/ 恰好是最有展示價值的內容。

### 查了大神怎麼做

**Mathias Bynens（dotfiles ⭐30k+）** — 工作 repo 直接公開，用 `.extra` 存私有設定。但他公開的是 shell 設定，不是完整工作系統。

**swyx（Learn in Public）** — 重點不是 repo，是內容。Blog、tweet、newsletter。

**Maggie Appleton（Digital Garden）** — 最新趨勢：不是公開原始資料，而是從工作中「長出」公開內容。持續演化，允許不完美。

### 最終決策：Digital Garden 模式

```
私有工作 repo（不動）
  ↓ 摘取
公開 Garden repo（策展後的內容）
```

理由：
1. **工作環境零改動** — 備份系統、自動化、所有腳本都不用動
2. **零洩漏風險** — 私有 repo 保持私有
3. **展示最有價值的東西** — 思考過程，而不是原始 config
4. **符合趨勢** — Digital Garden > 公開 dotfiles

---

## 學到的事

### 關於備份
- **安全和備份是矛盾的** — 需要第三條路（加密後異地備份）
- **一個資料夾 = 一個專案** — rsync 複製就是在製造不一致
- **災難恢復要實際模擬** — 不模擬就不知道缺口在哪
- **Single point of failure 要明確** — 我的是 Apple ID 密碼

### 關於公開
- **工作 repo 不適合直接公開** — 脫敏成本 > 建一個新 repo
- **大神看的是思考過程** — 不是 config 檔案
- **Simon Willison 說得對** — commit 是工作的基本單位，值得被認真對待
- **展示「整理過的思考」比「原始資料」更有效** — 對讀者更友善，對自己也是一次整理

### 關於人 + AI 協作
- **AI 負責展開選項，人負責選路** — 我的決策模式是：方向先定 → 展開 → 選路 → 檢查驗證 → 執行
- **「檢查與驗證」不是客氣** — 是核心節奏。每個方案都要被 challenge
- **AI 會被自己的慣性帶著走** — 我提出 A-1 後 AI 一路往下做，是我問「有沒有不妥」才發現根本問題

---

## 最終架構圖

```
macOS 工作機
├── ~/.openclaw/workspace/          ← 私有 Git repo（每日自動 push）
│   ├── scripts/                    ← 自動化腳本
│   ├── memory/                     ← 每日筆記
│   ├── skills/                     ← Agent Skills
│   ├── secrets/                    ← API keys（gitignored）
│   └── BACKUP-RESTORE.md          ← 災難恢復 SOP
│
├── ~/.openclaw/local-vault/        ← 加密備份（本機）
│   └── vault-YYYY-MM-DD.enc
│
└── iCloud Drive/HiSenzi-Vault/    ← 加密備份（異地）
    ├── vault-YYYY-MM-DD.enc
    └── vault-restore.sh           ← 雞生蛋解法

GitHub (private)
└── hisenzi/hisenzi-backup          ← workspace 的 remote

GitHub (public)
└── hisenzi/how-i-work              ← 你現在看的這個 repo
```
