# org.un.unstats

`unstats.un.org` の UN SDG API から取得した、世界のSDG 7エネルギー指標 snapshot を保全する DataLad dataset です。地域識別子は国連 M49 (`geoAreaCode`) を原典どおり保存します。統合 datom は `etzhayyim/global-energy-datoms` が生成します。

この dataset は、電力アクセス（SDG 7.1.1）と最終消費に占める再生可能エネルギー比率（SDG 7.2.1）を保持します。都市部・農村部の内訳を含む7.1.1について、統合層は国全体を表す `ALLAREA` のみを投影します。

## 2000–2023 snapshot（`:series`）

`raw/sdg-series/` に2系列の2000–2023年 snapshot を原典の応答のまま保全します。取得 endpoint は UN SDG API v1 の `GET https://unstats.un.org/sdgapi/v1/sdg/Series/Data`（応答は `ApiObservationPage` 形式で `totalElements` / `totalPages` / `pageNumber` / `size` / `data` を含み、既存 snapshot と同形）です。`seriesCode` と年一覧（`timePeriod` を2000から2023まで繰り返し指定）、`page`、`pageSize=1000` で取得しました。`timePeriodStart` / `timePeriodEnd` による範囲指定は server 側で end が無視されるため使っていません。

* `EG_ACS_ELEC`（7.1.1、電力アクセス）: 19ページ、18072件、年別753件×24年、地域251
* `EG_FEC_RNEW`（7.2.1、再生可能エネルギー比率）: 7ページ、6504件、年別271件×24年、地域271

取得前の検証として2022年単年を同条件で要求し、件数が既存ファイルと一致することを確認しました（`EG_ACS_ELEC` 753件、`EG_FEC_RNEW` 271件、改訂なし）。要求 URL・sha256・バイト数・取得時刻（UTC）は `raw/source-catalog.edn` の `:series` に記録しています。検証は `nbb bin/verify-source-catalog.cljk`（file 読みのみ、network なし）で行います。

## 2026-11-21 snapshot（`:snapshots`）

2026-11-21 の drift 報告で M49 overview page が pinned hash と一致しなくなったため、同ページと2系列の2022年応答を新しい dated set として保全します。既存ファイルは一切上書きしません。energy projection が読む pinned revision（2026-07-10）はそのまま残し、新 set へ移るかは owner が決めます。

* `raw/m49/2026-11-21/overview-en.html`: catalogued `:url` の再取得（1 request、redirect なし）。新 sha256 は `:snapshots` に記録。旧ページとの差分は表記の1列のみで、M49 `520` の国名が `Nauru` → `Naoero` に変わっています（各言語の翻訳表も同変更）。
* `raw/sdg/2026-11-21/`: 2系列の2022年を `pageSize=1000` で再取得。各1ページ（`EG_ACS_ELEC` 753件、`EG_FEC_RNEW` 271件、件数は旧 snapshot と一致）。値の改訂はありません。旧 snapshot との差分は `520` の `geoAreaName` のみで、応答 metadata の `dimensions` に `Reporting Type` の `Regional` code が増えています。
* `derived/2026-11-21/m49-iso3.edn`: `kbb bin/build-m49-crosswalk.cljk <input-html> <output-edn> <retrieved-at>` で新ページから再生成（引数なしの既定動作は従来どおり `derived/m49-iso3.edn` を再現します）。248件で、旧 crosswalk との差分は `520` の国名のみです。手編集はしていません。
* `derived/2026-11-21/DIFF.edn`: 旧 set と新 set の測定差分（crosswalk の件数・新旧いずれかのみに存在する M49 code・ISO3 差分・国名差分、系列ごとの新旧件数と値が異なる area code 数）。
* 検証: `nbb bin/verify-source-catalog.cljk` は `:snapshots` 行の sha256・バイト数、dated directory 配下の未登録 file の有無、snapshot page の `pageNumber` / `totalElements` / 件数も確認します。
