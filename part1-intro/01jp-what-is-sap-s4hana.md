# 01 - SAP・S/4HANAとは

**SAP ECC**（ERP Central Component）は、多くのクライアントが現在稼働させているレガシーのオンプレミスERPです。財務、HR、調達、製造などを統合管理します。

**SAP S/4HANA**は、その後継製品です。SAP HANAインメモリデータベース上に再構築され、簡素化されたデータモデルとFiori UIを採用しています。

## ECC と S/4HANA の比較

| | SAP ECC | SAP S/4HANA |
|---|---|---|
| データベース | 任意（Oracle、MSSQL等） | SAP HANA のみ |
| データモデル | 複雑・冗長なテーブル構成 | 簡素化（例：FI-CO統合、MATDOCがMSEG+MKPFを代替） |
| UI | SAP GUI | SAP Fiori（Webベース） |
| 処理方式 | バッチ処理中心 | リアルタイム・インメモリ処理 |
| AI/ML | 限定的 | 組み込み済み |
| 展開形態 | オンプレミスのみ | オンプレミス・プライベートクラウド・パブリッククラウド |

## 展開オプション

| オプション | 概要 | SAP の方針 |
|---|---|---|
| **オンプレミス** | 自社管理・フルカスタマイズ可能 | 2040年まで公式サポート継続。ただし**新機能の追加なし**。戦略的重点ではない。 |
| **プライベートクラウド**（RISE with SAP） | SAP管理・ハイパースケーラー（AWS/Azure/GCP）上の専用インスタンス | **SAP が推奨する主要な移行パス。** 継続的なイノベーション・AI・ESG機能を提供。 |
| **パブリッククラウド** | 標準化されたマルチテナント構成・カスタマイズ制限あり | 新規顧客や標準化を重視する企業向け。 |

> SAP の公式方針は**クラウドファースト**です。オンプレミスの S/4HANA は引き続きサポートされますが、イノベーションは縮小しています。新機能やAI機能はクラウドエディションにのみ提供されます。多くのクライアント移行において、RISE with SAP（プライベートクラウド）が事実上の推奨アプローチです。

> **重要期限：SAP ECC のメインストリームメンテナンスは2027年12月31日に終了します。**

## 参考資料

- SAP S/4HANA 製品ページ（公式）: https://www.sap.com/products/erp/s4hana.html
- SAP クラウドファースト戦略に関する声明（SAP News、2024年）: https://news.sap.com/2024/01/rise-with-sap-migration-and-modernization-cloud-first-business-strategy/
- RISE with SAP ECC 移行（公式）: https://www.sap.com/products/erp/rise/sap-ecc-migration.html
- S/4HANA オンプレミスの2040年までのイノベーションコミットメント（SAP Support）: https://support.sap.com/en/release-upgrade-maintenance/maintenance-information/maintenance-strategy/s4hana-business-suite7.html
- SAP Community — クラウド vs オンプレミス展開オプション（SAP公式ブログ）: https://community.sap.com/t5/enterprise-resource-planning-blog-posts-by-sap/sap-s-4hana-cloud-and-on-premise-deployment-options/ba-p/13410008
- SAP S/4HANA 変換ガイド 2025（公式PDF、`help.sap.com/docs/SAP_S4HANA_ON-PREMISE` 経由）: https://help.sap.com/doc/8cec006e962a46049506bc10ed64f557/2025/en-US/CONV_PE2025.pdf
