# Story 003: 一個人的 Agent 團隊 — main + worker + researcher

## 問題

所有任務都丟給同一個 agent（Opus），不管是跟人聊天、改 65KB 的 HTML、還是研究競品分析。結果：

- 簡單任務用了昂貴的模型
- 耗時的執行任務佔住主對話，人要等
- 沒有並行能力，一次只能做一件事

## 解法

把 agent 分成三個角色，用 OpenClaw 的 multi-agent 架構：

```
main (Opus)
├── worker (Sonnet) — 執行型任務
└── researcher (Sonnet) — 研究分析
```

### main — 指揮官

- 模型：Opus（最強推理）
- 職責：跟人對話、做判斷、分派任務、記憶管理
- 不做：大檔案改寫、耗時的 pipeline、純搜集型研究

### worker — 執行者

- 模型：Sonnet（高效）
- 職責：改程式碼、大型檔案處理、pipeline 執行、格式轉換
- 特點：透過 `sessions_spawn` 啟動，完成後自動回報

### researcher — 研究員

- 模型：Sonnet（高效）
- 職責：設計評估、SEO/AIO 分析、競品研究、UI/UX 審查
- 特點：產出結構化報告，main 做最終判斷

## 建立流程

用 OpenClaw CLI 建立，不直接編輯設定檔：

```bash
# 建立 agent
openclaw agents add worker --model sonnet
openclaw agents add researcher --model sonnet

# 設定 main 可以 spawn 子 agent
openclaw config set allowAgents '["worker","researcher"]'

# 重啟生效
openclaw gateway restart
```

六步確認清單：
1. `agents add` 建立 agent
2. `config set allowAgents` 授權
3. `gateway restart` 生效
4. 更新系統文件（架構圖）
5. 測試 spawn 一個任務
6. 確認回報機制正常

## 實際案例

### Worker 第一個任務：遊戲化改版

把 30KB 的學習儀表板用八角框架改成遊戲化版本。

- 啟動方式：`sessions_spawn` + 詳細 task 描述
- 模型：Sonnet
- 耗時：6 分鐘
- 產出：65KB 的完整 HTML（等級系統、成就徽章、進度條）
- main 做什麼：繼續跟人聊天，worker 完成後收到通知

如果用 main（Opus）自己做，要佔住主對話 6 分鐘。用 worker 做，main 繼續處理其他事，效率翻倍。

## 分工原則

**給 worker 的任務特徵：**
- 有明確輸入和輸出
- 不需要跟人互動
- 耗時超過 2 分鐘
- 不需要 Opus 級推理

**給 researcher 的任務特徵：**
- 需要搜集多方資料
- 產出是報告或建議（不是最終決策）
- main 做最終判斷

**留給 main 的：**
- 跟人對話
- 做判斷和決策
- 記憶管理
- 任務分派

## 學到的事

1. **模型分級不是省錢，是解放主對話。** Opus 的時間應該花在需要推理的地方，不是花在改 HTML。

2. **任務描述要完整。** 給 worker 的 task 越清楚，產出越好。模糊的指令 = 模糊的結果。

3. **六步確認清單很重要。** 漏掉 `allowAgents` 設定就 spawn 不了，漏掉文件更新下次就忘記有這個 agent。

---

*2026-03-09*
