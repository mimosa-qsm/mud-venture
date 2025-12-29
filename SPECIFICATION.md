# Mud-Venture Technical Specification
技術仕様書 / Technical Specification

[日本語](#日本語版) | [English](#english-version)

---

## 日本語版

### 📋 プロジェクト情報
- **プロジェクト名**: Mud-Venture
- **バージョン**: 1.0.0
- **作成日**: 2024年12月
- **ライセンス**: MIT

### 🏗️ アーキテクチャ

#### システム構成
```
┌─────────────────────────────────────┐
│         UI Layer (React)            │
│  - Game State Management            │
│  - User Input Handling              │
│  - HUD Display                      │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      Game Logic Layer               │
│  - Physics Simulation               │
│  - Collision Detection              │
│  - Mud System                       │
│  - Character Controller             │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│    Rendering Layer (Three.js)       │
│  - 3D Scene Management              │
│  - Camera Control                   │
│  - Material System                  │
│  - Lighting                         │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│    Audio Layer (Web Audio API)     │
│  - Sound Generation                 │
│  - Audio Context Management         │
└─────────────────────────────────────┘
```

### 🎮 コアシステム

#### 1. キャラクター制御システム

**キャラクター構造:**
```javascript
Player Group (THREE.Group)
├── Body (Cylinder) - y: 0.75
├── Head (Sphere) - y: 2.0
├── Hair Group - y: 2.0~0.4
│   ├── Hair1~9 (Spheres)
├── Eyes (2x Sphere) - y: 2.1
└── Legs (2x Cylinder) - y: -0.5
```

**移動システム:**
- 基本速度: 0.15 units/frame
- 泥の影響: `速度 = 基本速度 × 泥の移動係数`
- 境界: X: [-55, 55], Z: [-55, 55]

**キー入力処理:**
```javascript
keysRef.current = {
  'w': boolean,
  'a': boolean,
  's': boolean,
  'd': boolean,
  'arrowup': boolean,
  'arrowleft': boolean,
  'arrowdown': boolean,
  'arrowright': boolean
}
```

#### 2. 泥システム

**泥ゾーン定義:**
```javascript
{
  x: number,           // 左上X座標
  z: number,           // 左上Z座標
  width: number,       // 幅
  depth: number,       // 奥行き
  type: string,        // 泥タイプ
  color: hex,          // 色（16進数）
  stickiness: float,   // 粘着度 (0.0-1.0)
  moveSpeed: float     // 移動速度係数 (0.0-1.0)
}
```

**6種類の泥タイプ:**

| タイプ | 色コード | 粘着度 | 移動速度 | サイズ |
|--------|---------|--------|----------|--------|
| Clay | 0x8B7355 | 0.25 | 0.4 | 20x20 |
| Volcanic | 0x6B6B6B | 0.35 | 0.25 | 22x18 |
| Peat | 0x3D2817 | 0.5 | 0.2 | 24x20 |
| Slime | 0x4CAF50 | 0.4 | 0.35 | 18x18 |
| Sludge | 0x3E2723 | 0.55 | 0.15 | 15x15 |
| Snow | 0xE8F5E9 | 0.3 | 0.5 | 15x15 |

**沈み込みメカニクス:**
```javascript
// 泥の中にいる場合
if (inMud && !justEscaped) {
  sinkDepth += 0.015; // 毎フレーム0.015沈む
  sinkDepth = Math.min(2.5, sinkDepth); // 最大2.5
}

// 泥から出た場合
if (!inMud) {
  sinkDepth -= 0.08; // 毎フレーム0.08浮上
  sinkDepth = Math.max(0, sinkDepth);
}
```

**泥カバーレベル:**
```
Level 0 (Clean): sinkDepth = 0
Level 1 (Legs): sinkDepth > 0.3
Level 2 (Torso): sinkDepth > 0.8
Level 3 (Shoulders): sinkDepth > 1.5
Level 4 (Head): sinkDepth > 2.0
```

#### 3. ハマりシステム

**ランダムハマり条件:**
```javascript
const deepEnough = sinkDepth > 1.0;
const randomStuck = deepEnough && Math.random() < 0.02;

if (randomStuck && !stuck) {
  sinkDepth = 15.0; // 突然深く沈む
  stuck = true;
}
```

**もがきメカニクス:**
```javascript
// キー入力カウント
struggleInputs += 1; // キーを押すごとに+1

// 浮上処理（もがいている間）
if (pressing) {
  sinkDepth -= 0.08; // 毎フレーム0.08浮上
  sinkDepth = Math.max(1.5, sinkDepth);
}

// 沈下処理（もがいていない間）
if (!pressing) {
  sinkDepth += 0.04; // 毎フレーム0.04沈む
  sinkDepth = Math.min(15.0, sinkDepth);
}

// 脱出条件
if (struggleInputs >= 10) {
  stuck = false;
  sinkDepth = 0;
  justEscaped = true;
}
```

#### 4. 泥色ブレンドシステム

**色遷移ロジック:**
```javascript
// 新しい泥に入った時
if (currentMudType !== inMud.type) {
  currentMudType = inMud.type;
  mudTransition = 0; // 遷移開始
}

// 徐々にブレンド
mudTransition += 0.02; // 毎フレーム2%
mudTransition = Math.min(1, mudTransition); // 最大100%

// 色の補間
if (mudTransition < 1) {
  const blendedColor = oldColor.lerp(newColor, mudTransition);
  material.color = blendedColor;
}
```

**体パーツごとの管理:**
- 各体パーツが前の泥の色を `userData.previousMudColor` に保存
- 新しい泥に入ると、保存された色から新しい色へ線形補間
- 遷移完了後、新しい色を保存

#### 5. カメラシステム

**カメラ設定:**
```javascript
Camera: PerspectiveCamera
- FOV: 60°
- Aspect: 800/600
- Near: 0.1
- Far: 1000
```

**カメラコントロール:**
```javascript
// 回転（マウスドラッグ）
angle += deltaX * 0.01;

// ズーム（マウススクロール）
distance += deltaY * 0.01;
distance = Math.max(8, Math.min(25, distance));

// カメラ位置計算
camera.position.x = player.x + Math.sin(angle) * distance;
camera.position.z = player.z + Math.cos(angle) * distance;
camera.position.y = player.y + height;
camera.lookAt(player.position);
```

#### 6. オーディオシステム

**効果音タイプ:**

1. **Low Squelch** (低音もがき音)
   - 周波数: 60Hz → 30Hz
   - 時間: 0.25秒
   - 音量: 0.12

2. **High Squelch** (高音もがき音)
   - 周波数: 90Hz → 45Hz
   - 時間: 0.18秒
   - 音量: 0.1

3. **Pop** (脱出音)
   - 周波数: 150Hz → 50Hz
   - 時間: 0.3秒
   - 音量: 0.2

4. **Step** (歩行音)
   - 周波数: 80Hz → 40Hz
   - 時間: 0.15秒
   - 音量: 0.08
   - 発生確率: 5%/フレーム

**オーディオ生成:**
```javascript
const oscillator = ctx.createOscillator();
const gainNode = ctx.createGain();

oscillator.frequency.setValueAtTime(startFreq, ctx.currentTime);
oscillator.frequency.exponentialRampToValueAtTime(endFreq, ctx.currentTime + duration);
gainNode.gain.setValueAtTime(startGain, ctx.currentTime);
gainNode.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + duration);
```

### 🎨 レンダリング

**シーン構成:**
```
Scene
├── Ambient Light (強度: 0.6)
├── Directional Light (強度: 0.8, 影あり)
├── Ground Plane (120x120, 緑)
├── Mud Planes (6個, 各種色)
├── Mud Labels (6個, テキストテクスチャ)
└── Player Group
```

**マテリアル:**
- Ground: MeshLambertMaterial (0x8FBC8F)
- Mud: MeshLambertMaterial (各泥固有の色)
- Character: MeshLambertMaterial (動的に変更)

**ライティング:**
- Ambient: 環境光（全体を明るく）
- Directional: 方向光（影の生成）
  - Position: (20, 30, 20)
  - Shadow map: 2048x2048

### 📊 パフォーマンス

**ゲームループ:**
- フレームレート: 60 FPS
- 更新間隔: 1000/60 = 16.67ms

**最適化:**
- シンプルなジオメトリ（低ポリゴン）
- マテリアルの再利用
- 影マップの最適化

### 🔧 技術的制約

**現在の制約:**
1. **完全水没の視覚化**
   - キャラクターの頭（y: 2.0）が完全に消えない
   - depth = 15でも一部が見える場合がある
   - 原因: カメラ位置とキャラクター構造の相対的な問題

2. **効果音の質**
   - Web Audio APIによる合成音
   - 実際の泥音ではない

3. **キャラクターモデル**
   - プリミティブ形状の組み合わせ
   - 詳細なモデルではない

### 🛠️ 依存関係

```json
{
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "three": "^0.128.0"
  }
}
```

**CDN ライブラリ:**
- React: unpkg.com
- Three.js: cdnjs.cloudflare.com
- Babel Standalone: unpkg.com

---

## English Version

### 📋 Project Information
- **Project Name**: Mud-Venture
- **Version**: 1.0.0
- **Created**: December 28, 2024
- **License**: MIT

### 🏗️ Architecture

#### System Structure
```
┌─────────────────────────────────────┐
│         UI Layer (React)            │
│  - Game State Management            │
│  - User Input Handling              │
│  - HUD Display                      │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│      Game Logic Layer               │
│  - Physics Simulation               │
│  - Collision Detection              │
│  - Mud System                       │
│  - Character Controller             │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│    Rendering Layer (Three.js)       │
│  - 3D Scene Management              │
│  - Camera Control                   │
│  - Material System                  │
│  - Lighting                         │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│    Audio Layer (Web Audio API)     │
│  - Sound Generation                 │
│  - Audio Context Management         │
└─────────────────────────────────────┘
```

### 🎮 Core Systems

#### 1. Character Control System

**Character Structure:**
```javascript
Player Group (THREE.Group)
├── Body (Cylinder) - y: 0.75
├── Head (Sphere) - y: 2.0
├── Hair Group - y: 2.0~0.4
│   ├── Hair1~9 (Spheres)
├── Eyes (2x Sphere) - y: 2.1
└── Legs (2x Cylinder) - y: -0.5
```

**Movement System:**
- Base speed: 0.15 units/frame
- Mud effect: `speed = baseSpeed × mudMoveSpeed`
- Boundaries: X: [-55, 55], Z: [-55, 55]

**Input Handling:**
```javascript
keysRef.current = {
  'w': boolean,
  'a': boolean,
  's': boolean,
  'd': boolean,
  'arrowup': boolean,
  'arrowleft': boolean,
  'arrowdown': boolean,
  'arrowright': boolean
}
```

#### 2. Mud System

**Mud Zone Definition:**
```javascript
{
  x: number,           // Top-left X coordinate
  z: number,           // Top-left Z coordinate
  width: number,       // Width
  depth: number,       // Depth
  type: string,        // Mud type
  color: hex,          // Color (hexadecimal)
  stickiness: float,   // Stickiness (0.0-1.0)
  moveSpeed: float     // Movement speed multiplier (0.0-1.0)
}
```

**6 Mud Types:**

| Type | Color Code | Stickiness | Move Speed | Size |
|------|-----------|-----------|------------|------|
| Clay | 0x8B7355 | 0.25 | 0.4 | 20x20 |
| Volcanic | 0x6B6B6B | 0.35 | 0.25 | 22x18 |
| Peat | 0x3D2817 | 0.5 | 0.2 | 24x20 |
| Slime | 0x4CAF50 | 0.4 | 0.35 | 18x18 |
| Sludge | 0x3E2723 | 0.55 | 0.15 | 15x15 |
| Snow | 0xE8F5E9 | 0.3 | 0.5 | 15x15 |

**Sinking Mechanics:**
```javascript
// When in mud
if (inMud && !justEscaped) {
  sinkDepth += 0.015; // Sink 0.015 per frame
  sinkDepth = Math.min(2.5, sinkDepth); // Max 2.5
}

// When out of mud
if (!inMud) {
  sinkDepth -= 0.08; // Rise 0.08 per frame
  sinkDepth = Math.max(0, sinkDepth);
}
```

**Mud Coverage Levels:**
```
Level 0 (Clean): sinkDepth = 0
Level 1 (Legs): sinkDepth > 0.3
Level 2 (Torso): sinkDepth > 0.8
Level 3 (Shoulders): sinkDepth > 1.5
Level 4 (Head): sinkDepth > 2.0
```

#### 3. Stuck System

**Random Stuck Conditions:**
```javascript
const deepEnough = sinkDepth > 1.0;
const randomStuck = deepEnough && Math.random() < 0.02;

if (randomStuck && !stuck) {
  sinkDepth = 15.0; // Suddenly sink deep
  stuck = true;
}
```

**Struggle Mechanics:**
```javascript
// Key press counting
struggleInputs += 1; // +1 per key press

// Rising (while struggling)
if (pressing) {
  sinkDepth -= 0.08; // Rise 0.08 per frame
  sinkDepth = Math.max(1.5, sinkDepth);
}

// Sinking (while not struggling)
if (!pressing) {
  sinkDepth += 0.04; // Sink 0.04 per frame
  sinkDepth = Math.min(15.0, sinkDepth);
}

// Escape condition
if (struggleInputs >= 10) {
  stuck = false;
  sinkDepth = 0;
  justEscaped = true;
}
```

#### 4. Mud Color Blending System

**Color Transition Logic:**
```javascript
// When entering new mud
if (currentMudType !== inMud.type) {
  currentMudType = inMud.type;
  mudTransition = 0; // Start transition
}

// Gradually blend
mudTransition += 0.02; // 2% per frame
mudTransition = Math.min(1, mudTransition); // Max 100%

// Color interpolation
if (mudTransition < 1) {
  const blendedColor = oldColor.lerp(newColor, mudTransition);
  material.color = blendedColor;
}
```

**Per-Body-Part Management:**
- Each body part stores previous mud color in `userData.previousMudColor`
- Linear interpolation from stored color to new color when entering new mud
- Store new color after transition completes

#### 5. Camera System

**Camera Settings:**
```javascript
Camera: PerspectiveCamera
- FOV: 60°
- Aspect: 800/600
- Near: 0.1
- Far: 1000
```

**Camera Control:**
```javascript
// Rotation (mouse drag)
angle += deltaX * 0.01;

// Zoom (mouse scroll)
distance += deltaY * 0.01;
distance = Math.max(8, Math.min(25, distance));

// Camera position calculation
camera.position.x = player.x + Math.sin(angle) * distance;
camera.position.z = player.z + Math.cos(angle) * distance;
camera.position.y = player.y + height;
camera.lookAt(player.position);
```

#### 6. Audio System

**Sound Effect Types:**

1. **Low Squelch** (low-pitched struggling sound)
   - Frequency: 60Hz → 30Hz
   - Duration: 0.25s
   - Volume: 0.12

2. **High Squelch** (high-pitched struggling sound)
   - Frequency: 90Hz → 45Hz
   - Duration: 0.18s
   - Volume: 0.1

3. **Pop** (escape sound)
   - Frequency: 150Hz → 50Hz
   - Duration: 0.3s
   - Volume: 0.2

4. **Step** (walking sound)
   - Frequency: 80Hz → 40Hz
   - Duration: 0.15s
   - Volume: 0.08
   - Probability: 5%/frame

**Audio Generation:**
```javascript
const oscillator = ctx.createOscillator();
const gainNode = ctx.createGain();

oscillator.frequency.setValueAtTime(startFreq, ctx.currentTime);
oscillator.frequency.exponentialRampToValueAtTime(endFreq, ctx.currentTime + duration);
gainNode.gain.setValueAtTime(startGain, ctx.currentTime);
gainNode.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + duration);
```

### 🎨 Rendering

**Scene Composition:**
```
Scene
├── Ambient Light (Intensity: 0.6)
├── Directional Light (Intensity: 0.8, with shadows)
├── Ground Plane (120x120, green)
├── Mud Planes (6 units, various colors)
├── Mud Labels (6 units, text textures)
└── Player Group
```

**Materials:**
- Ground: MeshLambertMaterial (0x8FBC8F)
- Mud: MeshLambertMaterial (each mud's unique color)
- Character: MeshLambertMaterial (dynamically changed)

**Lighting:**
- Ambient: Environment light (overall brightness)
- Directional: Directional light (shadow generation)
  - Position: (20, 30, 20)
  - Shadow map: 2048x2048

### 📊 Performance

**Game Loop:**
- Frame rate: 60 FPS
- Update interval: 1000/60 = 16.67ms

**Optimization:**
- Simple geometry (low polygon)
- Material reuse
- Shadow map optimization

### 🔧 Technical Limitations

**Current Limitations:**
1. **Complete Submersion Visualization**
   - Character head (y: 2.0) doesn't completely disappear
   - Still partially visible even at depth = 15
   - Cause: Relative issue between camera position and character structure

2. **Sound Quality**
   - Synthesized sounds via Web Audio API
   - Not actual mud sounds

3. **Character Model**
   - Combination of primitive shapes
   - Not a detailed model

### 🛠️ Dependencies

```json
{
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "three": "^0.128.0"
  }
}
```

**CDN Libraries:**
- React: unpkg.com
- Three.js: cdnjs.cloudflare.com
- Babel Standalone: unpkg.com

---

## 📝 Notes

### Known Issues
1. Character doesn't completely disappear when fully submerged
2. Placeholder character model and sound effects
3. Camera angles may make it difficult to see complete submersion

### Future Improvements Priority
See README.md for detailed future development plans.

---

**Document Version**: 1.0.0  
**Last Updated**: December 28, 2024
