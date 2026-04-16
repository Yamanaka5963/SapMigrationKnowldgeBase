# 13 - ブラウンフィールド実行ガイド（システムコンバージョン）

## 概要

ブラウンフィールドは、SUM（Software Update Manager）とDMO（Database Migration Option）を使用して、既存のECCシステムをS/4HANAへインプレース変換します。SUM/DMOは以下の3つの処理を1回の実行で組み合わせます：
1. SAP HANAへのデータベース移行
2. ターゲットS/4HANAリリースへのシステムソフトウェアアップグレード
3. データモデル変換（S/4HANA簡素化データモデルへの適合）

## 前提条件

### ソースシステム要件
- SAP ERP 6.0以上（Unicodeのみ — 非Unicodeシステムは事前にUnicode変換が必要）
- SAP HANA上のABAP+Javaデュアルスタック不可
- 必須SAPノートの適用（以下参照）

### ターゲットシステム要件
- SAP HANA 2.0 SP04 revision 46以上
- ソースがすでにHANA上の場合：SAP HANA 2.0 SP05 revision 52以上
- SUM 2.0 SP20以上を推奨

### 適用必須SAPノート
| ノート番号 | 目的 |
|---|---|
| **3126522** | SUM 2.0を使用したS/4HANAへの変換 |
| **2399707** | SAP Readiness Check / 簡素化項目チェック |
| **2975231** | 変換中のデータ整合性 |
| **2214409** | サポート対象アドオン一覧 |
| **2388483** | 技術的ハウスキーピング作業 |

> ノートの完全なリストはMaintenance Plannerが生成します。stack.XMLの出力に必ず従ってください。

## 実行フェーズ

### フェーズ1 — 準備
- [ ] Maintenance Plannerを実行 → `stack.XML`の生成とソフトウェアパッケージのダウンロード
- [ ] 簡素化項目チェック（SI-Checks）の実行とブロッカーの解消
- [ ] SUM前提条件チェック拡張版（SUM Prerequisite Check Extended）の実行
- [ ] 必須SAPノートの適用
- [ ] 技術的ハウスキーピングの実施（ノート2388483に従ったアーカイブ、テーブルクリーンアップ）
- [ ] SPDD/SPAUトランスポート要求の事前準備
- [ ] カスタムコード改修計画の整備 → [16 - カスタムコード改修](16jp-custom-code-remediation.md)を参照

### フェーズ2 — アップタイムフェーズ（シャドウリポジトリ）
このフェーズ中、業務ユーザーはシステムを通常通り使用できます。

- SUMがターゲットHANA DB上にシャドウインスタンスとシャドウリポジトリを作成
- ソースOS/DB上で設定・リポジトリテーブルを準備
- ダウンタイムを最小化するため、大容量テーブルをアップタイム中に移行
- データレプリケーション（トリガー）でユーザーの継続的な変更をキャプチャ
- このフェーズ中の大量データ変更やアーカイブは避けること（レプリケーションのボトルネックになる）

> 推奨：アップタイムフェーズではR3Loadプロセスを10並列で使用

### フェーズ3 — ダウンタイムフェーズ（実際の変換）
業務ダウンタイム開始。主なSUMステップ：

| SUMステップ | 説明 |
|---|---|
| **EU_CLONE_MIG_DT_RUN** | 残存データのダウンタイム移行 |
| **PARCONV_UPG** | アプリケーションテーブル構造を新リリース向けに変換 |
| **XPRAS_AIMMRG** | ポストインポートプログラムとAfter Import Methods（データモデル変換）を実行 |
| **KX_SWITCH** | 新カーネルへの切り替え |
| **EU_SWITCH** | DB接続をターゲットリポジトリに切り替え |
| **DDIC_UPG（SPDD）** | データディクショナリ修正調整 — ダウンタイムの早い段階で実施 |
| **SPAU** | リポジトリオブジェクト修正調整 — SUMプロセスの終盤で実施 |

> 標準ダウンタイムの目安：1週末。合計2週末を計画（1回目は初期転送リハーサル、2回目が本番ダウンタイム）。

### フェーズ4 — 変換後作業
- [ ] SPDDの調整完了（データディクショナリオブジェクト）
- [ ] SPAUの調整完了（カスタム修正）— エンハンスメント修正にはSPAU_ENHを使用
- [ ] ATC（ABAP Test Cockpit）のS/4HANAチェックを実行し、残存するカスタムコード問題を修正
- [ ] スモークテストと機能検証
- [ ] パフォーマンスベースラインテスト
- [ ] クライアントとのGo/No-Go判定

## ダウンタイム最適化コンバージョン（オプション）

ほとんどのデータ変換処理をアップタイムフェーズに移動し、業務ダウンタイムを大幅に短縮します。必要条件：
- チームメンバー最低1名がSAPトレーニング **ADM329** と知識評価 **ADM329k** を修了していること
- ソースシステムにSUM Toolboxが導入済みであること（ノート3092738）
- 複雑な計画が必要 — 追加の準備期間を考慮すること

## 参考資料

- SAP Support — Software Update Manager（公式）: https://support.sap.com/en/tools/software-logistics-tools/software-update-manager.html
- SAP Support — DMO概要（公式）: https://support.sap.com/en/tools/software-logistics-tools/software-update-manager/database-migration-option-dmo.html
- SAP Help — SUM を使用したS/4HANAへの変換（公式）: https://help.sap.com/docs/SLTOOLSET/e239e55723bc4d8ca923bba137df205b/7ef3c766f02f45c7b218fed5b2b38157.html
- SAP Learning — SUM処理（公式）: https://learning.sap.com/courses/sap-s-4hana-conversion-and-sap-system-upgrade/processing-the-sum_c0b49fee-5d0d-4cd6-abd8-53098a9c3540
- SAP Learning — DMOの概要（公式）: https://learning.sap.com/courses/sap-s-4hana-conversion-and-sap-system-upgrade/exploring-the-database-migration-option-dmo-
- SAP Community — SUM DMO入門（SAP公式）: https://community.sap.com/t5/technology-blog-posts-by-sap/database-migration-option-dmo-of-sum-introduction/ba-p/13262160
- S/4HANA 変換ガイド 2025（公式PDF）: https://help.sap.com/doc/8cec006e962a46049506bc10ed64f557/2025/en-US/CONV_PE2025.pdf
