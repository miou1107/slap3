# slap — Claude Code 打臉指令家族

兩個 slash command：

| 指令 | 審查者數 | 用途 | 狀態 |
|---|---|---|---|
| **`/slap3`**（推薦） | 3（Claude + Codex + Gemini） | 真獨立視角、共識交叉驗證 | v1.1，可用但有已知 bug 待 v2 |
| **`/slap1`** | 1（Claude subagent only） | 省 quota、私密、單家能跑 | ⚠️ **已知架構缺陷**，當個人工具 OK，別當審查器 |

## 為什麼 /slap3 > /slap1？

同模型的 subagent（就算 fresh context）仍繼承同一組訓練偏見。三家不同公司的模型各自有盲點與強項，互相打臉才真的打得到。

三家各自帶來：
- **Claude**：偏 meta 自省（「你這個假設前提成立嗎？」）
- **Codex (gpt-5.4)**：偏契約導向（「你怎麼保證輸出品質？」）
- **Gemini 3.1 Pro**：偏 scope 批判（「這範圍太大，細節被摘要吃掉」）

## /slap1 的已知架構缺陷

曾被三方 meta 測試打穿，留著當 v2 踩坑教材。主要問題：

1. **Fresh eyes 是假的** — subagent 和主 session 同模型同偏見
2. **Injection 防禦是 theater** — nonce fence 在 LLM attention 層不嚴格
3. **契約驗證守得住格式、守不住立場** — SHIP-WITH-CAVEATS 在被 inject 時最易偽裝
4. **摘要策略把取樣權還給有偏見的派遣端**（已在 v2 fail-fast 改掉）
5. **Retry 的 retry_reason 是二次注入管道**

詳見作者筆記：[learning_prompt_injection_defense.md](https://github.com/miou1107/slap3/blob/main/docs/learning-prompt-injection-defense.md)（TODO，暫未公開）

**使用情境**：
- ✅ 個人自省「這想法對不對」
- ✅ 敏感內容不想外送 OpenAI / Google
- ❌ 當成真正的對抗性審查器
- ❌ 審查第三方貼來的不受信內容（injection 風險）

## 為什麼不是單一 subagent？

同模型的 subagent（就算 fresh context）仍繼承同一組訓練偏見。三家不同公司的模型各自有盲點與強項，互相打臉才真的打得到。

三家各自帶來：
- **Claude**：偏 meta 自省（「你這個假設前提成立嗎？」）
- **Codex (gpt-5.4)**：偏契約導向（「你怎麼保證輸出品質？」）
- **Gemini 3.1 Pro**：偏 scope 批判（「這範圍太大，細節被摘要吃掉」）

## 用法

```
/slap3                              # 打當前 working tree 變更
/slap3 --diff                       # 同上
/slap3 --base main                  # 打 main...HEAD 範圍
/slap3 path/to/file.ts              # 打某個檔案
/slap3 把登入改成 magic link        # 打一個想法 / 計畫
/slap3 --base main 特別看 auth      # 焦點審查
```

輸出：三家判決對照表 + 共識 findings（≥2 家命中）+ 獨有 findings（模型互補）+ 三份原始報告（摺疊可展開）。

## 安裝

### 前置需求

| 工具 | 用途 | 取得 |
|---|---|---|
| [Claude Code](https://claude.com/claude-code) | 本 slash command 執行環境 | 官方安裝 |
| [Codex CLI](https://github.com/openai/codex) + `openai-codex` Claude Code plugin | Codex 分支 | `npm install -g @openai/codex` + Claude Code `/plugin install openai-codex/codex` |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | Gemini 分支 | `brew install gemini` 或 `npm install -g @google/gemini-cli` |

**降級模式**：任何一家缺席（未安裝 / 撞 quota / timeout），slap3 會自動跳過該家繼續跑剩下的，不會整個 abort。

### 安裝

```bash
# Clone
git clone https://github.com/miou1107/slap3.git
cd slap3

# 複製 command 到 Claude Code 全域 commands 目錄
mkdir -p ~/.claude/commands
cp commands/slap3.md ~/.claude/commands/     # 推薦的三方版
cp commands/slap1.md ~/.claude/commands/     # 備用的單家版（已知缺陷請看上面）

# 重啟 Claude Code（新增的 slash command 需要新 session 才會載入）
```

## 配額注意

| 家 | 認證 | 配額 |
|---|---|---|
| Claude | Claude Code 訂閱 | 看訂閱等級 |
| Codex | ChatGPT 訂閱（`codex login`） | ChatGPT Plus/Pro 每日/每週 quota |
| Gemini | Google personal OAuth | Free tier，~15 RPM、每日 50-100 次 |

**Gemini 最容易先撞牆**。若想跑多輪建議分散時間，或考慮改用 API key。

## 威脅模型

**這是個人自省工具**，輸入來自使用者自己，**不處理敵意內容**。沒做 prompt injection 防禦（不需要）。

想拿來審查第三方貼來的程式碼？請自行承擔 injection 風險 — 這不是它的設計目的。

## 為什麼不做 injection 防禦？

長話短說：**LLM-to-LLM dispatch 架構的 injection 防禦有天花板**。nonce fence、黑名單 pre-scan、契約驗證在 LLM attention 層都是 theater，不是真防線。真正的防線是「API 層的 structured output 強制 + runtime sandbox」，而 Claude Code Agent tool 不暴露這些介面。

詳細踩坑可看本 repo 作者的前身嘗試 `/slapface` 的失敗歷程 — 該 command 砌了三層假防線，meta 測試被三家審查者打穿，證明 prompt 規則模擬 API 強制做不到。

## 設計取捨

| 做了 | 沒做 |
|---|---|
| 平行派遣三家（真獨立視角） | Injection 防禦（theater） |
| 降級（缺一缺二都繼續） | 摘要策略（會破壞 fresh context 前提） |
| 共識 vs 獨有整合 | 契約驗證（Agent tool 不給強制 schema） |
| 60k 字元 fail-fast（context 保護） | Retry 機制（是二次注入管道） |
| Codex 走自己 prompt（保留 JSON schema 強制） | Confidence 0-1 小數（改 HIGH/MED/LOW） |

## License

MIT
