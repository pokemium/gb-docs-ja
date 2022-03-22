# コマンド

SGBでは、ゲームボーイからスーファミにコマンドを送ることでスーファミに処理をさせることができます。

コマンドは、以下のコマンドパケットという形で転送されます。

## 📦 コマンドパケット

コマンドパケット（別名：レジスタファイル）は、JOYPADレジスタ（FF00h）のP14およびP15出力ラインを使用して、**ゲームボーイからスーファミ**に転送されます。

これらの出力ラインは、ゲームボーイのキーボードマトリックスの2列を選択するためにも使用されています。（これはSGBでも機能しています）

### bitの転送

コマンドパケットの転送はP14とP15の両方をLOWにすることで、SNESのパケット受信プログラムを、リセットした後に開始します。

P14=LOWでbitの`0`、P15=LOWでbitの`1`となり、LSBから順にデータが転送されます。

```
    RESET  0   0   1   1   0   1   0
P14  --_---_---_-----------_-------_--...
P15  --_-----------_---_-------_------...
```

Data and reset pulses must be kept LOW for at least 5us. P14 and P15 must be kept both HIGH for at least 15us between any pulses. Obviously, it’d be no good idea to access the JOYPAD register during the transfer, for example, in case that your VBlank interrupt procedure reads-out joypad states each frame, be sure to disable that interrupt during the transfer (or disable only the joypad procedure by using a software flag).

### パケットの転送

各パケットはRESETパルスで起動され、128bitのデータが転送され（16バイト, 最初のバイトのLSBから転送される）、最後にストップビットとして "0"ビットを転送する必要があります。通常のパケットの構造は次のようになっています。

1. 1パルス: 転送開始のシグナル
2. 1バイト: ヘッダバイト
3. 15バイト: コマンドのパラメータ
4. 1bit: ストップビット(0)

ヘッダバイトは、`コマンド*8 + Length`で表されます。`Length`は、送信されるパケットの総数（最初のパケットを含む1～7）を示しています。 

コマンドのパラメーターが15バイト以上の場合は、次のパケットが送信されますのでご注意ください。

1. 1パルス: 転送開始のシグナル
2. 16バイト: コマンドのパラメータ
3. 1bit: ストップビット(0)

1つのコマンドはパケットを最大7つまで使えるので、コマンドのパラメータは最大で111バイト(15+16\*6)まで利用できます。最後のパケットの末尾に未使用のバイトがある場合も出てくると思いますが、未使用のバイトがあっても特に問題にはなりません。

各パケット転送の間には、60ms（4フレーム）の遅延を発生させる必要があります。

## 🎮 コマンド一覧

Code | Name     | Explanation
-----|----------|--------------
 $00 | PAL01    | Set SGB Palette 0 &amp; 1
 $01 | PAL23    | Set SGB Palette 2 &amp; 3
 $02 | PAL03    | Set SGB Palette 0 &amp; 3
 $03 | PAL12    | Set SGB Palette 1 &amp; 2
 $04 | ATTR_BLK | "Block" Area Designation Mode
 $05 | ATTR_LIN | "Line" Area Designation Mode
 $06 | ATTR_DIV | "Divide" Area Designation Mode
 $07 | ATTR_CHR | "1CHR" Area Designation Mode
 $08 | SOUND    | Sound On/Off
 $09 | SOU_TRN  | Transfer Sound PRG/DATA
 $0A | PAL_SET  | Set SGB Palette Indirect
 $0B | PAL_TRN  | Set System Color Palette Data
 $0C | ATRC_EN  | Enable/disable Attraction Mode
 $0D | TEST_EN  | Speed Function
 $0E | ICON_EN  | SGB Function
 $0F | DATA_SND | SUPER NES WRAM Transfer 1
 $10 | DATA_TRN | SUPER NES WRAM Transfer 2
 $11 | MLT_REQ  | Multiple Controllers Request
 $12 | JUMP     | Set SNES Program Counter
 $13 | CHR_TRN  | Transfer Character Font Data
 $14 | PCT_TRN  | Set Screen Data Color Data
 $15 | ATTR_TRN | Set Attribute from ATF
 $16 | ATTR_SET | Set Data to ATF
 $17 | MASK_EN  | Game Boy Window Mask
 $18 | OBJ_TRN  | Super NES OBJ Mode

