# 30 - データベース概要：ECC vs S/4HANA

## 核心的な違い

SAP ECCはデータベース非依存型です — Oracle、Microsoft SQL Server、IBM DB2、SAP ASE（Sybase）、SAP MaxDBのいずれでも動作します。どのDBを使用していてもアプリケーション層は同じです。

SAP S/4HANAは**SAP HANA専用**です — インメモリ型のカラム型データベースです。これは単なるDB変更ではなく、S/4HANAの簡素化されたデータモデルはHANAのアーキテクチャ向けに設計されています。

| | SAP ECC | SAP S/4HANA |
|---|---|---|
| 対応DB | Oracle、MSSQL、DB2、SAP ASE、MaxDB、HANA | SAP HANA のみ |
| テーブル種別 | 透過、プール、クラスター | 透過テーブルのみ |
| データモデル | パフォーマンス用の冗長な集計/インデックステーブルあり | 簡素化 — HANAがインメモリで集計を処理 |
| 在庫数量 | 複数のスナップショットテーブルに保存 | MATDOCからオンザフライで計算 |
| 財務転記 | FI/CO/AA/MLテーブルに分散 | 単一テーブル：ACDOCA |
| ビジネスパートナー | 得意先（KNA1）と仕入先（LFA1）が別々 | 統合ビジネスパートナー（BUT000） |

## データモデルが変わった理由

ECCでは、パフォーマンスはサマリーテーブル（合計、インデックス、スナップショット）へのデータ事前集計によって実現されていました。HANAのカラム型インメモリエンジンはそれらの冗長テーブルを不要にします — むしろ負債となります（二重書き込みのオーバーヘッド、整合性リスク）。

S/4HANAの簡素化されたデータモデル：
- 冗長な集計/インデックステーブルを廃止
- モジュール別の明細テーブルを単一構造に統合
- プール/クラスターテーブル（HANA非対応）を透過テーブルに変換
- ビジネスマスターデータを統合

## 移行エンジニアへの影響

移行アプローチ（ブラウンフィールド/グリーンフィールド/シェル）に関わらず、すべての移行プロジェクトで必ず対応が必要な項目：

1. **ACDOCA** — 全財務転記の統合 → [31](31jp-acdoca-universal-journal.md)を参照
2. **MATDOC** — 全在庫移動の統合 → [32](32jp-matdoc-inventory.md)を参照
3. **プール/クラスター → 透過テーブル** — SUM実行時のテーブル構造変換 → [33](33jp-pool-cluster-conversion.md)を参照
4. **CVI / ビジネスパートナー** — 必須の得意先/仕入先統合 → [34](34jp-cvi-business-partner.md)を参照
5. **HANAシステムビュー** — HANAレイヤーへの直接クエリ → [35](35jp-hana-system-views-reference.md)を参照

置き換えられたECCテーブルに対して読み書きを行うカスタムコードはすべて改修が必要です。SAPが提供する互換性ビューとAPIが移行期間の経路として用意されています。

## 参考資料

- SAP S/4HANA 変換ガイド 2025（公式PDF、`help.sap.com/docs/SAP_S4HANA_ON-PREMISE` 経由）: https://help.sap.com/doc/8cec006e962a46049506bc10ed64f557/2025/en-US/CONV_PE2025.pdf
- 簡素化一覧リスト S/4HANA 2023（公式PDF）: https://help.sap.com/doc/c34b5ef72430484cb4d8895d5edd12af/2023/en-US/SIMPL_OP2023.pdf
- 簡素化一覧リスト S/4HANA 2022（公式PDF）: https://help.sap.com/doc/59bf7d0f62d24af78f87c560da8f18ce/2022/en-US/SIMPL_OP2022.pdf
