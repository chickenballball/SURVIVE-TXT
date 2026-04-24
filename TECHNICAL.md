# TECHNICAL.md — 技術實現詳細架構

---

## 🎮 引擎選擇

### Godot 4 vs Unity vs Unreal

| 項目 | Godot 4 ✅ | Unity | Unreal |
|------|-----------|-------|--------|
| 費用 | 完全免費 | 免費/付費 | 免費/訂閱 |
| 2D支援 | 優秀 | 優秀 | 一般 |
| 對新手 | 簡單易學 | 中等 | 困難 |
| 社區 | 成長中 | 龐大 | 龐大 |
| 發布 | 全平台 | 全平台 | 全平台 |

**結論：使用 Godot 4（推薦給你的團隊/自己）**

---

## 📁 專案結構 (Godot 4)

```
SURVIVE-TXT/
├── project.godot
├── scenes/
│   ├── main.tscn              # 主場景
│   ├── ui/
│   │   ├── hud.tscn           # HUD介面
│   │   ├── pause_menu.tscn    # 暫停選單
│   │   └── game_over.tscn     # 結束畫面
│   ├── player/
│   │   ├── player.tscn        # 玩家角色
│   │   └── forms/             # 形態sprite
│   ├── enemies/
│   │   ├── base_enemy.tscn   # 敵人基類
│   │   ├── worker.tscn       # 工廠工人
│   │   ├── driver.tscn       # Uber司機
│   │   └── boss.tscn         # BOSS
│   ├── weapons/
│   │   ├── base_weapon.tscn  # 武器基類
│   │   ├── melee_weapon.tscn # 近戰武器
│   │   └── ranged_weapon.tscn# 遠程武器
│   ├── pickups/
│   │   ├── token.tscn        # TOKEN道具
│   │   ├── weapon_pickup.tscn# 武器掉落
│   │   └── powerup.tscn      # 能力提升
│   ├── effects/
│   │   ├── explosion.tscn    # 爆炸特效
│   │   └── hit_effect.tscn   # 擊中特效
│   └── levels/
│       ├── stage_1.tscn      # Stage 1地圖
│       └── stage_2.tscn      # Stage 2地圖
├── scripts/
│   ├── main.gd               # 遊戲主控制器
│   ├── game_state.gd         # 全域遊戲狀態
│   ├── player/
│   │   ├── player.gd         # 玩家控制
│   │   ├── player_form.gd    # 形態進化
│   │   └── dash.gd            # 衝刺技能
│   ├── enemies/
│   │   ├── enemy.gd          # 敵人基類
│   │   ├── enemy_ai.gd      # AI行為
│   │   ├── enemy_spawner.gd  # 生成器
│   │   └── boss.gd           # BOSS邏輯
│   ├── weapons/
│   │   ├── weapon.gd         # 武器基類
│   │   ├── melee.gd          # 近戰武器
│   │   ├── ranged.gd         # 遠程武器
│   │   └── weapon_inventory.gd# 武器欄
│   ├── wave/
│   │   ├── wave_manager.gd    # 波次管理器
│   │   └── events.gd          # 隨機事件
│   ├── items/
│   │   ├── token.gd          # TOKEN
│   │   └── shop.gd           # 商店
│   ├── ui/
│   │   ├── hud.gd            # HUD
│   │   ├── minimap.gd        # 小地圖
│   │   └── dialogue.gd        # 對話系統
│   └── systems/
│       ├── save_system.gd     # 存檔
│       ├── settings.gd        # 設定
│       └── audio_manager.gd   # 音效
├── assets/
│   ├── sprites/
│   │   ├── player/
│   │   ├── enemies/
│   │   ├── weapons/
│   │   └── effects/
│   ├── backgrounds/
│   ├── audio/
│   │   ├── sfx/
│   │   └── music/
│   └── fonts/
├── shaders/
│   ├── pixelate.gdshader
│   ├── bloom.gdshader
│   └── lighting.gdshader
└── resources/
    ├── enemies/
    │   ├── worker.tres
    │   └── boss_supervisor.tres
    ├── weapons/
    │   ├── club.tres
    │   └── sword.tres
    └── stages/
        ├── stage1.tres
        └── stage2.tres
```

---

## 🎮 核心系統代碼架構

### 1. 遊戲主控制器 (main.gd)

```gdscript
# main.gd
extends Node2D

# 遊戲狀態
enum GameState { MENU, PLAYING, PAUSED, GAME_OVER }
var current_state: GameState = GameState.MENU

# 系統引用
@onready var player = $Player
@onready var wave_manager = $WaveManager
@onready var hud = $HUD
@onready var minimap = $HUD/MiniMap

func _ready():
    # 初始化
    load_settings()
    connect_signals()

func _process(delta):
    match current_state:
        GameState.PLAYING:
            update_game(delta)
        GameState.PAUSED:
            update_pause()

func update_game(delta):
    # 更新玩家
    player.update_player(delta)
    # 更新敵人
    wave_manager.update_waves(delta)
    # 更新道具
    update_pickups(delta)
    # 更新小地圖
    minimap.update_map()

func start_game():
    current_state = GameState.PLAYING
    wave_manager.start_waves()
    player.spawn()

func game_over():
    current_state = GameState.GAME_OVER
    hud.show_game_over()
    queue_free()
```

### 2. 玩家控制 (player.gd)

```gdscript
# player.gd
extends CharacterBody2D

# 移動屬性
@export var speed: float = 200.0
@export var dash_speed: float = 600.0
@export var dash_cooldown: float = 6.0
@export var dash_duration: float = 0.5

# 當前狀態
var velocity: Vector2 = Vector2.ZERO
var facing_direction: Vector2 = Vector2.ZERO
var can_dash: bool = true
var is_dashing: bool = false
var current_hp: int = 100
var max_hp: int = 100

# 武器系統
@onready var weapon_slot = $WeaponSlot
@onready var sprite = $Sprite2D
@onready var hitbox = $Hitbox

func _ready():
    add_to_group("player")
    setup_input()

func _physics_process(delta):
    handle_movement(delta)
    handle_attack()
    handle_dash(delta)

func handle_movement(delta):
    # 獲取移動輸入
    var input_dir = Vector2(
        Input.get_axis("move_left", "move_right"),
        Input.get_axis("move_up", "move_down")
    ).normalized()
    
    if is_dashing:
        velocity = facing_direction * dash_speed
    else:
        velocity = input_dir * speed
    
    move_and_slide()

func handle_attack():
    if Input.is_action_pressed("attack"):
        weapon_slot.attack(facing_direction)

func handle_dash(delta):
    if Input.is_action_just_pressed("dash") and can_dash:
        start_dash()
    
    if not can_dash:
        dash_timer -= delta
        if dash_timer <= 0:
            can_dash = true
            dash_cooldown_timer = dash_cooldown

func start_dash():
    is_dashing = true
    can_dash = false
    dash_timer = dash_duration
    emit_signal("dash_started")

func _on_dash_finished():
    is_dashing = false
    dash_cooldown_timer = dash_cooldown

# 瞄準方向跟隨鼠標
func _input(event):
    if event is InputEventMouseMotion:
        facing_direction = (event.position - global_position).normalized()
        sprite.rotation = facing_direction.angle()
        hitbox.rotation = facing_direction.angle()
```

### 3. 敵人AI (enemy.gd)

```gdscript
# enemy.gd
extends CharacterBody2D

# 敵人屬性
@export var max_hp: int = 20
@export var speed: float = 50.0
@export var damage: int = 10
@export var attack_range: float = 30.0
@export var attack_cooldown: float = 1.0

var current_hp: int
var target: Node2D
var attack_timer: float = 0.0
var state: EnemyState = EnemyState.IDLE

enum EnemyState { IDLE, CHASE, ATTACK, HURT, DEAD }

func _ready():
    current_hp = max_hp
    target = get_tree().get_first_node_in_group("player")
    add_to_group("enemies")

func _physics_process(delta):
    match state:
        EnemyState.CHASE:
            chase_player(delta)
        EnemyState.ATTACK:
            attack_player(delta)
        EnemyState.HURT:
            pass  # 等待動畫結束

func chase_player(delta):
    if target:
        var direction = (target.global_position - global_position).normalized()
        velocity = direction * speed
        move_and_slide()
        
        # 檢查是否進入攻擊範圍
        if global_position.distance_to(target.global_position) < attack_range:
            state = EnemyState.ATTACK

func attack_player(delta):
    if attack_timer <= 0:
        perform_attack()
        attack_timer = attack_cooldown
    else:
        attack_timer -= delta

func perform_attack():
    if target and global_position.distance_to(target.global_position) < attack_range:
        target.take_damage(damage)
        # 攻擊動畫
        play_attack_animation()

func take_damage(amount: int):
    current_hp -= amount
    state = EnemyState.HURT
    emit_signal("hit", amount)
    
    if current_hp <= 0:
        die()

func die():
    state = EnemyState.DEAD
    emit_signal("died")
    # 掉落TOKEN
    spawn_token()
    queue_free()

func spawn_token():
    var token = preload("res://scenes/pickups/token.tscn").instantiate()
    token.value = randi() % 15 + 5
    token.global_position = global_position
    get_parent().add_child(token)
```

### 4. 波次系統 (wave_manager.gd)

```gdscript
# wave_manager.gd
extends Node2D

signal wave_started(wave_number)
signal boss_spawned
signal wave_completed

@export var enemies_per_wave: int = 10
@export var wave_duration: float = 45.0
@export var max_waves: int = 10

var current_wave: int = 0
var wave_timer: float = 0.0
var enemies_remaining: int = 0
var spawn_timer: float = 0.0
var is_boss_wave: bool = false

@onready var enemy_spawner = $EnemySpawner
@onready var event_manager = $EventManager

func _process(delta):
    if current_wave > 0:
        wave_timer += delta
        
        # 檢查波次結束
        if enemies_remaining <= 0 and not is_boss_wave:
            complete_wave()
        
        # 生成敵人
        spawn_timer -= delta
        if spawn_timer <= 0 and enemies_remaining > 0:
            spawn_enemy()
            spawn_timer = 2.0

func start_waves():
    current_wave = 1
    enemies_remaining = enemies_per_wave
    emit_signal("wave_started", current_wave)

func spawn_enemy():
    if is_boss_wave:
        spawn_boss()
        return
    
    # 從地圖邊緣生成
    var spawn_pos = get_spawn_position()
    var enemy_scene = enemy_spawner.get_random_enemy()
    var enemy = enemy_scene.instantiate()
    enemy.global_position = spawn_pos
    enemy.died.connect(_on_enemy_died)
    get_parent().add_child(enemy)
    enemies_remaining -= 1

func spawn_boss():
    is_boss_wave = true
    emit_signal("boss_spawned")
    var boss = preload("res://scenes/enemies/boss_supervisor.tscn").instantiate()
    boss.global_position = get_spawn_position()
    boss.died.connect(_on_boss_died)
    get_parent().add_child(boss)

func _on_enemy_died():
    enemies_remaining -= 1

func _on_boss_died():
    emit_signal("wave_completed")
    is_boss_wave = false
    current_wave += 1
    
    if current_wave > max_waves:
        game_win()
    else:
        enemies_remaining = enemies_per_wave + (current_wave * 2)

func complete_wave():
    wave_timer = 0.0
    # 可能有隨機事件
    event_manager.try_trigger_event()
    current_wave += 1
    enemies_remaining = enemies_per_wave + (current_wave * 2)

func get_spawn_position() -> Vector2:
    # 從玩家周圍隨機位置生成
    var player = get_tree().get_first_node_in_group("player")
    var angle = randf() * TAU
    var distance = 800  # 生成在半徑800外
    return player.global_position + Vector2(cos(angle), sin(angle)) * distance
```

### 5. 小地圖系統 (minimap.gd)

```gdscript
# minimap.gd
extends Control

@export var map_size: Vector2 = Vector2(3000, 3000)
@export var minimap_size: Vector2 = Vector2(150, 150)
@export var update_interval: float = 0.1

var scale: float
var update_timer: float = 0.0
var enemies: Array = []
var pickups: Array = []

func _ready():
    # 計算縮放比例
    scale = minimap_size.x / map_size.x
    $MiniMapBG.size = minimap_size
    $MiniMapBG.position = Vector2(
        get_viewport_rect().size.x - minimap_size.x - 10,
        10
    )

func _process(delta):
    update_timer += delta
    if update_timer >= update_interval:
        update_timer = 0.0
        queue_redraw()

func _draw():
    # 繪製背景
    draw_rect(Rect2(Vector2.ZERO, minimap_size), Color.BLACK.with_alpha(0.5))
    
    # 繪製邊框
    draw_rect(Rect2(Vector2.ZERO, minimap_size), Color.WHITE.with_alpha(0.2), false, 1.0)
    
    # 繪製敵人
    for enemy in get_tree().get_nodes_in_group("enemies"):
        var pos = world_to_minimap(enemy.global_position)
        var color = Color.RED if "boss" in enemy.name else Color.ORANGE
        draw_circle(pos, 2.0 if "boss" in enemy.name else 1.0, color)
    
    # 繪製道具
    for pickup in get_tree().get_nodes_in_group("pickups"):
        var pos = world_to_minimap(pickup.global_position)
        draw_circle(pos, 1.0, Color.YELLOW)
    
    # 繪製玩家 (固定在中央)
    var player_pos = world_to_minimap(get_tree().get_first_node_in_group("player").global_position)
    draw_polygon(Vector2Array([
        player_pos + Vector2(0, -4),
        player_pos + Vector2(-3, 4),
        player_pos + Vector2(3, 4)
    ]), Color.GREEN)

func world_to_minimap(world_pos: Vector2) -> Vector2:
    var player = get_tree().get_first_node_in_group("player")
    var relative_pos = world_pos - player.global_position
    return (relative_pos * scale) + (minimap_size / 2)
```

### 6. 武器系統 (weapon.gd)

```gdscript
# weapon.gd
extends Node2D

@export var damage: int = 10
@export var attack_speed: float = 1.0
@export var range: float = 50.0
@export var knockback: float = 0.0

var cooldown_timer: float = 0.0
var is_attacking: bool = false

func _ready():
    add_to_group("weapons")

func _process(delta):
    if cooldown_timer > 0:
        cooldown_timer -= delta

func attack(direction: Vector2):
    if cooldown_timer <= 0:
        perform_attack(direction)
        cooldown_timer = 1.0 / attack_speed

func perform_attack(direction: Vector2):
    is_attacking = true
    rotation = direction.angle()
    
    # 創建攻擊區域
    var attack_area = $AttackArea
    attack_area.rotation = direction.angle()
    
    # 檢測範圍內敵人
    var enemies = attack_area.get_overlapping_bodies()
    for enemy in enemies:
        enemy.take_damage(damage)
        if knockback > 0:
            enemy.apply_knockback(direction * knockback)
    
    # 播放動畫/特效
    play_attack_effect()
    
    await get_tree().create_timer(0.1).timeout
    is_attacking = false

func play_attack_effect():
    # 播放武器動畫
    $Sprite2D.play("attack")
    # 播放音效
    # AudioManager.play_sfx("sword_swing")
```

### 7. 存檔系統 (save_system.gd)

```gdscript
# save_system.gd
extends Node

const SAVE_PATH = "user://save_game.dat"

func save_game():
    var save_data = {
        "player": {
            "stage": Global.stage,
            "level": Global.level,
            "tokens": Global.tokens,
            "hp": Global.current_hp,
            "weapons": Global.equipped_weapons,
            "skills": Global.unlocked_skills,
            "hardware": Global.hardware
        },
        "stats": {
            "total_kills": Global.total_kills,
            "total_time": Global.play_time,
            "high_scores": Global.high_scores
        }
    }
    
    var file = FileAccess.open(SAVE_PATH, FileAccess.WRITE)
    file.store_var(save_data)
    file.close()

func load_game():
    if not FileAccess.file_exists(SAVE_PATH):
        return false
    
    var file = FileAccess.open(SAVE_PATH, FileAccess.READ)
    var save_data = file.get_var()
    file.close()
    
    Global.stage = save_data.player.stage
    Global.level = save_data.player.level
    Global.tokens = save_data.player.tokens
    Global.current_hp = save_data.player.hp
    # ... 恢復其他數據
    return true
```

---

## ⚙️ Godot 4 專案設定

### project.godot 關鍵設定

```ini
[display]
window/size/viewport_width=1280
window/size/viewport_height=720
window/stretch/mode="viewport"
window/stretch/aspect="keep"

[rendering]
textures/canvas_textures/default_texture_filter=1  # Nearest (像素感)

[input]
move_left={ "deadzone": 0.5, "events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"alt":false,"shift":false,"control":false,"meta":false,"command":false,"pressed":false,"scancode":65,"physical_scancode":0,"unicode":0,"echo":false,"script":null) ] }
move_right={ "deadzone": 0.5, "events": [...] }
move_up={ ... }
move_down={ ... }
attack={ "deadzone": 0.5, "events": [Object(InputEventMouseButton,"resource_local_to_scene":false,"resource_name":"","device":-1,"alt":false,"shift":false,"control":false,"meta":false,"command":false,"pressed":false,"button_mask":1,"position":Vector2(0, 0),"global_position":Vector2(0, 0),"factor":1.0,"button_index":1,"canceled":false,"script":null) ] }
dash={ "deadzone": 0.5, "events": [Object(InputEventKey,"resource_local_to_scene":false,"resource_name":"","device":-1,"alt":false,"shift":false,"control":false,"meta":false,"command":false,"pressed":false,"scancode":4194325,"physical_scancode":0,"unicode":0,"echo":false,"script":null) ] }
```

---

## 🗺️ TileMap 地圖系統

```gdscript
# 使用 TileMap 創建地圖
@onready var tilemap = $TileMap

# 圖層設定
# Layer 1: Ground (地面)
# Layer 2: Walls (障礙物)
# Layer 3: Decoration (裝飾)
# Layer 4: Traps (陷阱)

# 生成障礙物
func place_traps():
    var trap_scene = preload("res://scenes/traps/oil_slick.tscn")
    for pos in trap_positions:
        var trap = trap_scene.instantiate()
        trap.global_position = tilemap.map_to_local(pos)
        add_child(trap)

# 碰撞層
func _ready():
    # 設置碰撞層
    tilemap.set_collision_layer_bit(0, true)  # Ground
    tilemap.set_collision_mask_bit(0, true)   # 可以行走的區域
```

---

## 🎮 控制設定 (Input Map)

| 功能 | 鍵盤 | 手掣 |
|------|------|------|
| 移動 | WASD | 左搖桿 |
| 瞄準 | 鼠標 | 右搖桿 |
| 攻擊 | 左鍵 | A/X |
| 衝刺 | Shift | LB |
| 武器1-6 | 1-6 | LB/RB |
| 商店 | Tab | Start |
| 地圖 | M | Select |

---

## 📦 第三方資源

### 需要的免費資源
```
- Kenney.nl 像素素材 (placeholder)
- Godot Asset Library 工具
- 免費像素字體 (Google Fonts: Press Start 2P)
```

### 音效 (可後加)
```
- 我可以幫你生成文字轉語音
- 需要音效時再添加
```

---

## 🚀 開發階段規劃

### 第一階段: MVP (1-2天)
```
Day 1:
- 創建 Godot 專案
- 設置2D環境
- 基本移動 + 瞄準
- 測試敵人移動

Day 2:
- 波次系統
- 武器攻擊
- 掉落系統
- 基礎HUD
```

### 第二階段: 核心玩法 (3-5天)
```
- 武器切換系統
- 技能樹系統
- 商店系統
- 所有敵人類型
```

### 第三階段: 內容 (1-2週)
```
- 8個關卡
- BOSS戰
- 粒子特效
- 嘲諷對話
```

---

## 📁 最終Repo結構

```
SURVIVE-TXT/
├── game/
│   ├── project.godot
│   ├── scenes/
│   ├── scripts/
│   ├── assets/
│   └── shaders/
└── docs/  (呢個係你現在睇緊的)
    ├── README.md
    ├── GAME_DESIGN.md
    └── ...
```

---

## ❓ 常見問題

**Q: 需要多少時間完成？**
A: MVP 1-2天，完整版 1-2個月

**Q: 一個人可以完成嗎？**
A: 可以，但需要時間。建議分工或分階段

**Q: 像素素材從邊度來？**
A: 免費資源 + 以後自定義

**Q: 音效/音樂點整？**
A: 我可以幫你生成文字轉語音，音樂稍後添加

---

準備好開始開發了嗎？我可以幫你生成初始的 Godot 專案代碼！