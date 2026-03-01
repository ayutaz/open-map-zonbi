# Overpass API

## 概要

Overpass APIは、OpenStreetMapデータに対して読み取り専用でアクセスできるWeb API。クエリ条件に合致するOSMデータセットを返す「Web上のデータベースエンジン」。

## エンドポイント

| サーバー | URL | 特記事項 |
|---------|-----|---------|
| 公式 | `https://overpass-api.de/api/interpreter` | 2スロット/IP、Attic Data対応 |
| VK Maps | `https://maps.mail.ru/osm/tools/overpass/api/interpreter` | レート制限なし |
| Private.coffee | `https://overpass.private.coffee/api/interpreter` | レート制限なし |
| Kumi | `https://overpass.kumi.systems/api/interpreter` | グローバル |

ステータス確認: `https://overpass-api.de/api/status`

## Overpass QL 基本構文

### グローバル設定（クエリ先頭）
```
[out:json]          // 出力形式（json, xml, csv）
[timeout:30]        // タイムアウト秒数（デフォルト180秒）
[maxsize:536870912] // 最大メモリ使用量（バイト）
[bbox:南緯,西経,北緯,東経]  // グローバルバウンディングボックス
```

### クエリ文
```
<オブジェクト型><フィルタ>;
```
オブジェクト型: `node`, `way`, `relation`, `nwr`（全て）

### タグフィルタ
```
["key"]            // キーが存在する
["key"="value"]    // 完全一致
["key"~"regex"]    // 正規表現マッチ
["key"!="value"]   // 不一致
[!"key"]           // キーが存在しない
```

### 出力文
```
out body;    // ID、子要素、タグ（デフォルト）
out skel;    // タグなし、座標のみ
out geom;    // wayにジオメトリ（座標列）を付与
out center;  // wayの中心座標を付与
```

### 再帰展開
```
(._;>;);   // 現在のセット + 参照するノード全て
```
wayはノードIDリストのみ持つため、座標を得るには `>` で展開するか `out geom` を使う。

## 道路データ取得クエリ例

### 本プロジェクトで使用するクエリ
```
[out:json][timeout:30];
way["highway"~"^(residential|tertiary|secondary|primary|trunk|
  unclassified|living_street|pedestrian|footway|path|service)$"]
  (南緯,西経,北緯,東経);
(._;>;);
out body;
```

### エリア名で指定する方法
```
[out:json][timeout:60];
area["name"="渋谷区"]->.searchArea;
way["highway"~"^(primary|secondary|tertiary)$"](area.searchArea);
out geom;
```

## bbox指定

書式: `(南緯, 西経, 北緯, 東経)` — 10進数度数表記

3つの指定方法:
1. クエリ文に直接: `node["amenity"](35.6,139.7,35.8,139.9);`
2. グローバル設定: `[bbox:35.6,139.7,35.8,139.9];`
3. Overpass Turbo: `({{bbox}})` で地図表示範囲を自動代入

## レスポンスJSON構造

```json
{
  "version": 0.6,
  "generator": "Overpass API ...",
  "osm3s": { "timestamp_osm_base": "...", "copyright": "..." },
  "elements": [ ... ]
}
```

### node
```json
{ "type": "node", "id": 123, "lat": 35.68, "lon": 139.76, "tags": { ... } }
```

### way（out body）
```json
{ "type": "way", "id": 456, "nodes": [123, 789, ...], "tags": { "highway": "primary" } }
```
※`nodes`はIDリストのみ。座標は `>` 展開か `out geom` で取得。

### way（out geom）
```json
{
  "type": "way", "id": 456,
  "geometry": [{"lat": 35.6, "lon": 139.7}, ...],
  "tags": { "highway": "primary" }
}
```

## レート制限

### 公式インスタンス
| 項目 | 値 |
|------|-----|
| 推奨最大リクエスト | 1日 ~10,000 |
| 推奨最大ダウンロード | 1日 1GB以下 |
| スロット数/IP | 2 |
| デフォルトタイムアウト | 180秒 |
| デフォルト最大メモリ | 512 MiB |

### スロットベースの制限
- IPアドレスごとに2スロット割り当て
- リクエスト中＋クールダウン中はスロット占有
- スロット空きなし → 最大15秒キュー待ち → HTTP 429

## エラーハンドリング

| HTTP | 原因 | 対処 |
|------|------|------|
| 400 | クエリ構文エラー | クエリを修正 |
| 429 | レート制限超過 | 指数バックオフ+ジッターでリトライ |
| 504 | タイムアウト/メモリ超過 | bboxを縮小、フィルタを厳しく |

ベストプラクティス:
- bboxを小さく保つ（数km四方）
- 取得データをキャッシュ（localStorage/IndexedDB）
- 複数ミラーへのフォールバック
- 並列リクエスト数を2以下に抑える
