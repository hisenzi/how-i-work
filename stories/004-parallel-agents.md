# 一次可以派幾個 Agent？平行開發的實戰經驗

> date: 2026-03-20 | context: HiBlocks + marketing-agent-pack 同時開發

## 場景

晚上 11 點，兩個專案都需要推進：
1. **HiBlocks**：模組化網頁系統，Phase 0.5 + Phase 1（移植模組 + 建新模組）
2. **行銷人的 AI Agent**：設計 Agent 人格包模板

一個一個做？太慢。同時派兩個子 Agent，各自在不同 repo 工作。

## 實際操作

```
主 Agent（HiSenzi, Opus）
├── 子 Agent 1（Sonnet）→ ~/.openclaw/hiblocks-astro/
│   任務：移植 Navbar/Portfolio + 建 Hero/Footer/CTA/Feature + GSAP
│   預估：5-10 分鐘
│
└── 子 Agent 2（Sonnet）→ ~/.openclaw/openclaw-marketing-agent-pack/
    任務：建立 SOUL.md / AGENTS.md / USER.md / TOOLS.md 模板
    預估：3-5 分鐘
```

主 Agent 負責規劃、派任、驗收。子 Agent 負責執行。完成自動通知，不需要輪詢。

## 能同時跑幾個？

### 技術限制

| 限制因素 | 說明 | 建議上限 |
|---|---|---|
| API Rate Limit | Anthropic 有並發限制 | 3-4 個 Sonnet |
| Token 成本 | 每個 Agent 獨立計費 | 看預算 |
| 檔案衝突 | 多 Agent 改同一個 repo 會打架 | 一個 repo 一個 Agent |
| 上下文品質 | 任務太複雜 Agent 容易跑偏 | 單次任務控制在 10-15 個步驟 |

### 實務建議

**安全範圍：同時 2-3 個**

超過 3 個的問題不是技術，是管理：
- 你要同時追蹤多個 Agent 的進度
- 驗收需要逐一確認
- 出錯時要判斷是哪個 Agent 的問題

**最佳實踐：**
1. 每個 Agent 在不同 repo / 不同目錄
2. 任務明確、步驟具體（不是「幫我做一個網站」）
3. 主 Agent 不輪詢，等完成通知
4. 完成後逐一驗收，不批量接受

## 分工模型

```
人（Vincent）
  ↕ 決策、方向、驗收
主 Agent（HiSenzi, Opus）
  ↕ 規劃、拆任務、派工、品質把關
子 Agent（Sonnet × N）
  ↕ 執行、建置、coding
```

這就是一人公司的槓桿：**一個人的判斷力 × N 個 Agent 的執行力**。

## 什麼時候不該平行

- 任務之間有依賴（A 的輸出是 B 的輸入）
- 需要人工決策的環節（先確認再繼續）
- 探索性工作（不確定方向，需要邊做邊調）

這些情況串行做比較好，避免浪費 token 在錯誤方向上。

## 成本考量

以今晚為例：
- 子 Agent 1（HiBlocks Phase 1）：預估 ~20k tokens
- 子 Agent 2（人格包模板）：預估 ~10k tokens
- 主 Agent（規劃 + 驗收）：正常對話量

兩個平行跑 vs 串行跑，時間省一半，token 成本幾乎一樣（因為任務量不變，只是並發）。

## 結論

> 平行 Agent 不是炫技，是時間管理。關鍵不是「能跑幾個」，而是「任務拆得夠不夠乾淨」。

拆得乾淨，3 個 Agent 各做各的，10 分鐘搞定 3 小時的工作。
拆不乾淨，1 個 Agent 就夠你收拾的了。

**能力在拆解，不在並發數。**
