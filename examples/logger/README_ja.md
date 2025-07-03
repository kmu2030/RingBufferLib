# この例示について
この例示は、RingBufferLibを使用する[UdpClientLib](https://github.com/kmu2030/UdpClientLib)と[FileStreamWriterLib](https://github.com/kmu2030/FileStreamWriterLib)を組み合わせたログ出力を行います。

バッファとしてRingBufferLibのリングバッファを使用するLoggerのログ書き込みについて、UdpClientLibのUdpClientがリアルタイムにリモートのUDPエンドポイントに送信しつつ、FileStreamWriterLibのFileStreamWriterがコントローラのSDカードのログファイルに出力します。Loggerのログ書き込みは、2ms周期のプライマリタスクで動作します。1回のログサイズは概ね68バイトです。各サイクルで書き込みをしても問題ありませんが、FileStreamWriterのバッファ容量を大きくしてもSDカードへのフラッシュがそれなりの頻度になります。

## 使用環境
このプロジェクトの使用には、以下の環境が必要です。

| Item          | Requirement |
| :------------ | :---------- |
| コントローラ   | NX or NJ    |
| Sysmac Studio | 最新版を推奨 |

## 構築環境
このプロジェクトは、以下の環境で構築しています。

| Item            | Version              |
| :-------------- | :------------------- |
| コントローラ     | NX102-9000 Ver 1.64  |
| Sysmac Studio   | Ver.1.62             |


