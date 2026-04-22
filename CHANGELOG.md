# Changelog

## v1.1 — 2026-04-21

### 修 bug（meta 測試三方打臉發現）

- **allowlist 與實作不符** (Codex U2)：`allowed-tools` 只列 `Bash(git:*)` 等前綴，但實作用了 `sort`/`tail`/`sed`/pipe/`trap` 等 shell composition，原本會被 Claude Code 執行器擋下。改為 `Bash` 無前綴，承認需要完整 shell 能力。
- **Bash 單引號逃逸崩潰** (Gemini U4)：原本把 focus text / prompt 直接插值進 `'...'`，只要內容含單引號就會炸掉 Codex/Gemini 分支（再觸發退化成只剩 Claude）。改為「先用 Write tool 寫進臨時檔 → `cat` 讀進環境變數 → `"$VAR"` 雙引號傳遞」。

### 未修（留到 v2）

- 降級到 N=1 時仍產共識格式（共識 C1）
- 派遣端球員兼裁判，共識判定主觀（共識 C2）
- mktemp 預設路徑 + trap 並行語意（共識 C3）
- Size 檢查 60k 字元門檻、檢查順序 fail-late（共識 C4）
- UNIVERSAL_PROMPT 三家共用違反「真不同視角」（Claude U1）
- Codex `--diff` 對 untracked 檔案 silent drop（Codex U3）
- LLM 遇 tool error 會本能道歉重試（Gemini U5）

## v1.0 — 2026-04-21

- 首版：三方平行打臉 slash command，派遣 Claude Opus + Codex gpt-5.4 + Gemini 3.1 Pro
- 威脅模型：個人自省工具，不處理敵意輸入
- 降級：三缺一或三缺二可繼續
- 輸出：判決對照表 + 共識/獨有 findings + 原始三份報告摺疊
