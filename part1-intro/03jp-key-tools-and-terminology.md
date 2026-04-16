# 03 - 主要ツールと用語集

| 用語 / ツール | 説明 | 参考資料 |
|---|---|---|
| **SUM**（Software Update Manager） | ブラウンフィールドのインプレース変換における中核ツール。アップグレードスタックの実行を管理する。 | [SAP Community（SAP公式）](https://community.sap.com/t5/technology-blog-posts-by-sap/database-migration-option-dmo-of-sum-introduction/ba-p/13262160) |
| **DMO**（Database Migration Option） | SUMのアドオン。システム変換と同時にDBをSAP HANAへ移行する。 | 上記と同じ |
| **Migration Cockpit**（移行コックピット） | 定義済みの移行オブジェクトを使用して、マスターデータおよびトランザクションデータをS/4HANAへ移行するSAPツール。 | [SAP Help Portal](https://help.sap.com/docs/SAP_S4HANA_CLOUD/d5699934e7004d048c4801b552f3b013/121b34742a904d10bca907bbf2fd5617.html) · [SAP Community トピック](https://pages.community.sap.com/topics/s4hana-migration-cockpit) |
| **Readiness Check**（レディネスチェック） | クライアントのECCシステムのS/4HANA適合性を自動分析するツール。アドオン、カスタムコード、簡素化項目を確認する。 | [SAP Help Portal](https://help.sap.com/docs/cloud-alm/applicationhelp/sap-readiness-check) |
| **RISE with SAP** | SAPのマネージドクラウド移行オファリング。S/4HANAプライベートクラウド・ツール・方法論をバンドルして提供。 | [SAP公式](https://www.sap.com/products/erp/rise.html) |
| **RISE Transition Workbench** | RISE移行のための統合ツール。5つの技術的移行アプローチをステップバイステップでガイド。移行サーバーにプリインストール済み。 | [SAP Support](https://support.sap.com/en/tools/software-logistics-tools/rise-with-sap-system-transition-workbench.html) |
| **SPDD / SPAU** | システム変換後にデータディクショナリオブジェクト（SPDD）とリポジトリオブジェクト（SPAU）を再調整するトランザクション。アップグレード後の必須作業。 | 標準SAPトランザクション |
| **SAP Activate** | SAPの現行プロジェクト方法論（アジャイルベース、ASAPを代替）。6フェーズ構成。 | [04 - SAP Activate 方法論](04jp-sap-activate-methodology.md)を参照 |
| **Simplification Items**（簡素化項目） | S/4HANAで変更または削除されたECCの機能・テーブル一覧。アセスメント中に必ずレビューが必要。 | Readiness Checkの出力結果に含まれる |
