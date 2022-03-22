# マルチカートMBC

マルチカートMBCとは、1つのカートリッジに複数のゲームソフトを内包しているカートリッジのことです。内部実装の違いからMBC1m, MMM01, EMSなどいくつかの種類が存在しています。

## MBC1m

MBC1mはMBC1と同じICを利用したマルチカートMBCです。よってメモリやレジスタの役割はMBC1と同じですが、**MBC1mは、MBC1のA18アドレス出力をROMに接続していません。**

これにより、複数の1MB（16バンク）のゲームを搭載することができ、[RAMバンクの選択](./mbc1.md#4000-5fff---ramバンク番号--romバンク番号の上位bit-w)（\$4000）により、最大4つのゲームのうちどのゲームをスイッチインするかを選択することができます。

理論的には、MBC1MのA17とA16の出力を接続しないことで、1Mbitや512kbitのゲームを作ることができますが、ライセンスゲームでは行われていないようだ。

### MBC1mの仕組み

MBC1mは、メインROMのバンクレジスタの最上位bitを無視して4bitレジスタとして扱い、2bitレジスタをバンク番号のbit4-5（通常のbit5-6ではなく）に適用するという代替配線を採用しています。

つまり、[モード1](./mbc1.md#6000-7fff---バンクモードセレクト-w)では、2bitレジスタでバンク$00、$20、$40、$60を選択するのではなく、$00、$10、$20、$30を選択することになります。

```
x: ROMバンク番号のbit(0x2000..3FFF)
y: ROMバンク番号の上位bit(0x4000..5FFF)

bit      7 6 5 4 3 2 1 0
------------------------
MBC1  => 0 y y x x x x x
MBC1m => 0 0 y y x x x x
```

これらのカートリッジは、モード1が`$0000..3FFF`の領域をリマップすることを利用してゲームを切り替えています。

2bitのレジスタはゲームの選択に使われ、メモリ領域`$0000.3FFF`と`$4000..7FFF`に対応しているバンクを選択されたゲームのものに切り替え、その後ゲームはメインROMのバンクレジスタのみを変更します。選択されたゲームは、256KiBのカートから実行されていることになります。

これらのカートは通常、バンク`$10`に任天堂の著作権ヘッダがあることで識別できます。不良ダンプされたMBC1mのROMは、バンク\$10-\$1Fに\$00-\$0Fのダブりとバンク\$30-\$3Fに\$20-\$2Fのダブりと、重複したコンテンツがあることで識別できます。

MBC1mROMは、ROMサイズを1MBから2MBにして、各サブROMを複製することで、通常のMBC1 ROMに変換することができます。つまり、\$00-\$0Fを\$10-\$1Fに複製し、元の\$10-\$1Fを\$20-\$2Fに配置し、元の\$20-\$2Fを\$30-\$3Fに複製する、といった具合です。

## MMM01

[こちら](https://wiki.tauwasser.eu/view/MMM01)を参照

## EMS

PinoBatch learned the game selection protocol for EMS flash carts from beware, who in turn learned it from nitro2k01. Take this with a grain of salt, as it hasn't been verified on the authentic EMS hardware.

A [header](../header.md) matching any of the
following is detected as EMS mapper:

-   Header name is "EMSMENU", NUL-padded
-   Header name is "GB16M", NUL-padded
-   Cartridge type ($0147) = $1B and region ($014A) = $E1

Registers:

$2000 write: Normal behavior, plus save written value in $2000 latch
$1000 write: $A5 enables configure mode, $98 disables, and other values have no known effect
$7000 write while configure mode is on: Copy $2000 latch to OR mask

After the OR mask has been set, all reads from ROM will OR A21-A14 (the
bank number) with the OR mask. This chooses which game is visible to the
CPU. If the OR mask is not aligned to the game size, the results may be
nonsensical.

The mapper does not support an outer bank for battery SRAM.

To start a game, do the following in code run from RAM: Write $A5 to
$1000, write game starting bank number to $2000, write any value to
$7000, write $98 to $1000, write $01 to $2000 (so that 32K games
work), jump to $0100.

