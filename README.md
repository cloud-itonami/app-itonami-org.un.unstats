# org.un.unstats

`unstats.un.org` の UN SDG API から取得した、世界のSDG 7エネルギー指標 snapshot を保全する DataLad dataset です。地域識別子は国連 M49 (`geoAreaCode`) を原典どおり保存します。統合 datom は `etzhayyim/global-energy-datoms` が生成します。

この dataset は、電力アクセス（SDG 7.1.1）と最終消費に占める再生可能エネルギー比率（SDG 7.2.1）を保持します。都市部・農村部の内訳を含む7.1.1について、統合層は国全体を表す `ALLAREA` のみを投影します。
