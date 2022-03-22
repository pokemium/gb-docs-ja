# SGB機能を利用するには

## カートリッジヘッダ

SGBのゲームは、通常のゲームボーイのゲームと同様に、カートリッジのヘッダに任天堂の文字と適切なチェックサムが必要です。

また、SGB機能を解除するためには、カートリッジヘッダの[SGBフラグ](../cartridge/header.md#0146---sgbフラグ)と[ライセンスコード(旧)](../cartridge/header.md#014b---ライセンスコード旧)に決まった値を設定する必要があります。

これらの項目が設定されていない場合、ゲームはモノクロのゲームボーイのゲームと同様に動作しますが、SGBの特別な機能にはアクセスできません。

## SGBであるかどうかを検知する

ゲームプログラムは、ハードウェアがSGBであるかどうかを、起動直後のCレジスタの初期値を調べることで検出できます。0x14の値は、SGBまたはSGB2ハードウェアを示します。

また、起動直後のAレジスタの初期値を調べることで、SGBとSGB2を区別することができます。なお、DMGはSGBと、MGBはSGB2とAレジスタの初期値が同じです。

機種 | Aレジスタ | Cレジスタ
--------|------------|------------
DMG     | $01        | $13
SGB     | $01        | $14
MGB     | $FF        | $13
SGB2    | $FF        | $14
CGB     | $11        | $00
AGB     | $11        | $00

すべてのシステムの初期レジスタ値については、電源投入後のすべてのCPUレジスタの表を参照してください。

The SGB2 doesn’t have any extra features which’d require separate SGB2 detection except for curiosity purposes, for example, the game “Tetris DX” chooses to display an alternate SGB border on SGB2s.

Only the SGB2 contains a link port.

SGB hardware has traditionally been detected by sending MLT_REQ commands, but this method is more complicated and slower than checking the value of the A and C registers after startup. The MLT_REQ command enables two (or four) joypads; a normal handheld Game Boy will ignore this command, but an SGB will return incrementing joypad IDs each time when deselecting keypad lines (see MLT_REQ description). The joypad state/IDs can then be read out several times, and if the IDs are changing, then it is an SGB (a normal Game Boy would typically always return $0F as the ID). Finally, when not intending to use more than one joypad, send another MLT_REQ command in order to disable the multi-controller mode. Detection works regardless of how many joypads are physically connected to the SNES. However, unlike the C register method, this detection works only when SGB functions are unlocked from the cartridge header.

