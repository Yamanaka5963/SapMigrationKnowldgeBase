# 17 - データ移行（Migration Cockpit）

## 概要

SAP S/4HANA Migration Cockpitは、ビジネスデータ（マスターデータとトランザクションデータ）をS/4HANAへ移行するためのSAP標準ツールです。定義済みの移行オブジェクトを使用し、バリデーション・マッピング・シミュレーション機能を内蔵しています。追加ライセンスは不要 — S/4HANAに含まれています。

グリーンフィールドおよびシェルコンバージョンプロジェクトで主に使用します。ブラウンフィールドでは既存データはSUM/DMOで引き継がれますが、カットオーバー未処理明細の初期ロードなどにCockpitを使用することもあります。

## 移行アプローチ

| アプローチ | 説明 | 適したケース |
|---|---|---|
| **ファイルベース（ローカルテンプレート）** | XML/CSVテンプレートをダウンロードして手動入力し、アップロード | 小〜中規模のデータ量；シンプルな設定 |
| **ステージングテーブル** | ETLツールまたはSAP Data ServicesでHANA DBの中間テーブルに投入 | 大規模データ；複雑なデータ変換 |
| **ダイレクトトランスファー（RFC）** | ソースSAPシステムとターゲットS/4HANA間のRFC接続；データモデルを自動変換 | SAP間移行；対応システムへの最速手段 |

## ワークフロー（ファイルベースの場合）

1. **移行オブジェクトの選択** — 例：得意先マスター、未処理購買発注、G/L勘定残高
2. **テンプレートのダウンロード** — 定義済みフィールド構造のXMLまたはCSV
3. **テンプレートへの入力** — レガシーデータ抽出からマッピングして入力
4. **テンプレートのアップロード** — Migration Cockpitへインポート
5. **シミュレーション** — 転記なしの検証実行；エラーログを確認
6. **エラーの修正** — ソーステンプレートのデータを修正して再アップロード
7. **移行の実行** — 本番への最終データロード

## 主な移行オブジェクト（例）

| カテゴリ | オブジェクト |
|---|---|
| **財務** | G/L勘定、コストセンター、利益センター、未処理明細（AR/AP）、固定資産マスター |
| **調達** | 仕入先マスター、購買情報レコード、未処理購買発注 |
| **販売** | 得意先マスター、販売オーダー（未処理）、価格条件 |
| **ロジスティクス** | 品目マスター、保管場所、バッチマスター、在庫残高 |
| **HR** | 組織単位、役職（S/4HANA HCMがスコープの場合） |

## ビジネスパートナー（CVI）— 必須対応

S/4HANAでは、得意先と仕入先を**ビジネスパートナー（BP）**へ変換することが必須です。これはすべての移行プロジェクトで対応が必要です — ブラウンフィールドでも同様です。

- CVI（顧客仕入先統合）同期化を実行
- BP重複レコードがないことを確認
- 本番稼働前にReadiness CheckでのCVIエラーをすべて対処

## ベストプラクティス

- 最終移行実行前に、エラー件数がゼロになるまでシミュレーションを繰り返す
- データロードの順序を設計する：マスターデータを先に、次に未処理トランザクションデータ
- シーケンスの順序が重要：参照先オブジェクトを参照元より先に作成すること（例：G/L勘定よりコストセンターを先に）
- カットオーバーロード用の最終データ抽出を行ったら、レガシーシステムを読み取り専用モードにすること

## 参考資料

- SAP Help Portal — Migration Cockpit（公式）: https://help.sap.com/docs/SAP_S4HANA_CLOUD/d5699934e7004d048c4801b552f3b013/121b34742a904d10bca907bbf2fd5617.html
- SAP Community トピックページ — Migration Cockpit: https://pages.community.sap.com/topics/s4hana-migration-cockpit
- SAP Learning — Migration Cockpitの紹介（公式）: https://learning.sap.com/courses/implementing-sap-s-4hana-cloud-public-edition/introducing-the-sap-s-4hana-migration-cockpit
- SAP Community — Migration Cockpitステップバイステップガイド（メンバー投稿）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-members/step-by-step-guide-to-ecc-to-s-4hana-migration-with-migration-cockpit/ba-p/13983763
