# DMA転送(OMA DMA)

このページは[OAM DMA Transfer](https://gbdev.io/pandocs/OAM_DMA_Transfer.html)を翻訳したものです。

HDMAは[別のページ](../../cgb/hdma.md)で説明しています。

## 概要

DMA転送は`Direct Memory Access`転送の略で、CPUを介さずに周辺機器やメモリ間で直接データ転送を行う転送方式です。

ゲームボーイではROMまたはRAMからOAMへのDMA転送が行えます。

## FF46 - DMA - DMA制御レジスタ (R/W)

このレジスタに書き込むと、ROMまたはRAMからOAMへのDMA転送が開始されます。

このレジスタに書き込まれる値は、転送元アドレスを\$100で割ったものになります。

つまり転送元と転送先は次のようになります。

```
転送元: $XX00-$XX9F ; XX = $00~$DF
転送先: $FE00-$FE9F
```

転送には 160 [M-cycle](../../cpu/cycle.md) かかります。よって転送にかかる時間は、通常速モードでは152μs、CGB倍速モードでは76μsです。

DMGでは、この間、CPUはHRAM（`$FF80..FFFE`）のみにアクセスできます。

CGBでは、転送元のメモリ領域で使用しているバスは使用できません。今のところ、CGBのこの点はよく理解されていないので、DMGと同じ動作とすることが推奨されています。

そのためソフトの開発者はDMAを利用する際には、次のような短いプログラムをHRAMにコピーし、このプログラムを使ってHRAM内部から転送を開始し、転送が終了するまで待つ必要があります。

```asm
run_dma:
    ld a, HIGH(start address)
    ldh [$FF46], a  ; start DMA transfer (starts right after instruction)
    ld a, 40        ; delay for a total of 4×40 = 160 cycles
.wait
    dec a           ; 1 cycle
    jr nz, .wait    ; 3 cycles
    ret
```

(OAM)DMA転送中はスプライトが表示されないため、多くのプログラムではVBlankハンドラ内で上記のコードを実行しています。

```
Note:

ディスプレイの描画中（モード2,3）にDMA転送を実行することも可能で、こうすると40個以上のスプライトを画面に表示することができます（例えば、上半分に40個のスプライト、下半分に40個のスプライト）

しかし、その代償として、数行に渡ってPPUがOAMを\$FFとして読み込むため、スプライトが不足します。また、モード3でOAMのDMA転送が開始されると、グラフィックの不具合が発生する可能性があります。
```

