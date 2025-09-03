# RingBufferLib
RingBufferLibは、OMRON社のNX/NJ向けのBYTE型を対象としたリングバッファライブラリです。
このライブラリのリングバッファ (RingBuffer) は、バッファへの読み書きの主体がそれぞれ1つである時に限り、ロックを使用しなくてもマルチタスクに使用することができます。また、リングバッファの読み書きを追跡しての差分取得とバッファ情報を比較しての差分取得ができます。
["リングバッファの機能を彫り出す"](https://zenn.dev/kitam/articles/546f4610e9f39f)に幾らか付加的な情報があります。

以下は、RingBufferへの基本的な読み書きです。

```iecst
// リングバッファは初期化する必要がある。
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);

// リングバッファにSizeで指定した値より大きな書き込み可能な空きがない時、FALSEを返す。
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 // Byte array that holds the bytes to write.
                 In:=iWriteData,
                 // Starting position of the bytes to write.
                 // Default is 0.
                 Head:=0,
                 // Number of bytes to write.
                 // Default is 0.
                 Size:=iWriteDataSize,
                 // Whether to overwrite unread bytes.
                 // Default is FALSE (Do not overwrite).
                 AllowOverwrite:=FALSE);

// Sizeに指定した値を読み出すバイト数の最大値としてリングバッファから読み出す。
RingBuffer_read(Context:=iBufferContext,
                Buffer:=iBuffer,
                // Byte array into which the read bytes are written.
                Out:=iReadData,
                // Position at which to start writing bytes.
                // Default is 0.
                Head:=0,
                // Number of bytes to read. Default is 0.
                Size:=SizeOfAry(iReadData),
                // Number of bytes read.
                ReadSize=>iReadDataSize,
                // Whether an overflow occurred.
                Overflow=>iOverflow);
iReadDataStr := AryToString(In:=iReadData[0],
                            Size:=iReadDataSize);

iOk := iReadDataStr = 'data';
```

RingBufferは、BYTE型配列とのやり取りだけではなく、RingBuffer間での処理も行えます。
例えば、以下のようにRingBuffer間でバイト列を転送することができます。

```iecst
RingBuffer_init(Context:=iSourceBufferContext,
                Buffer:=iSourceBuffer);
RingBuffer_init(Context:=iDestinationBufferContext,
                Buffer:=iDestinationBuffer);

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iSourceBufferContext,
                 Buffer:=iSourceBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);

RingBuffer_getReadableSize(Context:=iSourceBufferContext,
                           Size=>iSourceReadableSize);

// 転送元バッファから転送先バッファにSizeで指定した値を転送する。
// 転送先バッファの書き込み可能なバイト数がSizeより小さいか、
// 転送元バッファの読み出し可能なバイト数がSizeより小さいとき、FALSEを返し何もしない。
RingBuffer_transfer(Source:=iSourceBufferContext,
                    // Byte array targeted by the source RingBufferContext.
                    SourceBuffer:=iSourceBuffer,
                    // Destination RingBufferContext.
                    Destination:=iDestinationBufferContext,
                    // Byte array targeted by the destination RingBufferContext.
                    DestinationBuffer:=iDestinationBuffer,
                    // Number of bytes to transfer.
                    Size:=iSourceReadableSize,
                    // Whether to overwrite unread bytes.
                    // Default is FALSE (Do not overwrite).
                    AllowOverwrite:=FALSE,
                    // Number of bytes transferred.
                    TransferSize=>iTransferSize);
                    
RingBuffer_read(Context:=iDestinationBufferContext,
                Buffer:=iDestinationBuffer,
                Out:=iReadData,
                Head:=0,
                Size:=SizeOfAry(iReadData),
                ReadSize=>iReadDataSize);
iReadDataStr := AryToString(In:=iReadData[0],
                            Size:=iReadDataSize);

iOk := iReadDataStr = 'data';
```

この他にも、リングバッファを使用する処理において必要となる幾つかのPOUを備えています。
`RingBufferLibExample.smc2`に例示があります。

## リングバッファとしての特性
RingBufferは、リングバッファとして以下の特性があります。

* 無限には使用できない   
  単調に増分する書き込み数と読み出し数を内部で使用しているためです。   
  1msに65535バイトの書き込みを行うとして、約8900年で使用上限に達します。

* バッファの最大容量は65535バイト
  定義可能な配列の最大の要素数が65535であるためです。

* バッファへの書き込みはオーバーフロー(未読み出し領域への書き込み)を許容する  
  ユーザーが指定しない限りオーバーフローしませんが、バッファへの書き込みを行うPOUはオーバーフローを許容するオプションを指定できます。

* バッファへの読み書きの主体がそれぞれ1つである時に限り、ロックを使用しなくてもマルチタスクに使用可能   
  単調に増分する書き込み数と読み出し数を使用することで、書き込み処理と読み出し処理が共通して更新する値が無いためです。

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

## ライブラリの使用手順
ライブラリ(RingBufferLib.slr)は以下の手順で使用します。

1.  `RingBufferLib.slr`をプロジェクトで参照する

2.  プロジェクトをビルドしてエラーが無いことを確認する   
    ライブラリは名前空間を使用しています。   
    プロジェクト内の識別子と名前空間の衝突が生じていないことを確認します。

## 例示プロジェクトについて
RingBufferLibの例示として以下のSysmacプロジェクトがあります。

* `RingBufferLibExample.smc2`   
  RingBufferLibの基本的な使用方法についての例示です。   
  シミュレータで実行して確認することができます。   
  RingBufferLibの単体テストを含みます。

* `examples\`   
  RingBufferLibを使用した実践的な例示です。   
  各プログラムともコントローラで動作させることを前提にしています。

## RingBufferの差分取得について
RingBufferは、読み書きを追跡しての差分取得とバッファ情報を比較しての差分取得ができます。
差分取得は、リングバッファ自体の実装に要する基本情報のみで行い、特別な機構や追加情報は使用しません。
そのため、バッファ容量や差分取得のタイミングによる制約がありますが、一般的なリングバッファの使用状態において意図した動作を期待できます。
また、それら制約は設計上のトレードオフ対象として取り扱うことができるため、要件に合わせて取捨選択することができます。

差分は、バッファへの書き込みである書き込み差分とバッファからの読み出しである読み出し差分を対象としています。
2つのバッファ情報の差異の指す値がバッファに残っているとき差分を取得できます。
バッファ情報は、その初期化から使用によって常に増分する書き込み数と読み出し数を保持しています。
そのため、2つのバッファ情報を比較することで常に差異を取得することができます。
しかし、バッファは循環的に使用されるため、差異の指す値がバッファに残っているかは状況によります。

差分は、バッファへの書き込みによって差分を保持する領域を上書きすることで消費します。
そのため、バッファが使用される限り、読み出し差分の取得可能期間は、書き込み差分と比較して常に短くなります。
書き込み差分の消費は、バッファ容量の書き込みが生じたときであるのに対し、読み出し差分の消費は、読み出した時点の書き込み可能な容量の書き込みが生じたときであるためです。

また、差分取得は、バッファに対する並列的な処理です。
そのため、処理中にバッファへの読み書きが生じて処理の開始時点と終了時点で状態が変化する可能性があります。
差分取得を行うPOUは戻り値として、バッファ状態に変化が無いかを確認し、変化があるとFALSEを返します。
差分取得中にバッファ状態が変化しても、多くの場合で、再度取得することでその時点での差分を得ることができます。
継続的な処理であれば次の処理に持ち越しても問題ないかもしれません。
しかし、バッファへの書き込みが、バッファ状態の変化に先行して実行されることには注意が必要です。
非常に高頻度に循環するバッファの差分取得でPOUがFALSEを返したとき、取得した差分は新しく書き込まれた値を含んでいる可能性があります。
そのようなバッファからの差分取得が必要である場合、そのバッファへの操作と同期的にスケジューリングするか、バッファ容量を増やして対応します。

---
### 差分取得の制約
リングバッファは、その用途の多くの場合でオーバーフローすることも非常に高頻度に循環することも意図しません。
そのため、制約は多くの場合で制約とならない可能性が高いです。
また、いずれもバッファ容量の調整と設計上の工夫で対処可能です。

#### 書き込み差分取得の制約
書き込み差分取得には、以下の制約があります。

* 比較期間でバッファ容量を超える書き込みが行われていない   
  バッファ容量を超える書き込みは、未取得である差分を保持する領域への書き込みです。

#### 読み出し差分取得の制約
読み出し差分取得には以下の制約があります。

* 比較期間の開始時点の書き込み可能な容量を超える書き込みが行われていない

* オーバーフローしていない

* 比較期間にオーバーフローが発生して解消し、正常な読み出しが行われていてもその読み出しは取得できない   
  バッファ操作と比較期間中のオーバーフローの解消は認識できません。

#### 書き込み差分の再取得の制約
書き込み差分の再取得には以下の制約があります。

* FALSEを返した差分取得処理の開始時点からバッファ容量を超える書き込みが行われていない

#### 読み出し差分の再取得の制約
読み出し差分の再取得には以下の制約があります。

* FALSEを返した差分取得処理の開始時点で書き込み可能であったバイト数を超える書き込みが行われていない

---
### 差分取得 POU
差分取得には、以下のPOUを使用します。

* `RingBuffer_createWriteTracker`   
   バッファの書き込みを追跡するためのトラッカーを作成します。

* `RingBuffer_createReadTracker`   
   バッファの読み出しを追跡するためのトラッカーを作成します。

* `RingBuffer_createTracker`   
   バッファの書き込みと読み出しを追跡するためのトラッカーを作成します。

* `RingBuffer_pullWrite`   
   `RingBuffer_createWriteTracker`で作成したトラッカーを使用して書き込み差分をRingBufferに取得します。

* `RingBuffer_pullRead`   
   `RingBuffer_createReadTracker`で作成したトラッカーを使用して読み出し差分をRingBufferに取得します。

* `RingBuffer_pull`   
   `RingBuffer_createTracker`で作成したトラッカーを使用して書き込み差分と読み出し差分をそれぞれRingBufferに取得します。

* `RingBuffer_diffWrite`   
   バッファ情報を比較して書き込み差分をBYTE型配列に取得します。

* `RingBuffer_diffRead`   
   バッファ情報を比較して読み出し差分をBYTE型配列に取得します。

* `RingBuffer_diff`   
   バッファ情報を比較して書き込み差分と読み出し差分をそれぞれBYTE型配列に取得します。

---
#### 書き込み追跡
書き込みの追跡は、バッファへの書き込みについて継続的に差分を取得することです。
`RingBuffer_createWriteTracker`でトラッカーを作成し、定期的に`RingBuffer_pullWrite`で差分を取得し続けることで、バッファへの書き込みを追跡します。
以下が例です。

```iecst
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);
RingBuffer_init(Context:=iTrackWriteBufferContext,
                Buffer:=iTrackWriteBuffer);

// トラッカーを作成する。
// トラッカーを作成した時点の状態が比較の起点になる。
RingBuffer_createWriteTracker(Target:=iBufferContext,
                              Tracker=>iWriteTracker);

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);

// トラッカーとの差分をリングバッファに取得する。
// 差分の取得に成功するとトラッカーを更新し、比較した状態を比較の起点にする。
// 正常に差分を取得できるとTRUEを返す。
RingBuffer_pullWrite(Context:=iTrackWriteBufferContext,
                     // Byte array of the RingBuffer to which the obtained difference is written.
                     Buffer:=iTrackWriteBuffer,
                     // Tracking context.
                     Tracker:=iWriteTracker,
                     // RingBufferContext of the RingBuffer being tracked.
                     Tracked:=iBufferContext,
                     // Byte array of the RingBuffer being tracked.
                     TrackedBuffer:=iBuffer,
                     // Whether to overwrite unread bytes in the RingBuffer where the difference is written.
                     // Default is FALSE (Do not overwrite).
                     AllowOverwrite:=FALSE,
                     // Whether the retrieved difference has missings.
                     Missing=>iMissing,
                     // Number of bytes of difference obtained.
                     PullSize=>iPullSize);

RingBuffer_read(Context:=iTrackWriteBufferContext,
                Buffer:=iTrackWriteBuffer,
                Out:=iTrackData,
                Head:=0,
                Size:=SizeOfAry(iTrackData),
                ReadSize=>iTrackDataSize);
iTrackDataStr := AryToString(In:=iTrackData[0],
                            Size:=iTrackDataSize);

iOk := iTrackDataStr = 'data';
```

---
#### 読み出し追跡
読み出し追跡は、バッファからの読み出しについて継続的に差分を取得することです。
`RingBuffer_createReadTracker`でトラッカーを作成し、定期的に`RingBuffer_pullRead`で差分を取得し続けることで、バッファからの読み出しを追跡します。
以下が例です。

```iecst
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);
RingBuffer_init(Context:=iTrackReadBufferContext,
                Buffer:=iTrackReadBuffer);

// トラッカーを作成する。
// トラッカーを作成した時点の状態が比較の起点になる。
RingBuffer_createReadTracker(Target:=iBufferContext,
                             Tracker=>iReadTracker);

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                         AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);
RingBuffer_read(Context:=iBufferContext,
                Buffer:=iBuffer,
                Out:=iReadData,
                Head:=0,
                Size:=SizeOfAry(iReadData),
                ReadSize=>iReadDataSize);
iReadDataStr := AryToString(In:=iReadData[0],
                            Size:=iReadDataSize);

// トラッカーとの差分をリングバッファに取得する。
// 差分の取得に成功するとトラッカーを更新し、比較した状態を比較の起点にする。
// 正常に差分を取得できるとTRUEを返す。
RingBuffer_pullRead(Context:=iTrackReadBufferContext,
                    // Byte array of the RingBuffer to which the obtained difference is written.
                    Buffer:=iTrackReadBuffer,
                    // Tracking context.
                    Tracker:=iReadTracker,
                    // RingBufferContext of the RingBuffer being tracked.
                    Tracked:=iBufferContext,
                    // Byte array of the RingBuffer being tracked.
                    TrackedBuffer:=iBuffer,
                    // Whether to overwrite unread bytes in the RingBuffer where the difference is written.
                    // Default is FALSE (Do not overwrite).
                    AllowOverwrite:=FALSE,
                    // Whether the retrieved difference has missings.
                    Missing=>iMissing,
                    // Number of bytes of difference obtained.
                    PullSize=>iPullSize,
                    // Whether the RingBuffer tracked has overflowed.
                    Overflow=>iOverflow);

RingBuffer_read(Context:=iTrackReadBufferContext,
                Buffer:=iTrackReadBuffer,
                Out:=iTrackData,
                Head:=0,
                Size:=SizeOfAry(iTrackData),
                ReadSize=>iTrackDataSize);
iTrackDataStr := AryToString(In:=iReadData[0],
                             Size:=iReadDataSize);

iOk := iReadDataStr = 'data'
    AND iTrackDataStr = 'data';
```

---
#### 書き込み差分取得
書き込み差分取得は、比較の起点となる時点のバッファ情報を保持しておき、そのバッファ情報と任意の時点のバッファ情報を比較して差分を取得します。
差分の出力先がRingBufferでない場合や、差分取得の失敗の可能性が高いRingBufferに対して使用します。
`RingBuffer_diffWrite`で差分を取得します。
継続して差分を取得する場合、POUが返す`ComparedContext`を次回に使用することで空隙の無い差分取得ができます。
以下が例です。

```iecst
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);

// 比較の起点となる状態を保存する。
iCompareContext := iBufferContext;

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);

// 比較するRingBufferContextとの書き込み差分を取得する。
// 継続して差分を取得したい場合、RingBuffer_diffWriteの戻り値がTRUEであることを確認し、
// ComparedContextを保存する。
RingBuffer_diffWrite(Context:=iBufferContext,
                     Buffer:=iBuffer,
                     // RingBufferContext to compare and calculate the difference.
                     RefContext:=iCompareContext,
                     // Byte array that stores the difference.
                     Diff:=iDiff,
                     // Starting position of the byte array where the difference will be stored. Default is 0.
                     Head:=0,
                     // Number of bytes of the difference stored.
                     Size=>iDiffSize,
                     // Whether the retrieved difference has missing.
                     Missing=>iMissing,
                     // Current RingBufferContext being compared.
                     ComparedContext=>iComparedContext);
iDiffStr := AryToString(In:=iDiff[0],
                        Size:=iDiffSize);

iOk := iDiffStr = 'data';
```

---
#### 読み出し差分取得
読み出し差分取得は、比較の起点となる時点のバッファ情報を保持しておき、そのバッファ情報と任意の時点のバッファ情報を比較して差分を取得します。
差分の出力先がRingBufferでない場合や、差分取得の失敗の可能性が高いRingBufferに対して使用します。
`RingBuffer_diffRead`で差分を取得します。
継続して差分を取得する場合、POUが出力する`ComparedContext`を次回に使用することで空隙の無い差分取得ができます。
以下が例です。

```iecst
RingBuffer_init(Context:=iBufferContext,
                Buffer:=iBuffer);

// 比較の起点となる状態を保存する。
iCompareContext := iBufferContext;

iWriteDataStr := 'data';
iWriteDataSize := StringToAry(In:=iWriteDataStr,
                              AryOut:=iWriteData[0]);
RingBuffer_write(Context:=iBufferContext,
                 Buffer:=iBuffer,
                 In:=iWriteData,
                 Head:=0,
                 Size:=iWriteDataSize);

// 比較するRingBufferContextとの書き込み差分を取得する。
// 継続して差分を取得したい場合、RingBuffer_diffWriteの戻り値がTRUEであることを確認し、
// ComparedContextを保存する。
RingBuffer_diffWrite(Context:=iBufferContext,
                     Buffer:=iBuffer,
                     // RingBufferContext to compare and calculate the difference.
                     RefContext:=iCompareContext,
                     // Byte array that stores the difference.
                     Diff:=iDiff,
                     // Starting position of the byte array where the difference will be stored. Default is 0.
                     Head:=0,
                     // Number of bytes of the difference stored.
                     Size=>iDiffSize,
                     // Whether the retrieved difference has missing.
                     Missing=>iMissing,
                     // Current RingBufferContext being compared.
                     ComparedContext=>iComparedContext);
iDiffStr := AryToString(In:=iDiff[0],
                        Size:=iDiffSize);

iOk := iDiffStr = 'data';
```
