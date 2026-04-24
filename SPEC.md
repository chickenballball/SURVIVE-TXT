# SURVIVE.TXT — 技術規格書 (HD-2D)

> ⚠️ **重要：呢個係 2D 遊戲，唔係 3D 遊戲**
> 
> HD-2D = 2D像素精靈圖 + 2D/3D光影Shader
> 
> **禁止**使用任何 3D 模型、3D 場景、3D Camera

---

## 🎮 遊戲類型

- **類型**: Horizontal scrolling 2D action rogue-like (Vampire Survivor 類型)
- **視角**: Top-down / Isometric 2.5D (上帝視角)
- **技術棧**: Godot 4 + GDScript
- **美術風格**: HD-2D (像素角色 + Shader 光影效果)

---

## 🚫 絕對禁止 (3D相關)

```
❌ 3D model / 3D mesh
❌ Camera3D
❌ DirectionalLight3D (除非係2D場景裝飾用)
❌ 3D environment
❌ 3D animation
❌ .obj / .glb / .gltf 模型檔案
❌ 3D particle system (用2D的)
```

---

## ✅ 必須使用 (2D相關)

### 精靈圖 (Sprite)
```
- 格式: PNG sprite sheet
- 尺寸: 32x32 或 48x48 像素
- 動畫: 4-8 幀 (行走、攻擊、受傷)
- Filter: Nearest (像素感)
```

### 2D 場景
```
- 背景: 靜態圖片 或 簡單2D tilemap
- 裝飾: 2D精靈圖裝飾物
- 燈光: 透過 Shader 實現 (唔係3D光線)
```

### 2D Camera
```
- Camera2D
- 可以有 limited zoom
- 可以有 smooth follow
```

### Shader 效果 (用程式碼实现)
```
- 像素化渲染 (Pixelate shader)
- 光影效果 (Light shadow 2D)
- Bloom (发光效果)
- Color grading
- 2D lighting (點光源精靈圖)
```

---

## 📁 專案結構

```
SURVIVE-TXT/
├── project.godot
├── scenes/
│   ├── main.tscn          # 主場景
│   ├── player.tscn        # 玩家角色
│   ├── enemy/
│   │   ├── worker.tscn    # 工廠工人
│   │   ├── driver.tscn    # 司機
│   │   └── boss.tscn      # BOSS
│   └── ui/
│       └── hud.tscn       # HUD介面
├── scripts/
│   ├── player.gd         # 玩家控制
│   ├── enemy.gd          # 敵人AI
│   ├── wave.gd           # 波次系統
│   └── game.gd           # 遊戲主控制器
├── assets/
│   ├── sprites/
│   │   ├── player/       # 玩家精靈圖
│   │   ├── enemies/      # 敵人精靈圖
│   │   └── effects/      # 特效
│   ├── backgrounds/      # 背景圖
│   └── audio/            # 音效
├── shaders/
│   ├── pixelate.gdshader
│   ├── bloom.gdshader
│   └── lighting.gdshader
└── SPEC.md               # 呢份文件
```

---

## 🎨 角色尺寸 (像素)

| 角色 | 尺寸 | 動畫幀數 |
|------|------|----------|
| 玩家 (AI) | 48x48 | 4-8 幀 |
| 普通敵人 | 32x32 | 4 幀 |
| 菁英敵人 | 48x48 | 4 幀 |
| BOSS | 96x96 | 6-8 幀 |
| 環境道具 | 16x16 或 32x32 | 1 幀 |

---

## 🖼️ 美術資源獲取策略

### 暫時用 Placeholder (免費)

```
itch.io (免費像素資源)
OpenGameArt.org
Kenney.nl (免費)
Godot Asset Library
```

### 之後自定義

```
用 Aseprite 製作像素角色
或Leonardo AI 生成像素圖
匯出 PNG sprite sheet
```

---

## 🎮 核心系統

### 1. 玩家控制
```
移動: WASD
瞄準: 鼠標位置 (角色面向鼠標)
攻擊: 左鍵 (自動攻擊最近敵人)
衝刺: Shift (5-8秒冷卻, 0.5秒無敵)
切換武器: Q/E 或 1-6
```

### 2. 敵人系統
```
敵人從四周生成
向玩家移動並攻擊
擊殺後掉落 TOKEN / 道具
每10波有 BOSS
```

### 3. 波次系統
```
每波次: 30-60秒
每5波: 出現商店
每10波: BOSS戰
難度指數增加
```

### 4. 武器系統
```
最多6把武器同時裝備
武器自動攻擊
可以手動切換主武器
```

### 5. 技能系統
```
TOKEN 解鎖技能
技能樹升級
永久保存
```

---

## 📋 开发顺序

### Phase 1: MVP (1-2日)
```
1. 創建 Godot 4 專案
2. 設定 2D 環境 (唔好有3D野)
3. 用 placeholder 精靈圖測試玩家移動
4. 實現基礎 WASD 移動 + 鼠標瞄準
5. 實現左鍵攻擊
6. 生成幾個 test敵人
7. 基本的碰撞檢測
8. 基礎 HP 系統
```

### Phase 2: 核心玩法 (3-5日)
```
1. 波次系統
2. 敵人生成器
3. 武器系統 (多武器切換)
4. TOKEN 掉落和收集
5. 商店系統
6. HUD 介面 (HP, TOKEN, 波次)
```

### Phase 3: 內容 (1-2週)
```
1. 所有敵人類型
2. 所有武器類型
3. 技能樹系統
4. 8個關卡
5. BOSS戰
6. 粒子特效 / Shader效果
```

### Phase 4: 完成 (1週)
```
1. 音效 / 音樂
2. 嘲諷對話系統
3. 成就系統
4. 存檔系統
5. 測試 / Bug修復
```

---

## 🔧 關鍵設定 (Godot 4)

### project.godot 必須設定
```
[display]
window/stretch/mode="viewport"
window/stretch/aspect="keep"

[rendering]
textures/canvas_textures/default_texture_filter=1  # Nearest
```

### Camera2D 設定
```
Zoom: 2.0 (如果需要看到更多範圍)
Process Mode: Always
Current: true
```

### Shader 範例 (pixelate.gdshader)
```gdscript
shader_type canvas_item;

uniform float pixel_size = 4.0;

void fragment() {
    vec2 uv = UV;
    uv = floor(uv / pixel_size) * pixel_size;
    COLOR = texture(TEXTURE, uv);
}
```

---

## 📝 備注

1. **暫時用 placeholder 資源**，美術之後再替換
2. **專注遊戲性**，視覺效果係後加的
3. **每週期要能跑一次**，確認功能正常
4. **問題隨時問**，我幫你解決

---

## ⚠️ 給 Claude Code 的最終指示

```
請嚴格遵守以下規則：

1. 呢個係 2D 遊戲，唔係 3D
2. 所有角色/敵人必須用 2D sprite (PNG)
3. Camera 必須用 Camera2D，唔用 Camera3D
4. 場景必須係 2D，唔用 3D 環境
5. 光源效果用 2D sprite 或 shader 实现
6. 如果唔確定，問我先

跟住 SPEC.md 呢份文件做。
```

---

## 🎯 目標

做出一个可以玩的 MVP：
- 玩家可以移動
- 可以攻擊敵人
- 有波次系統
- 有 TOKEN 獎勵
- 有商店可以買道具/武器
- 可以升級技能

之後再慢慢加更多內容。

---

**準備好開始了嗎？我們可以先從創建 Godot 專案開始！** 🚀
