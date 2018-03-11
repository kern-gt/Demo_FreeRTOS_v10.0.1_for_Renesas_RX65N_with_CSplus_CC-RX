# FreeRTOS Ver10.0.0 DemoProject for Renesas RX65N (CS+,CC-RX)

　このデモはFreeRTOSv10.0.1をRenesas RX65Nマイコン用に移植したものです。評価ボード上の2つのLEDを2タスクでLチカするだけの簡単なサンプルです。そのままビルドしてデバッグモードで動きます。
 
　最近ルネサスから発売された[Target Board for RX family](https://www.renesas.com/ja-jp/products/software-tools/boards-and-kits/cpu-mpu-boards/rx-family-target-board.html)(RX65N)を手に入れたのでFreeRTOSを動かしてみました。
秋葉原の[マルツ](https://www.marutsu.co.jp/pc/i/953239/)で2,980円（税抜き）で売られています。デバッガのE2Liteがボード上に搭載されてこの値段は安いと思います。

　なお、ド素人の学生が趣味で作成したものですので、もしご利用の際はくれぐれも自己責任でお願いします。
 
参考にさせていただいたもの  
　<https://blog.goo.ne.jp/lm324/e/99f735aef942ce6b965cd2f985acdf73>  
　<http://be-con.jp/shiryo/renesas-rx62-freertos-csprj.html>  
　<https://www.freertos.org/RX64M_RTOS_Renesas_GCC_e2studio.html>  
大変参考になりました。ありがとうございました。

文字コードは __UTF-8__ を使用しています。

## 動作環境
* FreeRTOS:v10.0.1 (RX600 RXv2)  
* 開発環境：CS+forCC V6.01.00  
* コンパイラ：CC-RX V2.08.00 (C99)  
* CPUボード：TARGET BOARD for RX65N (RTK5RX65N0C00000BR)  
* CPU(ボード上)：R5F565NEDDFP (100-pin LFQFP,120MHz,RAM 640KB,ROM 2MB+32KB)  
* "エミュレータ"(ボード上) ~~"E2エミュレータLite"~~：CS+環境ではデバッグに使用できました。E2Liteとして認識しますが、ドキュメントの方では「エミュレータ」としか書いてないので厳密にはE2Liteではないようです(チップの見た目はRX231?)。また、RFPによるプログラム書き込みはできません(現在)。


## サンプルコード内容
CPUボード上のLED0(PD6)を1Hz、LED1(PD7)を5Hzで点滅させる2つのタスクを動かします。

　クロック発生回路とポート初期化をスマートコンフィグレータで設定しています。 
FreeRTOSではカーネルタイマにコンペアマッチタイマ0(CMT0),コンテキストスイッチにソフトウェア割込み(SWINT)を使用しているので、その周辺機能は使用しないでください。カーネルタイマなどはソース改変すればCMT0以外のタイマも使えるはずです。

## ファイル構成
　自力でFreeRTOSプロジェクトを作るためのメモになります。サンプルコードの`main.c`と`ApplicationHook.c`は公式サンプルコードを参考に作成しました。

  1. CS+でプロジェクト新規作成します。ここで自分はビルド設定(CC-RXのプロパティ)で文字コードをUTF-8に変更してしまいますが、SHIFT-JISのままでいけるかどうかは未検証です。ちなみにUTF-8の変更箇所はコンパイル・オプションで2か所、アセンブル・オプションで1か所です。
  1. スマートコンフィグレータで周辺機能の設定を必要があれば設定してください。ただし、コンペアマッチタイマ(CMT0)、ソフトウェア割込み(SWINT)はFreeRTOS側が使うので何もせずほっといてください。  
  1. サンプルコードのFreeRTOSフォルダ以下をそのままプロジェクトフォルダにコピーしてプロジェクトに登録してください。
  1. サンプルコードの`ApplicationHook.c`の中身はフック関数類を定義してあります。`FreeRTOSConfig.h`でフック関数の有無を設定できますが、取りあえず最小限のものだけ定義してあります。取りあえずこれもファイルごとコピーしてプロジェクトに登録してください。また、必要に応じて自力で関数の追加記述をしてください。
  1. サンプルコードのmain.cにある`vApplicationSetupTimerInterrupt()`はカーネルタイマの初期設定で必ず必要です。関数を新規の`main.c`にコピーするなり新たにソースファイル作るなり自由に定義してください。
  1. `/FreeRTOS/FreeRTOSConfig.h`を目的に合わせて設定してください。
  1. 各ユーザソースファイルで`FreeRTOS.h, task.h, queue.h`など適切なヘッダをインクルードしてください。
  1. ビルドします。おそらく通る(はず?)。
  1. 任意でデバッグなどなど楽しんでください。

## 注意点
* ソースコードの文字エンコードに __UTF-8__ を使用するため、ビルド設定をShift-JISからUTF-8に変更してあります。
* FreeRTOSのメモリ管理ファイルは「heap_1.c」を使用しています。目的に応じて変更してください。`/FreeRTOS/portable/heap_1.c`に置いてあります。
* iodefine.hの不具合について  
プロジェクト生成直後はルートにRX65N用レジスタ定義ファイルが作成されます。しかし、スマートコンフィグレータを使用すると 
`/src/smc_gen/r_bsp/mcu/rx65n/register_access/iodefine.h`   
に新たに作られ、ルートにある方のファイルは登録が解除されます（ファイル削除はされない）。しかし、新たに作られた方はなぜかRX651用なので（バグ？）他のソースでiodefine.hをインクルードすると多重インクルードエラーが出ることがあります。対策として、
    1. インクルードパスをフルパスで記述する。
    1. ルートにあるRX65N用のファイルをカットし、RX651用のファイルにペーストする。ただし、スマートコンフィグレータでコード生成するたび元に戻ってしまう。
    1. RX651の機能のみしか使わないのであれば、ルートにあるRX65Nのファイルを最初に削除するだけ。

あたりを試してください。このサンプルではすでにRX65N用に差し替えてあります。
また、`/no_used/iodefine_rx65n/iodefine.h`にRX65N用のレジスタ定義ファイルを置いてあります（最新とは限りません）。
