---
tags:
  - permanent
  - ガイド
  - zotero
---

# Zotero連携ガイド

> [!NOTE] Permanent Note
> このファイルはpermanentファイル（永久ノート）として、いつでも編集・更新可能です。

## 概要

ZoteroとObsidianの`02_literature.md`テンプレートを連携させることで、文献管理と知識管理を統合できます。

## 連携方法

### 方法1：Obsidian Citationsプラグインを使用（推奨）

Obsidian Citationsプラグインを使用すると、Zoteroの文献をObsidian内で直接引用・参照できます。

#### 必要な準備

1. **Zotero Better BibTeXプラグインのインストール**
   - Zoteroを開く
   - ツール → アドオン → アドオンの取得
   - "Better BibTeX"を検索してインストール
   - Zoteroを再起動

2. **BibTeXファイルの生成**
   - Zoteroで、エクスポートしたい文献を選択（または全て選択）
   - 右クリック → "Better BibTeX Export" → "Export Collection"
   - 形式：Better CSL JSON または Better BibTeX
   - 保存先：Obsidian Vault内の任意の場所（例：`bibliography.json` または `bibliography.bib`）
   - **重要**：自動エクスポートを有効にする（文献を追加・更新するたびに自動的にエクスポート）

3. **Obsidian Citationsプラグインのインストール**
   - Obsidianの設定 → コミュニティプラグイン
   - "Citations"を検索してインストール
   - プラグインを有効化

4. **Citationsプラグインの設定**
   - 設定 → Citations
   - Citation database path：生成したBibTeXファイルのパスを指定
   - Citation format：好みの引用形式を選択
   - Literature note folder：`01_data`を指定（または空欄でデフォルト）

#### 使用方法

1. **文献の引用**
   - ノート内で`@`を入力すると、Zoteroの文献が自動補完されます
   - 例：`@smith2020`と入力すると、文献が引用されます

2. **文献ノートの作成**
   - Citationsプラグインのコマンドパレットから「Insert literature note」を実行
   - または、`@citekey`を右クリック → 「Open literature note」
   - 自動的に`02_literature.md`テンプレートを使用して文献ノートが作成されます

3. **文献ノートのカスタマイズ**
   - `02_literature.md`テンプレートにZoteroのcitekeyを追加する（後述）

### 方法2：手動でcitekeyを追加

Citationsプラグインを使用しない場合でも、Zoteroのcitekeyを手動で文献ノートに追加できます。

#### 手順

1. **Zoteroでcitekeyを確認**
   - Zoteroで文献を選択
   - 右クリック → "Copy Better BibTeX Citation Key"
   - または、Better BibTeXの設定でcitekeyの形式を確認

2. **文献ノートにcitekeyを追加**
   - `02_literature.md`テンプレートに`zotero-citekey`プロパティを追加（後述）

## 02_literature.mdテンプレートの更新

Zoteroと連携するために、`00_templates/02_literature.md`テンプレートを以下のように更新することを推奨します：

```yaml
---
created: 2026-01-15 23:59:26
tags:
  - 文献
  - PDF
title:
source:
author:
published:
description:
zotero-citekey:  # Zoteroのcitekey（例：smith2020）
zotero-link:     # Zoteroへのリンク（オプション）
---
```

### citekeyの取得方法

- **Better BibTeX使用時**：
  - Zoteroで文献を選択 → 右クリック → "Copy Better BibTeX Citation Key"
  - 例：`smith2020`, `doe2021quantum`

- **標準Zotero使用時**：
  - 文献の「キー」フィールドを確認
  - または、Zoteroの設定でcitekeyの形式を設定

### Zoteroへのリンク

Zoteroへの直接リンクを追加する場合：

1. **Zotero URIスキームを使用**
   ```
   zotero-link: zotero://select/items/[itemKey]
   ```
   - `itemKey`は、Zoteroで文献を選択 → 右クリック → "Copy Link"で取得

2. **Zotero Web Libraryを使用**
   ```
   zotero-link: https://www.zotero.org/users/[userID]/items/[itemKey]
   ```

## ワークフロー例

### 新規文献ノート作成時

1. **Zoteroで文献を追加**
   - ブラウザ拡張機能で文献を追加
   - または、手動で文献情報を入力

2. **Obsidianで文献ノートを作成**
   - `Ctrl + J`で`02_literature.md`テンプレートを使用
   - または、Citationsプラグインで自動生成

3. **citekeyを追加**
   - Zoteroからcitekeyをコピー
   - 文献ノートの`zotero-citekey`プロパティに貼り付け

4. **文献情報を記入**
   - `title`, `author`, `published`, `source`などを記入
   - PDFファイルがある場合は、`01_data/YYYY/MM/DD/`に配置してリンク

5. **関連ノートとリンク**
   - 関連する過去のノートへのリンクを作成
   - 適切なタグを設定

### 既存文献ノートにcitekeyを追加

1. Zoteroで対応する文献を検索
2. citekeyをコピー
3. 文献ノートのプロパティに`zotero-citekey`を追加

## 検索とフィルタリング

### citekeyで検索

- Floating Search（Alt + F）で`zotero-citekey:smith2020`と検索
- DataViewでcitekeyでフィルタリング

### DataViewクエリ例

```dataview
TABLE zotero-citekey, author, published
FROM "01_data"
WHERE zotero-citekey
SORT published DESC
```

## よくある質問

### Q: Citationsプラグインは必須ですか？

A: 必須ではありません。手動でcitekeyを追加する方法でも連携できます。ただし、Citationsプラグインを使用すると、引用の挿入や文献ノートの自動生成が便利です。

### Q: Better BibTeXは必須ですか？

A: 必須ではありませんが、**強く推奨**します。Better BibTeXを使用すると、citekeyの管理や自動エクスポートが簡単になります。

### Q: 文献ノートはいつ作成すべきですか？

A: Zoteroで文献を追加した時点で作成することを推奨します。ただし、このシステムの編集制限ルールに従い、作成日（睡眠前）までに完成させてください。

### Q: PDFファイルはどこに配置すべきですか？

A: ZoteroのPDFファイルを`01_data/YYYY/MM/DD/`配下にコピーして配置することを推奨します。Zoteroの元ファイルはそのままにしておき、Obsidian内でも参照できるようにします。

### Q: 文献ノートとZoteroの同期は自動ですか？

A: いいえ、自動同期はありません。Zoteroで文献を追加・更新したら、手動でObsidianの文献ノートも更新する必要があります。ただし、Better BibTeXの自動エクスポート機能を使用すると、BibTeXファイルは自動更新されます。

## 参考リソース

- [Obsidian Citations プラグイン](https://github.com/hans/obsidian-citation-plugin)
- [Zotero Better BibTeX](https://retorque.re/zotero-better-bibtex/)
- [[02_literature]] - 文献ノートテンプレート
- [[タグ基準]] - タグ付けの基準
- [[リンク基準]] - リンク付けの基準
