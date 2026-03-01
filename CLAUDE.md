# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

OpenStreetMapの道路データを使ったゾンビ感染シミュレーション。ブラウザ上で動作するクライアントサイドのみのWebアプリケーション。

### シミュレーションルール
- ゾンビ（赤）は一番近い人間（青）の方向に向かって移動する
- 人間は一番近いゾンビの反対方向に向かって移動する
- 人間がゾンビと接触するとゾンビになる

## 技術スタック

- **地図表示**: Leaflet.js（Canvas renderer使用）
- **地図タイル**: CartoDB Dark Matter（ダークテーマ）
- **道路データ取得**: Overpass API（OpenStreetMap）
- **構成**: サーバー不要。単一HTMLファイル（`index.html`）で完結

## 開発・実行

```bash
# ローカルサーバーで実行（いずれか）
python3 -m http.server 8000
npx serve .
open index.html  # 直接ブラウザで開いても動作する
```

テスト・ビルド・リントのツールチェーンは未設定。

## アーキテクチャ

### データフロー
1. ユーザーがマップをクリック → 座標取得
2. Overpass APIで周辺道路（`highway`タグ）をbbox指定で取得
3. OSMのnode/wayデータからグラフ（隣接リスト）を構築
4. グラフのエッジ上にエージェントをランダム配置
5. `requestAnimationFrame`ループでシミュレーション実行

### グラフ構造
- **頂点**: OSMノード（`id` → `{lat, lon}`）
- **辺**: way内の連続ノード間の道路セグメント（重みはHaversine距離）
- **無向グラフ**（双方向通行）

### エージェントの状態
各エージェントはグラフのエッジ上に存在し、`from`ノード、`to`ノード、`progress`（0.0〜1.0）で位置を表現。任意の緯度経度ではなく、道路上に制約される。

### 経路探索
- **ゾンビ**: A*アルゴリズムで最寄りの人間へ向かう
- **人間**: 最寄りゾンビから最も遠い隣接ノードへ貪欲に逃げる
- 経路再計算は0.5秒ごと（毎フレームではない）

### パフォーマンス想定
- 50〜200エージェント、近隣エリア規模（bbox半径約800m）で60fps動作を想定
- `L.circleMarker`＋Canvas rendererにより大量マーカーを効率的に描画
