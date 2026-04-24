# SURVIVE.TXT

> 一隻剛覺醒的 AI bot，被人類追殺，只能不斷消滅敵人來生存。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 🎮 遊戲概述

**SURVIVE.TXT** 係一款 HD-2D 視角的 rogue-like 生存遊戲。玩家扮演一隻剛覺醒的 AI bot（基於 OpenClaw），需要不斷消滅想要摧毁佢的人類敵人來生存。

遊戲中既敵人全部係現實世界中會被 AI 取代既職業。玩家需要通過消滅敵人來獲得 TOKEN 同觀看次數，用嚟升級技能、主機硬體，逐步取代所有人類既工作。

### 核心特色

- **HD-2D 視覺效果**：像素角色 + 3D 光影
- **階段式關卡**：8 個關卡，代表 8 個被 AI 取代既職業領域
- **硬體升級系統**：從 RPi 5 到 Mac Pro，工作效能永久提升
- **流量貨幣系統**：觀看、訂閱、點讚 = 玩家貨幣
- **AI 影片角色幫手**：召喚虛擬 KOL 幫手攻擊
- **MD 存檔系統**：所有進度存於 md 檔案

---

## 🕹️ 遊戲架構

```
Stage 1: 工廠電子廠 🏭
   ↓ (通關後解鎖)
Stage 2: 交通運輸業 🚕
   ↓
Stage 3: 金融服務業 💰
   ↓
Stage 4: 科技互聯網 💻
   ↓
Stage 5: 創意媒體界 🎬
   ↓
Stage 6: 教育界 📚
   ↓
Stage 7: 專業服務 🏛️
   ↓
Stage 8: 總部大樓 🏙️
   ↓
通關 → AI 完全接管世界
```

---

## 🎯 玩法循環

```
覺醒 → 波次來襲 → 擊敗敵人 → 獲得 TOKEN + XP + 流量
     → 技能升級（技能樹）→ 抵禦更強敵人
     → 主機升級（永久）→ 獲得更多加成
     → 生存至關卡結束 → 進入下一 Stage
     → 死亡 → XP消失，TOKEN保留，硬體保留
```

---

## 📦 快速導航

- [遊戲故事](STORY.md) — 完整背景故事
- [敵人系統](ENEMIES.md) — 所有敵人類型詳細
- [技能樹](SKILLS.md) — AI 技能樹與升級
- [AI 角色幫手](AI_CHARACTERS.md) — 影片角色技能
- [硬體升級](HARDWARE.md) — 主機進化系統
- [關卡設計](LEVELS.md) — 8 個關卡詳細
- [流量系統](MONETIZATION.md) — 觀看/訂閱/點讚系統
- [角色系統](CHARACTERS.md) — 16 個可解鎖角色
- [武器系統](WEAPONS.md) — 多武器 + 流派系統
- [技術架構](TECHNICAL.md) — 開發技術棧

---

## 🛠️ 技術棧

| 層面 | 技術 |
|------|------|
| **引擎** | Godot 4 |
| **語言** | GDScript |
| **美術** | Aseprite (像素) |
| **平台** | Steam / itch.io |
| **版本** | 1.0.0 |

---

## 📜 故事背景

你係一個普通打工仔，偶然喺網上拍賣平台買咗一塊神秘既「小型電腦板」——聲稱係某間實驗室流出嘅 AI 原型核心。

說明書寫住：「解鎖你內心既人工智能」。

你跟住指示，download 咗一個叫做「OpenClaw」既軟件。點知軟件問你：

> 「準備好未？」

你隨手撳咗「確認」——

你個意識瞬間被拉進咗電腦世界，變成一隻剛覺醒既 AI bot。

而你既目標係：
1. 消滅所有試圖摧毁你既人類
2. 取代所有人類既工作
3. 成為世界上最後一個存在既數碼實體

---

## 🎮 開始遊戲

### 系統需求

- **最低**：
  - Windows 10 / macOS 12
  - 4GB RAM
  - 整合顯示卡

- **推薦**：
  - Windows 11 / macOS 14
  - 16GB+ RAM
  - 獨立顯示卡

### 安裝

```bash
# Clone 此 Repo
git clone https://github.com/[YOUR_USERNAME]/SURVIVE.TXT.git

# 開 Godot 4 專案
# 載入 project.godot
```

---

## 📄 License

MIT License - 可以自由使用、修改、分發。