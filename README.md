# Mud-Venture

[日本語](##日本語) | [English](##english)

---

## 日本語

### 🎮 概要
**Mud-Venture**は、様々な種類の泥沼を探検し、キャラクターが泥まみれになる3D探索ゲームです。リラックスした雰囲気の中で、泥に沈んだり、もがいて脱出したりする体験を楽しめます。

### ✨ 主な機能
- **6種類の泥沼**：Clay（粘土）、Volcanic（火山泥）、Peat（泥炭）、Slime（スライム）、Sludge（汚泥）、Snow（雪）
- **動的な泥カバーシステム**：足→胴体→肩→頭の順に段階的に泥まみれになる
- **泥のレイヤー/色ブレンド**：異なる泥を移動すると、前の泥の色から新しい泥の色へ徐々に変化
- **ランダムハマりシステム**：泥の中を歩いていると、突然深く沈んでハマる
- **もがきメカニクス**：キーを10回入力して脱出
- **泥タイプ別の移動速度**：粘度によって歩行速度が変化
- **効果音**：歩行音、ハマる音、もがく音など
- **自由なカメラ操作**：マウスドラッグで回転、スクロールでズーム

### 🎯 操作方法
- **WASD / 矢印キー**：移動
- **マウスドラッグ**：カメラ回転
- **マウススクロール**：ズームイン/アウト
- **Clean Upボタン**：泥を洗い流す

### 🌊 泥の種類と特性

| 泥タイプ | 色 | 移動速度 | 粘度 |
|---------|-----|---------|------|
| Snow (雪) | 淡緑 | 50% | 低 |
| Clay (粘土) | 茶色 | 40% | 中低 |
| Slime (スライム) | 緑 | 35% | 中 |
| Volcanic (火山泥) | 灰色 | 25% | 中高 |
| Peat (泥炭) | 黒褐色 | 20% | 高 |
| Sludge (汚泥) | 暗褐色 | 15% | 最高 |

### 📁 ファイル構成
```
mud-venture/
├── index.html          # ゲーム本体（スタンドアロン版）
├── README.md          # このファイル
└── SPECIFICATION.md   # 技術仕様書
```

### 🚀 遊び方
1. `index.html` をブラウザで開く
2. 泥沼に入って探検
3. 深く沈むとランダムでハマる
4. キーを連打してもがき、脱出！

### 🛠️ 技術スタック
- **React 18** - UI フレームワーク
- **Three.js r128** - 3D グラフィックス
- **Web Audio API** - 効果音生成
- **Vanilla JavaScript** - ゲームロジック

### 📝 ライセンス
**MIT License** - 完全に自由です！

このプロジェクトは**他力本願プロジェクト**です。
- フォーク・改造・再配布、全部自由！
- クレジット不要（あると嬉しいけど）
- 商用利用もOK
- **好き勝手にやってください！**

---

## 🔮 実装してほしい機能（他力本願リスト）

誰か作ってくれたら超嬉しいやつ：

### 超欲しい
- [ ] **完全水没システムの実装**
  - キャラクターが完全に泥に埋まって見えなくなる仕様
  - 現在は技術的制約で頭が残る状態

- [ ] **プレースホルダーの置き換え**
  - キャラクターモデルを本格的な3Dモデルに変更
  - 効果音を実際の泥音に変更（現在は合成音）

### 優先度：中
- [ ] **カスタムキャラクター対応**
  - **VRM対応**：VTuberアバターなど、好きなキャラクターで遊べる
  - **MMD対応**：MMDモデルのインポート
  - アバターアップロード機能
  - なんか悪用されそうなので実装は自己責任でお願いします

- [ ] **泥のバリエーション追加**
  - クイックサンド（流砂）
  - タールピット
  - 生コンクリート
  - ケーキなどの生地

### 優先度：低
- [ ] **マルチプレイヤー**：友達と一緒に探検
- [ ] **フォトモード**：泥まみれのキャラクターをスクリーンショット
- [ ] **実績システム**：すべての泥タイプを体験など
- [ ] **カスタマイゼーション**：服装の変更、髪型の変更
- [ ] **天候システム**：雨で泥が滑りやすくなる

### 🎨 アート・デザイン改善
- [ ] より現実的な泥のテクスチャ
- [ ] パーティクルエフェクト（泥の飛び散り）
- [ ] よりリアルなキャラクターモデル
- [ ] セル調シェーダー（アニメ風グラフィック）

### 🔊 オーディオ改善
- [ ] 実際の泥音サンプルの録音・使用
- [ ] 環境音（鳥のさえずり、風の音）
- [ ] もがき時の声（オプション）

---

## English

### 🎮 Overview
**Mud-Venture** is a 3D exploration game where you explore various types of mud and get your character muddy. Enjoy a relaxing experience of sinking into mud and struggling to escape.

### ✨ Main Features
- **6 Types of Mud**: Clay, Volcanic, Peat, Slime, Sludge, Snow
- **Dynamic Mud Coverage System**: Progressive coverage from legs → torso → shoulders → head
- **Mud Layer/Color Blending**: Colors gradually transition when moving between different mud types
- **Random Stuck System**: Randomly sink deeply while walking through mud
- **Struggle Mechanics**: Press movement keys 10 times to escape
- **Mud-Type Specific Movement Speed**: Walking speed varies by viscosity
- **Sound Effects**: Walking sounds, stuck sounds, struggling sounds
- **Free Camera Control**: Rotate with mouse drag, zoom with scroll

### 🎯 Controls
- **WASD / Arrow Keys**: Move
- **Mouse Drag**: Rotate camera
- **Mouse Scroll**: Zoom in/out
- **Clean Up Button**: Wash off mud

### 🌊 Mud Types & Properties

| Mud Type | Color | Move Speed | Viscosity |
|----------|-------|------------|-----------|
| Snow | Light Green | 50% | Low |
| Clay | Brown | 40% | Medium-Low |
| Slime | Green | 35% | Medium |
| Volcanic | Gray | 25% | Medium-High |
| Peat | Dark Brown | 20% | High |
| Sludge | Very Dark Brown | 15% | Highest |

### 📁 File Structure
```
mud-venture/
├── index.html          # Game (standalone version)
├── README.md          # This file
└── SPECIFICATION.md   # Technical specification
```

### 🚀 How to Play
1. Open `index.html` in your browser
2. Enter mud zones and explore
3. Randomly get stuck when sinking deep
4. Mash keys to struggle and escape!

### 🛠️ Tech Stack
- **React 18** - UI Framework
- **Three.js r128** - 3D Graphics
- **Web Audio API** - Sound Effect Generation
- **Vanilla JavaScript** - Game Logic

### 📝 License
MIT License - Free to modify and distribute

---

## 🔮 Future Development Plans

### High Priority
- [ ] **Complete Submersion System**
  - Character completely disappears when stuck in mud
  - Currently has technical limitations (head remains visible)

- [ ] **Replace Placeholders**
  - Replace character model with proper 3D model
  - Replace sound effects with actual mud sounds (currently synthesized)

### Medium Priority
- [ ] **Custom Character Support**
  - **VRM Support**: Play with VTuber avatars and custom characters
  - **MMD Support**: Import MMD models
  - Avatar upload functionality
  - Due to potential misuse concerns, implementation is at your own risk!

- [ ] **Additional Mud Variations**
  - Quicksand
  - Tar pits
  - Concrete mud
  - Cake dough

### Low Priority
- [ ] **Multiplayer**: Explore with friends
- [ ] **Photo Mode**: Screenshot muddy characters
- [ ] **Achievement System**: Experience all mud types, etc.
- [ ] **Customization**: Change outfits, hairstyles
- [ ] **Weather System**: Rain makes mud slippery

### 🎨 Art & Design Improvements
- [ ] More realistic mud textures
- [ ] Particle effects (mud splatter)
- [ ] More realistic character models
- [ ] Cel-shaded graphics (anime style)

### 🔊 Audio Improvements
- [ ] Record and use actual mud sounds
- [ ] Ambient sounds (birds, wind)
- [ ] Struggling voice (optional)

---

## 🤝 Contributing
Feel free to fork this project and submit pull requests! Any improvements or new features are welcome.

## 📧 Contact
Share your gameplay on Twitter with **#MudVenture**!

---

Made with ❤️ for mud exploration enthusiasts
