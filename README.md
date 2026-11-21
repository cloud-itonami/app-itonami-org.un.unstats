# org.un.unstats

`unstats.un.org` の UN SDG API から取得した、世界のSDG 7エネルギー指標 snapshot を保全する DataLad dataset です。地域識別子は国連 M49 (`geoAreaCode`) を原典どおり保存します。統合 datom は `etzhayyim/global-energy-datoms` が生成します。

この dataset は、電力アクセス（SDG 7.1.1）と最終消費に占める再生可能エネルギー比率（SDG 7.2.1）を保持します。都市部・農村部の内訳を含む7.1.1について、統合層は国全体を表す `ALLAREA` のみを投影します。

## 2000–2023 snapshot（`:series`）

`raw/sdg-series/` に2系列の2000–2023年 snapshot を原典の応答のまま保全します。取得 endpoint は UN SDG API v1 の `GET https://unstats.un.org/sdgapi/v1/sdg/Series/Data`（応答は `ApiObservationPage` 形式で `totalElements` / `totalPages` / `pageNumber` / `size` / `data` を含み、既存 snapshot と同形）です。`seriesCode` と年一覧（`timePeriod` を2000から2023まで繰り返し指定）、`page`、`pageSize=1000` で取得しました。`timePeriodStart` / `timePeriodEnd` による範囲指定は server 側で end が無視されるため使っていません。

* `EG_ACS_ELEC`（7.1.1、電力アクセス）: 19ページ、18072件、年別753件×24年、地域251
* `EG_FEC_RNEW`（7.2.1、再生可能エネルギー比率）: 7ページ、6504件、年別271件×24年、地域271

取得前の検証として2022年単年を同条件で要求し、件数が既存ファイルと一致することを確認しました（`EG_ACS_ELEC` 753件、`EG_FEC_RNEW` 271件、改訂なし）。要求 URL・sha256・バイト数・取得時刻（UTC）は `raw/source-catalog.edn` の `:series` に記録しています。検証は `nbb bin/verify-source-catalog.cljk`（file 読みのみ、network なし）で行います。
