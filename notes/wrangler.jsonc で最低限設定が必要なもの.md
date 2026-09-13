## サマリー

https://developers.cloudflare.com/workers/wrangler/configuration/#inheritable-keys

name, main, comnpatiablity_date が required

name
Worker の名前になる値。
アンダースコアは使えない。ハイフンの利用を推奨。

main
Worker が実行された際、最初に実行されるファイル。
ファイルパスを指定する。

compatiability_date
ランタイムで利用する Worker のバージョンを指定する。
フォーマットは yyyy-mm-dd
バージョンは永久に残るので、一切アップデートせずに同じバージョンを使い続けることも可能
https://developers.cloudflare.com/workers/configuration/compatibility-dates/#setting-compatibility-date
