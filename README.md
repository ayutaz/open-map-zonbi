# ゾンビ感染シミュレーション（open-map-zonbi）

OpenStreetMap の実際の道路データの上で、ゾンビ感染の広がりをシミュレーションするブラウザアプリです。地図をクリックした地点の周辺道路を取得し、人間（青）とゾンビ（赤）が道路上だけを移動します。サーバー不要で、`index.html` 1ファイルで完結しています。

**Live Demo**: https://ayutaz.github.io/open-map-zonbi/

## 遊びかた

1. 地図上の好きな場所をクリックすると、その周辺（緯度・経度 ±0.008°の範囲）の道路データを Overpass API から取得します。
2. 右のパネルで **人間の数**（10–200）、**ゾンビの数**（1–20）、**速度倍率**（0.5–5倍）を設定します。プリセット（パニック映画 / アウトブレイク / 絶望）も選べます。
3. **開始** でシミュレーションが始まります。一時停止・リセットもできます。
4. 人間とゾンビの数・経過時間・比率の推移がリアルタイムに表示され、全員がゾンビ化すると結果（所要時間と推移グラフ）が出ます。**もう一度** で同じエリアを再実行、**エリアを変える** で別の場所を選び直せます。

## シミュレーションのルール

- エージェントは道路グラフの辺の上だけを移動します（任意の緯度経度には移動しない）。道路は無向グラフとして扱います。
- **ゾンビ**（1.2 m/s）は、最寄りの人間へ A* 経路探索（ヒューリスティックは Haversine 距離）で向かいます。
- **人間**（1.8 m/s）は、最寄りのゾンビから最も遠くなる隣接ノードへ貪欲に逃げます。
- 人間がゾンビから 8 m 以内に入るとゾンビになります。
- 経路の再計算は 0.5 秒ごとです（毎フレームではない）。

## 仕組み

1. クリック地点を中心とした bbox で、Overpass API から `highway` タグ（residential / tertiary / secondary / primary / trunk / unclassified / living_street / pedestrian / footway / path / service）の道路を取得します。
   - エンドポイントは `overpass-api.de` → `overpass.kumi.systems` の順にフォールバックし、429 / 504 は指数バックオフ付きでリトライします（タイムアウト 30 秒）。
2. OSM の node / way から隣接リスト（辺の重みは Haversine 距離）を構築します。
3. 辺の上にエージェントをランダムに配置し、`requestAnimationFrame` のループで動かします。

| 用途 | 使用しているもの |
|------|------------------|
| 地図表示 | [Leaflet](https://leafletjs.com/) 1.9.4（Canvas renderer、unpkg から読み込み、失敗時は jsDelivr にフォールバック） |
| 地図タイル | CartoDB Dark Matter |
| 道路データ | [Overpass API](https://wiki.openstreetmap.org/wiki/Overpass_API)（OpenStreetMap） |

地図データ © [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors、タイル © [CARTO](https://carto.com/)。

## ローカルで動かす

ビルドは不要です。静的ファイルとして配信するだけで動きます。

```bash
python3 -m http.server 8000   # http://localhost:8000/ を開く
# または
npx serve .
```

`index.html` を直接ブラウザで開いても動作します。地図タイル・Leaflet・Overpass API の取得にインターネット接続が必要です。

テスト・ビルド・リントのツールチェーンは未設定です。

## ディレクトリ構成

```text
index.html          # アプリ本体（HTML / CSS / JavaScript をすべて含む）
docs/               # 技術調査（OpenStreetMap、highway タグ、Overpass API、Leaflet、道路グラフ）
docs/requirements/  # 要件定義（機能 / 技術 / UX・UI）
CLAUDE.md           # 設計の要点（データフロー、グラフ構造、経路探索）
```

## デプロイ

`master` ブランチへの push で [`.github/workflows/pages.yml`](./.github/workflows/pages.yml) がリポジトリの内容をそのまま GitHub Pages へ公開します。

## ライセンス

未定。
