# SAP技術スタック全体像

## レイヤー構造

```
[SAPGUI]                    ← 管理者・開発者向け操作ツール（現役・並列存在）
[Fiori / UI5]               ← エンドユーザー向けフロントエンド
[BTP]
  ├── CAP（Node.js/Java）   ┐
  └── ABAP on BTP（RAP）    ┘← 排他の実装手段選択
[OData API]                 ← 両者共通の出力規約（GraphQL的なクエリ記法）
[S/4HANA]
  └── ABAP                  ← ERPコアのバックエンド言語
[HANA DB]                   ← データ層
```

---

## 排他関係の整理

### ① CAPとRAP（最重要の排他）
OData APIを公開する**実装手段**としての選択。

| | CAP | RAP |
|---|---|---|
| 言語 | Node.js / Java | ABAP |
| 向いている場面 | 新規開発・他システム統合 | S/4HANA連携・既存ABAP資産活用 |
| 実行環境 | BTP | ABAP on BTP |
| 出力 | OData / REST | OData |

→ **どちらを選んでもODataを出力する。OData自体は排他にならない。**

### ② FioriとSAPGUI（並列・排他ではない）

| | Fiori/UI5 | SAPGUI |
|---|---|---|
| 用途 | カスタム開発・新規UI | 標準トランザクション操作 |
| 現状 | SAPの推奨方向 | 今も現役・置き換え対象外 |

→ **置き換え関係ではなく並列して存在。**

### ③ Fiori ElementsとFreestyle UI5（UI実装の排他）

| | Fiori Elements | Freestyle UI5 |
|---|---|---|
| コード量 | 少（設定ベース自動生成） | 多（フルカスタム） |
| 柔軟性 | 低 | 高 |
| SAPの推奨 | ✅ | 必要な場合のみ |

---

## ODataについて

- GraphQLと同カテゴリの**クエリ記法・データ形式の標準仕様**
- REST上で動作しURLパラメータでクエリを表現
- CAPでもRAPでも出力形式として使われる共通規約

---

## デプロイモデルとABAPカスタマイズ自由度

| モデル | ABAPカスタマイズ | Clean Core |
|---|---|---|
| On-Premise | ✅ フル（標準コード変更も可・自己責任） | 任意 |
| Private Cloud | ⚠️ 可だが非推奨 | 強く推奨 |
| Public Cloud | ❌ 不可 | 強制 |
