# パレットコマンド

## SGBコマンド 00h - PAL01

SGBのパレット0の色0~3, パレット1の色1~3へと色データを送信するコマンドです。送られるパケットは1つだけです。

```
 Byte  Content
 0     0*8+1 (Command*8+Length)
 1-E   色データ(2バイト) x 7:
 F     不使用 (00h)
```

これは、[ゲームボーイカラーのパレット](../../video/register/palette.md#カラーパレットcgbモードのみ)と同じRGB5フォーマットですが、液晶補正はありません。色データ0として転送された値は、4つのパレットすべてに適用されます。

```
色データ
    Bit 0-4   - 赤    (0-31)
    Bit 5-9   - 緑    (0-31)
    Bit 10-14 - 青    (0-31)
    Bit 15    - 不使用 (zero)
```

## SGBコマンド 01h - PAL23

PAL01と内容は同じですが、対象がパレット2,3になりました。

## SGBコマンド 02h - PAL03

PAL01と内容は同じですが、対象がパレット0,3になりました。

## SGBコマンド 03h - PAL12

PAL01と内容は同じですが、対象がパレット1,2になりました。

## SGBコマンド 0Ah - PAL_SET

[SGBシステムのカラーパレット](../palette.md#-システムカラーパレット)にあらかじめ設定されている仮想のパレットデータを、[実際のスーファミのパレット](../palette.md#-パレットの種類)にコピーするために使用します。送られるパケットは1つだけです。

```
 Byte  Content
 0     0x0A*8+1 (Command*8+Length)
 1-2   SGBのパレット0にセットするのシステムカラーパレットの番号(0-511)
 3-4   SGBのパレット1にセットするのシステムカラーパレットの番号(0-511)
 5-6   SGBのパレット2にセットするのシステムカラーパレットの番号(0-511)
 7-8   SGBのパレット3にセットするのシステムカラーパレットの番号(0-511)
 9     属性ファイル
         Bit 0-5 - 属性ファイル番号 (00h-2Ch) (Used only if Bit7=1)
         Bit 6   - Cancel Mask           (0=No change, 1=Yes)
         Bit 7   - Use Attribute File    (0=No, 1=Apply above ATF Number)
 A-F   不使用 (zero)
```

パレット番号はリトルエンディアンで表します。

このコマンドを使う前に、PAL_TRNコマンドでシステムパレットのデータを初期化し、ATTR_TRNコマンドで属性ファイルのデータを初期化しておく必要があります。

## SGBコマンド 0Bh - PAL_TRN

スーファミのRAM内のシステムカラーパレットの初期化に使用されます。送られるパケットは1つだけです。

[PAL_SET](#sgbコマンド-0ah---pal_set)コマンドを使えば、この仮想のパレットのうち4個を実際のSGBのハードウェアパレットに転送することができます。

また、OBJ_TRNコマンドでは、4つのシステムカラーパレット（4x4色）のグループを、SNESのOBJパレット（16色）に使用します。

```
 Byte  Content
 0     0x0B*8+1 (Command*8+Length)
 1-F   不使用 (zero)
```

パレットデータは[VRAM転送（4KB）](../vram_transfer.md)で送られます。

```
000-FFF  Data for System Color Palette 0-511
```

Each Palette consists of four 16-bit color definitions (8 bytes). 

Note: The data is stored at 3000h-3FFFh in SNES memory.
