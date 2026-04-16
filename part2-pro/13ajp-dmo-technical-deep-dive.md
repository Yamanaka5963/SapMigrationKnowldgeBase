# 13a - DMO テクニカルディープダイブ

> このドキュメントは [13 - ブラウンフィールド実行ガイド](13jp-brownfield-execution.md) の補足資料です。内部で何が起きているかを理解する必要があるエンジニア向けに、DMO（Database Migration Option）の技術的な仕組みを詳しく解説します。

## DMOとは

DMOは以下の3つの処理を**1回のSUM実行**で組み合わせます：
1. **ソフトウェアアップグレード** — ECC からターゲットS/4HANAリリースへ
2. **データベース移行** — ソースDB（Oracle、MSSQL、DB2等）から SAP HANA へ
3. **データモデル変換** — S/4HANA 簡素化データモデルへのデータ再構成

中核エンジンは **R3load** — SAP の独自データ移行ユーティリティです。

## R3load — 移行エンジン

R3load は**並列ペア**（エクスポートプロセス + インポートプロセス）で動作します：

- エクスポートR3load が**ソースDB**からデータを読み取り
- インポートR3load が**ターゲット SAP HANA DB**にデータを書き込み
- データは**メモリパイプ**経由で転送 — ホストのメインメモリを直接使用。ダンプファイルなし、圧縮なし
- 作業は**バケット**（テーブルのワークパッケージ）に分割。各R3loadペアが1バケットを処理し、完了後に終了。SUMが次のペアを起動。

```
ソースDB → [R3loadエクスポート] ——パイプ——> [R3loadインポート] → ターゲット HANA DB
```

> パイプモードは従来のファイルベース移行（ディスクへのエクスポート → 圧縮 → 転送 → 解凍 → インポート）と比較して大幅に高速です。中間ファイルは一切作成されません。

**技術的詳細：** MIGRATE_UTコマンドファイルでは、エクスポートR3loadがソケットオプション付きで起動し、ポートでリッスンします。インポートR3loadがそのポートに接続し、パイプ経由のデータストリームを直接受信します。

## シャドウリポジトリ & シャドウインスタンス

ダウンタイムが始まる前に、SUM はターゲットシステム上に2つのコンポーネントを作成します：

| コンポーネント | 説明 |
|---|---|
| **シャドウインスタンス** | SUMがプライマリアプリケーションサーバー上に作成する追加のABAPインスタンス。**ターゲットリリースカーネル**（ソースカーネルではない）で動作する |
| **シャドウリポジトリ** | アップタイム中に**ターゲット HANA DB**上に構築される完全なデータベースリポジトリ — 新しいリリースにおけるABAPディクショナリとリポジトリオブジェクト |

シャドウリポジトリは、ユーザーが本番システムで作業を続けている間に、R3loadパイプモードを使用して移行されます。**SUM 2.0 SP08以降**、シャドウリポジトリはターゲットHANA DB上に直接作成されます（以前はソースDB上にステージングされていました）。

**シャドウシステムの目的：**
- 本番環境に触れることなく、ターゲットリリースでシステム変換を準備・テスト
- ターゲットS/4HANAリリースに必要な新しいオブジェクトタイプを作成
- 本番稼働と並行してアップタイム中のデータ移行を実施

## アップタイムフェーズ — 詳細

システムは**稼働中** — ユーザーは通常通り作業を続けます。

### 処理内容：

1. SUMが選択した大容量アプリケーションテーブルに**データベーストリガー**を設置
2. R3loadペアがパイプ経由でそれらのテーブルの初期データをターゲットHANA DBへ移行
3. 初期移行中にユーザーが行ったすべてのINSERT、UPDATE、DELETEをトリガーが記録
4. **CRR（Change Recording & Replication）**フレームワークがシャドウインスタンス上で専用レプリケーターを起動 — トリガーログを読み取り、デルタ差分をターゲットDBへ継続的にレプリケート
5. **Writer**（ターゲットカーネルを使用する一時的なTMPインスタンス）がデルタを受け取りターゲットHANA DBに書き込み

```
本番システムでのユーザー操作
        ↓
DBトリガーが変更をキャプチャ
        ↓
CRRレプリケーターがトリガーログを読み取り
        ↓
Writer（TMP、ターゲットカーネル）→ ターゲット HANA DB
```

**キーフェーズ名：** `EU_CLONE_MIG_OPTDMO_INI_RUN`（ダウンタイム最適化バリアント）

**アップタイムフェーズ中の注意事項：**
- 大規模なバッチジョブや大量データ変更は避けること — トリガーログが急増しレプリケーションが遅延する
- 移行中のテーブルへのアーカイブ処理は避けること — アーカイブ済みデータはデルタレプリケーションで処理できない
- SLT（SAP Landscape Transformation）レプリケーション：既存のSLTトリガーがDMOトリガーに置き換わる場合がある。変換後にSLTで初期ロードが必要になる可能性がある

## ダウンタイムフェーズ — 詳細

システムは**オフライン** — ユーザーはシステムにアクセスできません。

| ステップ | SUMフェーズ | 処理内容 |
|---|---|---|
| 1 | — | SAPシステムのシャットダウン；新たな変更は不可 |
| 2 | `MIG2NDDB_SWITCH` | **DB接続をソースDBからターゲットHANAに切り替え** |
| 3 | — | ターゲットカーネルディレクトリへの切り替え（カーネルスイッチ） |
| 4 | — | アップタイム中に移行できなかった残存テーブルの最終R3load移行 |
| 5 | — | デルタリプレイ — アップタイム中にトリガーで記録されたすべての変更をターゲットHANAに適用 |
| 6 | `PARCONV_UPG` | アプリケーションテーブル構造を新しいS/4HANAリリース形式に変換 |
| 7 | `XPRAS_AIMMRG` | ポストインポートプログラムとAfter Import Methods（AIM）を実行 — データをS/4HANAデータモデルに変換 |
| 8 | — | システムがターゲットHANA DB + ターゲットカーネルで起動 |

> ソースデータベースはプロセス全体を通じて**完全に変更されません**。フォールバックとして機能します — 問題が発生した場合、SUM RESETでソースDBへの接続を復元できます。

## DMOのバリアント

| バリアント | 説明 | 用途 |
|---|---|---|
| **標準DMO** | インプレース移行；アプリケーションサーバーはそのままで、同一または近接ホスト上のDBをHANAに移行 | 標準的なオンプレミスからオンプレミスへのブラウンフィールド |
| **DMO with System Move** | 変換実行中に別のサーバーやデータセンターへ移行 | 移行と同時にインフラ更新 |
| **DMOVE2S4** | クラウド専用：ECCをRISE with SAP S/4HANA Private Cloudへ1回のSUM実行で移行 | RISEプライベートクラウドへの移行 |
| **Homogeneous DMO** | HANA→HANA移行（ソースがすでにHANA上） | すでにHANA上のシステムをS/4HANAにアップグレード |
| **ダウンタイム最適化DMO（doDMO）** | CRR経由でアップタイム中に大容量テーブルの移行とデータ変換を実施；ダウンタイムウィンドウを最小化 | ダウンタイムがハード制約となる大規模DB |

## DMOVE2S4 — クラウド移行バリアント

DMOVE2S4は、ECCオンプレミスを**RISE with SAP S/4HANA Private Cloud**へ1回のSUM実行で移行するアプローチです。

**標準DMOとの主な違い：**

| | 標準DMO | DMOVE2S4 |
|---|---|---|
| ターゲット | オンプレミスHANA | S/4HANA Private Cloud（ハイパースケーラー） |
| R3loadモード | ファイルモードまたはパイプモード | **パイプモード必須** |
| ネットワーク遅延要件 | なし | ソース-ターゲット間 **< 20ms** |
| ネットワーク帯域要件 | なし | **> 400 Mbit/s** |
| HANA→HANAサポート | Homogeneous DMO経由 | 非対応 — Homogeneous DMOを使用 |

**技術的な違い：**
- SUM は最初はソースオンプレミスDBに接続しながら、**ターゲットクラウド環境**にインストールされたアプリケーション上で動作する
- R3loadパイプモードでクラウドターゲットに直接データをストリーミング — 中間ファイルなし
- SUM起動時に必須の遅延チェックを実施；遅延が20msを超えるとCHECKS.logとDBSTAT.LOGに警告が記録される

**参考資料：**
- SAP Community — DMOVE2S4：クラウドへの一段階アプローチ（SAP公式）: https://community.sap.com/t5/technology-blog-posts-by-sap/dmove2s4-your-one-step-approach-into-the-cloud/ba-p/13872751
- SAP Help — ハイパースケーラー上のS/4HANAへのDMO移行（公式）: https://help.sap.com/docs/SLTOOLSET/7d57e56e12104cc68bce7646cd9f4cbf/c17f45ffa4004ca4837913a745b3a081.html

## ダウンタイム最適化DMO（doDMO）

大容量テーブルの移行と**データ変換**をアップタイムフェーズに移動します。ダウンタイムウィンドウは「最終デルタリプレイ + カーネルスイッチ + 整合性チェック」のみに絞られます。

### 実績データ（約30TBのデータベースの事例）：

| アプローチ | ダウンタイム |
|---|---|
| 標準DMOVE2S4 | 94.5時間 |
| ダウンタイム最適化DMO | **26.5時間** |
| **削減率** | **72%** |

### 前提条件：
- SUM 2.0 SP17以上（2023年5月に一般提供開始）
- チームメンバー最低1名がトレーニング **ADM329** を修了し、知識評価 **ADM329k** に合格していること
- ソースシステムにSUM Toolboxが導入済みであること（SAP Note 3092738）
- インパクト分析から取得したZDIMPANA.ZIPファイルが必要

**参考資料：**
- SAP Support — ダウンタイム最適化コンバージョンアプローチ（公式）: https://support.sap.com/en/tools/software-logistics-tools/software-update-manager/downtime-optimized-conversion-approach.html
- SAP Learning — ダウンタイム最適化コンバージョンの概念（公式）: https://learning.sap.com/courses/minimizing-downtime-of-system-conversion-to-sap-s-4hana-with-downtime-optimized-conversion-of-sum/understanding-the-concept-of-downtime-optimized-conversion
- SAP Community — 大規模DBに向けたdoDMO戦略（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/strategies-for-downtime-optimized-dmo-dodmo-doc-dmove2s4-for-large/ba-p/14366512

## キーコンセプトまとめ

| 概念 | 技術的役割 |
|---|---|
| **R3load** | SAPデータ移行ユーティリティ；並列エクスポート+インポートペアで動作 |
| **メモリパイプ** | R3loadペア間のメインメモリ直接チャネル；ファイルなし、圧縮なし |
| **バケット** | テーブルのワークパッケージ；1ペアが1バケットを順次処理 |
| **シャドウインスタンス** | ターゲットカーネルを使用する追加ABAPインスタンス；シャドウリポジトリとデルタ書き込みを管理 |
| **シャドウリポジトリ** | アップタイム中にターゲットHANA DB上に構築される完全なABAPリポジトリ |
| **CRR** | Change Recording & Replication；アップタイム移行中にトリガー経由でユーザー変更をキャプチャ |
| **Writer（TMP）** | ターゲットカーネルを使用してCRRデルタ変更をターゲットHANAに書き込む一時インスタンス |
| **MIG2NDDB_SWITCH** | DB接続をソースからターゲットHANAに切り替えるSUMフェーズ |
| **DMOVE2S4** | クラウド移行向けDMOバリアント；RISEプライベートクラウド向けでパイプモード必須 |
| **doDMO** | ダウンタイム最適化DMO；アップタイム中に大容量テーブルを事前移行・変換 |

## 参考資料

- SAP Support — DMO公式ページ: https://support.sap.com/en/tools/software-logistics-tools/software-update-manager/database-migration-option-dmo.html
- SAP Community — DMO入門（SAP公式）: https://community.sap.com/t5/technology-blog-posts-by-sap/database-migration-option-dmo-of-sum-introduction/ba-p/13262160
- SAP Community — シャドウリポジトリ on ターゲットDB（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/dmo-shadow-repository-on-target-database-for-conversion-to-sap-s4hana-1909/ba-p/13372144
- SAP Community — アップタイム中のテーブル移行によるダウンタイム最適化（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/dmo-downtime-optimization-by-migrating-app-tables-during-uptime/ba-p/13113084
- SAP Community — CRRコントロールセンターの紹介（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/introducing-the-new-crr-control-center-to-monitor-record-replay-in-sum-2-0/ba-p/13372144
