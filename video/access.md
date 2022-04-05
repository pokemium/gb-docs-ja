# VRAM・OAMへのアクセス

PPUが画面を描画しているときは、ビデオメモリ（VRAM）とスプライト属性テーブル（OAM）を直接読み込んでいます。

この間、ゲームボーイのCPUはVRAMやOAMにアクセスできません。つまり、VRAMやOAMに書き込もうとしても無視され、内容は変更されません。

また、VRAMやOAMから読み出そうとすると、未定義データ（通常は`$FF`）が返されます。

このため、プログラムは実際に読み書きする前に、VRAM/OAMにアクセスできるかどうかを確認する必要があります。これは通常、STATレジスタ（FF41）からモードビット(bit0-1)を読み出すことで行います。これを行う場合（以下の例で説明します）、待機ループと次のメモリアクセスの間に割り込みが発生しないように注意する必要があります。

## VRAMへのアクセス

VRAM(`0x8000..9fff`)にアクセス可能なのは、モード0~2のときです。

```
モード0: HBlank
モード1: VBlank
モード2: OAM Search
```

VRAMへのアクセスを待つ典型的なサンプルコードは以下の通りです。

```asm
    ld   hl, $FF41     ; $FF41=STAT
.wait
    bit  1, [hl]       ; モードが0か1になるまで待つ
    jr   nz, .wait
```

仮に、モード0またはモード1の終了時に上記のサンプルコードを実行したとしても、どちらの場合も次の期間はモード2となり、VRAMへのアクセスが可能なため、あと数サイクルはVRAMへのアクセスが可能と考えてよいでしょう。

ただし、STAT割り込みなど、割り込みから復帰するまでにモード3に戻ってしまう可能性のある割り込みには注意が必要です。

また、CGBモードでVRAMにデータを書き込むには、HDMA（`FF51..FF55`）を使用する方法があります。

STAT割り込みが不要な場合は、

- モード0以外の個々のSTAT割り込みを無効（STAT.3）
- STAT割り込みを有効（IE.1）
- DI命令でIMEを無効

にしてHALT命令を使用することで、モード0の開始まで待つこともできます。This allows use of the entire Mode 0 on one line and Mode 2 on the following line, which sum to 165 to 288 dots. For comparison, at single speed (4 dots per machine cycle), a copy from stack that takes 9 cycles per 2 bytes can push 8 bytes (half a tile) in 144 dots, which fits within the worst case timing for mode 0+2.

## OAMへのアクセス

OAM(`0xFE00..FE9F`)にアクセス可能なのは、モード0~1のときです。

```
モード0: HBlank
モード1: VBlank
```

これらのモードの間、OAMは直接またはDMA転送(`0xFF46`)によってアクセスすることができます。

これらのモード以外では、DMAがPPUより優先してOAMにアクセスするため、PPUはその間にOAMから`0xFF`を読み出します。

OAMへのアクセスを待つ典型的なサンプルコードは以下の通りです。

```asm
    ld   hl, $FF41    ; STAT Register

; モード0、モード1でなくなるまで待つ
.waitNotBlank
    bit  1, [hl]
    jr   z, .waitNotBlank

; モード0、モード1の開始まで待つ(ほとんどの場合モード0が来る)
.waitBlank
    bit  1, [hl]
    jr   nz, .waitBlank
```

サンプルコードの2つのループにより、モード0（フレームの終わりであればモード1）がサンプルコードの実行完了後、数クロックサイクル続くことが保証されるので、OAMへのアクセスを行うことができます。

もし、VBlank期間を待つ必要があるのであれば、すべての手順をスキップして、代わりにSTAT割り込みを使用する方が良いでしょう。

いずれにしても、DMA転送を行う方が、OAMに直接書き込むよりも効率的です。

```
Note:

LCDC.7でディスプレイを無効にしている間は、VRAMもOAMもアクセス可能です。  
ただし、この間は画面が真っ白になるので、初期化時にのみディスプレイを無効にすることをお勧めします。
```

