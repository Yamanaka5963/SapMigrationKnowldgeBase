# 45 - プール/クラスターテーブルパターン（STXH / STXL / PCL1 / PCL2 / VBUK / VBUP）

> **背景：** DMO/SUM変換中に、SAPはプールテーブルとクラスターテーブルを自動的に透明テーブルに変換します。標準ABAP構文でこれらのテーブルを使用するカスタムコードは引き続き動作します。ただし、**ネイティブSQL**（EXEC SQL）を使用してプール/クラスターテーブルにアクセスするコードは破損するため、書き換えが必要です。

---

## パターン1 — ロングテキストアクセス：STXH / STXL → READ_TEXT ファンクションモジュール

### ビフォー（ECC）— 誤ったアプローチ（ネイティブSQL）

```abap
" ネイティブSQL経由でSTXH/STXLにアクセスしないでください — S/4HANAで破損
EXEC SQL.
  SELECT * FROM stxh WHERE tdname = :lv_name
ENDEXEC.
```

> **問題点：** STXH/STXLはECCではクラスターテーブルです。透明テーブルへの変換後、内部フォーマットが変わります。さらに重要なことに、ネイティブSQLを使用してアクセスすることは常に誤りでした — 標準FMを使用してください。

### ビフォー（ECC）— 正しいアプローチ（ECCとS/4HANA両方で動作）

```abap
DATA: lt_lines TYPE TABLE OF tline.

" 標準ファンクションモジュール — ECCとS/4HANA両方で動作
CALL FUNCTION 'READ_TEXT'
  EXPORTING
    id       = 'ABCD'       " テキストID
    language = 'J'          " 言語
    name     = '12345678'   " テキスト名（例：伝票番号）
    object   = 'VBBK'       " テキストオブジェクト（例：受注ヘッダー）
  TABLES
    lines    = lt_lines
  EXCEPTIONS
    not_found     = 4
    OTHERS        = 8.

IF sy-subrc = 4.
  " テキストが見つからない — 適切に処理
ELSEIF sy-subrc <> 0.
  " 予期しないエラー
ENDIF.
```

### アフター（S/4HANA）— 新アプローチ：I_TextObjectPlainLongText CDSビュー

S/4HANAでの新規開発では、SAPがロングテキストアクセス用のCDSビューを提供しています：

```abap
" CDSビュー経由でロングテキストを読み取り（S/4HANA推奨アプローチ）
SELECT * FROM i_textobjectplainlongtext
  INTO TABLE @DATA(lt_texts)
  WHERE language   = 'J'
    AND textobject = 'VBBK'
    AND textname   = '12345678'
    AND textid     = 'ABCD'
  ORDER BY PRIMARY KEY.
```

**参照元：** https://community.sap.com/t5/technology-blog-posts-by-members/accessing-long-text-of-sap-master-data-objects-or-fields-in-s4-hana-through/ba-p/13574939

---

## パターン2 — HR給与クラスター：PCL1 / PCL2

### ビフォー（ECC）

```abap
" IMPORT/EXPORTステートメント経由でHR給与結果クラスターにアクセス
" これはクラスターDBインターフェースを使用 — ネイティブSQLではない
DATA: st  TYPE pay99_result,
      ert TYPE hrpay99_ert.

IMPORT st ert FROM DATABASE pcl1(b1) ID b1_key.
```

> **重要：** PCL1/PCL2のIMPORT/EXPORTステートメントはSAPのクラスターDB抽象レイヤーを使用しており、ネイティブSQLではありません。DMO変換後、基礎となるストレージは透明テーブルになりますが、IMPORT/EXPORTインターフェースは変わりません。このコードはS/4HANAでも**引き続き動作**します。

### アフター（S/4HANA）— 推奨：標準HRファンクションモジュールを使用

```abap
" 推奨：給与データアクセスには標準HRファンクションを使用
CALL FUNCTION 'PYXX_READ_PAYROLL_RESULT'
  EXPORTING
    clusterid                    = 'RD'
    employeenumber               = lv_pernr
    sequencenumber               = '01'
    payroll_area                 = lv_area
  CHANGING
    payroll_result               = ls_result
  EXCEPTIONS
    illegal_isocode_or_clusterid = 1
    error_generating_import      = 2
    OTHERS                       = 3.
```

> **移行すべき場合：** PCLテーブルへのアクセスにネイティブSQL（EXEC SQL）を使用しているコードのみ。`IMPORT FROM DATABASE pcl1/pcl2`構文は安全です。

**参照元：** SAP Help — PCL1/PCL2クラスターアクセス: https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html

---

## パターン3 — 受注ステータス：VBUK / VBUP → VBAK / LIKP / VBRK

### ビフォー（ECC）

```abap
DATA: ls_vbuk TYPE vbuk,
      ls_vbup TYPE vbup.

" 受注ヘッダーステータスを読み取り
SELECT SINGLE * FROM vbuk INTO ls_vbuk
  WHERE vbeln = lv_vbeln.

" 明細ステータスを読み取り
SELECT SINGLE * FROM vbup INTO ls_vbup
  WHERE vbeln = lv_vbeln
    AND posnr = lv_posnr.
```

> **背景：** S/4HANAでは、VBUKとVBUPのステータスフィールドがメイン伝票ヘッダーテーブルに直接組み込まれます（受注はVBAK、出荷はLIKP、請求はVBRK）。独立したステータステーブルは互換構造として残りますが、SAPはメインテーブルからステータスフィールドにアクセスすることを推奨しています。

### アフター（S/4HANA）— ヘッダーテーブルから直接ステータスを読み取り

```abap
DATA: ls_vbak TYPE vbak.

" ステータスフィールドはVBAKに直接あります
SELECT SINGLE vbeln erdat ernam gbstk lfstk fkstk  " ヘッダー + ステータスフィールド
  FROM vbak
  INTO CORRESPONDING FIELDS OF @ls_vbak
  WHERE vbeln = @lv_vbeln.

" GBSTK = 全体処理ステータス
" LFSTK = 出荷ステータス
" FKSTK = 請求ステータス
```

### アフター（S/4HANA）— 伝票フローにはVBFAを使用（引き続き有効）

```abap
" VBFA（伝票フローテーブル）はS/4HANAでも変更なし
SELECT vbelv posnv vbeln posnn vbtyp_n
  FROM vbfa
  INTO TABLE @DATA(lt_vbfa)
  WHERE vbelv IN @so_vbeln
  ORDER BY PRIMARY KEY.
```

**参照元：** https://www.kodyaz.com/sap-abap/s4hana-search-for-database-operations-db-operation-select-found.aspx

---

## まとめ

| テーブル | 分類 | S/4HANAでの状態 | 対応 |
|---|---|---|---|
| STXH/STXL（EXEC SQL経由） | **必須修正** | 透明、フォーマット変更 | READ_TEXT FMまたはCDSビューを使用 |
| STXH/STXL（READ_TEXT FM経由） | 変更不要 | 変更なしで動作 | そのまま維持 |
| PCL1/PCL2（IMPORT FROM DATABASE経由） | 変更不要 | 変更なしで動作 | そのまま維持；新規コードにはPYXX_READ_PAYROLL_RESULTを推奨 |
| PCL1/PCL2（EXEC SQL経由） | **必須修正** | 非対応 | 標準FMを使用して書き直し |
| VBUK/VBUP（読み取り） | 推奨修正 | 互換として存在 | VBAK/LIKP/VBRKからステータスを読み取り |
| VBFA | 変更不要 | S/4HANAで変更なし | そのまま維持 |

## 参考資料

- SAP Community — S/4HANAのCDS経由でのロングテキストアクセス（メンバー投稿）: https://community.sap.com/t5/technology-blog-posts-by-members/accessing-long-text-of-sap-master-data-objects-or-fields-in-s4-hana-through/ba-p/13574939
- SAP Help — S/4HANAのPCLクラスターテーブル: https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/c6c3ffd90792427a9fee1a19df5b0925/17a2e2539c70424de10000000a174cb4.html
- このKBのプール/クラスターリファレンス: [33 — プール/クラスター変換](../part3-database/33jp-pool-cluster-conversion.md)
