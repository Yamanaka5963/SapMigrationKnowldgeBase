# ABAPコード収集・AI解析ツール構想

## コンセプト
SAP ECC→S/4HANA移行支援のため、カスタムABAPコードをAIエージェントで自動解析するツール。

---

## コード収集手段（3択）

| 手段 | 特徴 |
|---|---|
| **RFC**（RPY_PROGRAM_READ等） | 外部からプログラム的に取得 |
| **ADT REST API** | `GET /sap/bc/adt/programs/.../source/main` |
| **abapGit** | GitリポジトリへエクスポートしてCI/CD親和性高 |

abapGit経由が**Claude Codeとの親和性が最も高い**。

---

## 処理フロー

```
[SAP DEV環境]
    ↓ abapGit / RFC / ADT REST
[Gitリポジトリ / Azure Storage]
    ↓
[Claude Code エージェント]
    ├── Clean Core非準拠箇所の検出
    ├── 未リリースAPI使用箇所のリスト化
    ├── S/4HANA移行影響分析
    └── リファクタリング提案生成
    ↓
[RAGナレッジベースと統合]
    └── 「このコードは移行でどう影響を受けるか」
        を自然言語で回答
```

---

## 重要制約

- **対象はZ/Yカスタムコードのみ**
- SAP標準コードの外部持ち出し・学習データ化はライセンス上グレー
- 実施環境はDEV環境が原則（本番不可）
- S_DEVELOP Display権限が必要

---

## ビジネス価値

KEYSTONEのSAP案件における**差別化サービス**として、移行影響調査の工数を大幅削減できる可能性あり。既存のRAGナレッジベース構想とも自然に統合できる。
