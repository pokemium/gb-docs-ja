# VRAMの転送

## 概要

パケット転送の他に、映像信号を利用して4KBの大きなデータブロックを**ゲームボーイからスーファミ**へと転送することができます。

これらの転送は、まず通常のパケット転送で末尾に`_TRN`を付けたコマンドを送信することで起動します。

4KBのデータブロックは、次のフレームでゲームボーイのディスプレイメモリからスーファミによって読み出されます。

## データの転送

通常、転送されるデータはゲームボーイのVRAMの`0x8000..8FFF`に格納されています。

スーファミはディスプレイの走査線からデータを受け取っても、ゲームボーイのメモリの`0x8000..8FFF`に格納されていたのと同じビットとバイトの並びを自動的に再現してくれます。

## ディスプレイの準備

The above method works only when recursing the following things: BG Map must display unsigned characters 00h-FFh on the screen; 00h..13h in first line, 14h..27h in next line, etc. The Game Boy display must be enabled, the display may not be scrolled, OBJ sprites should not overlap the background tiles, the BGP palette register must be set to E4h.

## 転送時間

転送のためのコマンドパケットを送信する前に，転送されるデータをVRAMに用意しておく必要があります。

実際の転送は、コマンド送信後の次のフレームの先頭から開始され、コマンド送信から5フレーム目の終了時に終了します。この5フレームにコマンドを送信したフレームはカウントされません。

SGBは複数のチャンクに分けてデータを読み出すため、転送中にVRAMのデータを変更してはいけません。

## ノイズを避けるには

転送中はディスプレイにノイズが入ってしまいます。

このノイズは、`MASK_EN`コマンドを使って転送前の状態で画面をフリーズさせることで回避することができます。

もちろん、これは実際にSGB上でゲームを実行する場合にのみ有効であり、携帯型ゲームボーイシステムでは使えない手法のため、やみくもにVRAMデータを送信する前にSGB上かどうかを検出する必要があります。
