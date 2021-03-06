# タイルマップ

ゲームボーイでは、VRAM上のメモリエリア`0x9800..9BFF`と`0x9C00..9FFF`(両方とも1024バイト)に、2つの32x32のタイルマップが格納されています。

これらのマップのいずれかを使用して、背景やウィンドウを表示することができます。

## タイル番号(タイルID, タイルインデックス)

タイルマップは、1バイトごとにタイル番号(タイルID, タイルインデックスとも呼ばれる)というものを持っています。

このタイル番号とLCDC.4で指定するアドレッシングモードから、[タイルデータ](tiledata.md)領域のどのタイルを指しているかが一意に定まります。

1つのタイルは8x8ピクセルであり、各タイルマップは32x32タイルなので、各タイルマップは256x256ピクセルの画像データが取得できます。そのうち160x144ピクセルだけがLCDに表示されます。

## 背景マップ属性(CGBモードのみ)

CGBモードでは、32x32バイトの追加マップがVRAMのバンク1に格納されます。

各バイトは、VRAMのバンク0の対応するタイル番号マップエントリの属性を定義します。つまり、バンク1の0x9800はバンク0の0x9800のタイルの属性を定義します。

```
Bit 0-2  背景パレット番号            (0..7 => BGP0 から BGP7)
Bit 3    タイル VRAM バンク番号      (0 = バンク 0、 1 = バンク 1)
Bit 4    不使用
Bit 5    水平方向に反転              (1 = 反転)
Bit 6    垂直方向に反転              (1 = 反転)
Bit 7    背景 - OAM の優先度         (0 = OAMの優先度ビットを使用、 1 = 背景優先)
```

bit7がセットされると、OAMメモリの優先度ビットに関わらず、対応するBGタイルが全てのOBJよりも優先されます。

また、LCDCレジスタのBit0にはマスター優先度フラグがあり、これがクリアされると他のすべての優先度ビットよりも優先されます。
