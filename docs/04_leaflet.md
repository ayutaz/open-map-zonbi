# Leaflet.js

## 概要

Leaflet.jsは、インタラクティブな地図を構築するための軽量オープンソースJavaScriptライブラリ。

- **安定版**: v1.9.4
- **次期バージョン**: Leaflet 2.0.0-alpha.1（2025年8月リリース。ESM化、IE廃止、グローバル`L`削除など）
- CDN: `https://unpkg.com/leaflet@1.9.4/dist/leaflet.js`

## タイルプロバイダー

### OpenStreetMap標準
```
https://tile.openstreetmap.org/{z}/{x}/{y}.png
```

### CartoDB Dark Matter（本プロジェクトで使用）
```
https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png
```
- APIキー不要、帰属表示必須
- ライセンス: BSD 3-Clause + CC-BY 4.0
- バリエーション: `dark_all`(ラベルあり), `dark_nolabels`, `dark_only_labels`

### Stadia Maps
- 無料プラン: 月2,500クレジット（非商用のみ）
- APIキー必要
- 商用利用は有料プラン必須

### 帰属表示
```
© OpenStreetMap contributors © CARTO
```

## Canvas Renderer

`preferCanvas: true` を指定するとSVGの代わりにCanvas APIで描画。

```javascript
const map = L.map('map', { preferCanvas: true });
```

### SVGとの違い
- **SVG**: 各シェイプがDOM要素 → 大量描画で遅い
- **Canvas**: 単一Canvas上に全描画 → DOM生成なし、高速

### 注意点
- 一部プラグインやPanesでSVGにフォールバックすることがある
- 10万オブジェクト超ではCanvasでも限界

## L.circleMarker vs L.marker

| | L.marker | L.circleMarker |
|---|---------|---------------|
| 描画方式 | DOM要素（`<img>`/`<div>`） | ベクター（Canvas対応） |
| Canvas対応 | 不可 | 可能 |
| 大量描画 | ~1,000個が限界 | 数万個まで対応 |
| サイズ指定 | - | ピクセル単位（ズーム非連動） |
| カスタマイズ | アイコン画像 | 色・半径・透明度 |

**本プロジェクトではL.circleMarkerを使用** — エージェント表示に最適。

### パフォーマンスTips
- `weight: 0`（ストロークなし）で描画負荷を大幅軽減
- ストローク付き円 → 2つの塗りつぶし円に置き換えで約50%高速化

## 大量マーカーの最適化

| マーカー数 | 推奨アプローチ |
|-----------|--------------|
| ~1,000 | 通常の`L.marker` |
| 1,000〜10,000 | `preferCanvas` + `circleMarker` |
| 10,000〜50,000 | クラスタリング or Canvas circleMarker |
| 50,000〜500,000 | Supercluster + ビューポート最適化 |
| 500,000以上 | サーバーサイドクラスタリング |

関連ライブラリ:
- **Leaflet.markercluster**: 近接マーカーのグループ化
- **Supercluster**: 高速階層的クラスタリング（50万マーカーが1-2秒）
- **Leaflet.Canvas-Markers**: 通常markerもCanvas描画可能にするプラグイン
