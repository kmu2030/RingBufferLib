# この例示について
この例示は、RingBufferLibを使用する[UdpClientLib](https://github.com/kmu2030/UdpClientLib)と[FileStreamWriterLib](https://github.com/kmu2030/FileStreamWriterLib)を組み合わせたログ出力を行います。

[examples/logger](https://github.com/kmu2030/RingBufferLib/tree/main/examples/logger)と処理内容は同じですが、Loggerのバッファを保持メモリとして電源断を越えて値を保持します。
また、ログのSDカードへの書き込みとリモートUDPエンドポイントへの送信は、
それら出力プログラムが起動する以前にロガーが書き込んだログも参照して出力します。

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


