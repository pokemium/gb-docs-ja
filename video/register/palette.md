# 🎨 パレット

## モノクロパレット(非CGBモードのみ)

### FF47 - BGP - BGパレットデータ (R/W)

このレジスタは、背景タイルとウィンドウタイルの色番号に対してグレーシェードを割り当てます。

- bit0-1: 色番号0の色データ
- bit2-3: 色番号1の色データ
- bit4-5: 色番号2の色データ
- bit6-7: 色番号3の色データ

色データは

- 0: 白
- 1: 白灰
- 2: 黒灰
- 3: 黒

になっています。

例えば、2bppフォーマットでピクセルが2を示した場合、BGPのbit4-5(色番号2)を見ます。  
ここでbit4-5が3を示していたとするなら対応する色データは黒なのでこのピクセルの色は黒くなります。

### FF48 - OBP0 - OBJパレットデータ0 (R/W)

このレジスタは、このパレットを使用するOBJの色番号にグレーシェードを割り当てます。

OBP0とOBP1のパレットのどちらを使用するかはOAMの属性/フラグのbit4で切り替えられます。

スプライトの色番号0は透明を意味するため、下位2ビットが無視されることを除けば、BGPと同じように動作します。

### FF49 - OBP1 - OBJパレットデータ1 (R/W)

OBP0と内容は同じです。

OAMの属性/フラグのbit4が1の場合はこちらのパレットが使われます。

## カラーパレット(CGBモードのみ)

CGBでは64バイト(0x40バイト)のパレット用のメモリなるものが、BG用とOBJ用に合計2つ存在しますが、これはアドレスと紐づけられておらず、通常のメモリアクセスではアクセスできません。

そこで利用されるのがこれから説明されるレジスタたちです。

BG/OBJパレットインデックスは、パレットメモリのアドレスを指定します。(バンク番号に近い)

BG/OBJパレットデータは、パレットインデックスで指定したパレットメモリの内容がマッピングされます。このレジスタを読み書きすることでパレットメモリにアクセスすることができます。

### FF68 - BCPS/BGPI - BGパレットインデックス

このレジスタは、CGBにおいてBGパレットメモリのアドレスを指定するために利用します。

上で述べたようにBGパレットメモリは全部で64バイトで、1つ8バイトのパレットデータが8つ格納されています。(BGP0~BGP7)

```
0   BGP0
        - 色番号0の色データ(2バイト, 後述)
        - 色番号1
        - 色番号2
        - 色番号3
8   BGP1
        - 色番号0 
        - 色番号1
        - 色番号2
        - 色番号3
16  BGP2
        - 色番号0 
        - 色番号1
        - 色番号2
        - 色番号3
    ...
56  BGP7
        - 色番号0 
        - 色番号1
        - 色番号2
        - 色番号3
```

```
Bit 7     オートインクリメント  (0=無効, 1=書き込み後にインクリメント発生)
Bit 5-0   インデックス (00-3F)
```

BGパレットメモリのインデックスを指定した後は、後述のレジスタFF69を介して、BGパレットデータを読み書きすることができます。

オートインクリメント(bit7)がセットされている場合、FF69への書き込みごとにインデックスが自動的にインクリメントされます。FF69からの読み出し時にはオートインクリメントの効果はないので、その場合は手動でインデックスをインクリメントする必要があります。レンダリング中にFF69に書き込んでも、オートインクリメントは発生します。

### FF69 - BCPD/BGPD - BGパレットデータ

このレジスタは，レジスタ`$FF68`を介してアドレス指定されたCGBの背景パレットメモリへのデータの読み書きを可能にします。

各色は2バイトで定義されます。bit0-7が1バイト目、bit8-15が2バイト目です。

```
 Bit 0-4   赤   (00 - 1F)
 Bit 5-9   緑   (00 - 1F)
 Bit 10-14 青   (00 - 1F)
```

### FF6A - OCPS/OBPI - OBJパレットインデックス

BGパレットと内容は同じです。

### FF6B - OCPD/OBPD - OBJパレットデータ

BGパレットと内容は同じです。

ただし色0は透明を表すことには注意してください。

## RGB Translation by CGBs

![](../../images/VGA_versus_CGB.png)

When developing graphics on PCs, note that the RGB values will have different appearance on CGB displays as on VGA/HDMI monitors calibrated to sRGB color. Because the GBC is not lit, the highest intensity will produce Light Gray color rather than White. The intensities are not linear; the values 10h-1Fh will all appear very bright, while medium and darker colors are ranged at 00h-0Fh.

The CGB display’s pigments aren’t perfectly saturated. This means the colors mix quite oddly; increasing intensity of only one R,G,B color will also influence the other two R,G,B colors. For example, a color setting of 03EFh (Blue=0, Green=1Fh, Red=0Fh) will appear as Neon Green on VGA displays, but on the CGB it’ll produce a decently washed out Yellow. See the image above.

## RGB Translation by GBAs

Even though GBA is described to be compatible to CGB games, most CGB games are completely unplayable on older GBAs because most colors are invisible (black). Of course, colors such like Black and White will appear the same on both CGB and GBA, but medium intensities are arranged completely different. Intensities in range 00h..07h are invisible/black (unless eventually under best sunlight circumstances, and when gazing at the screen under obscure viewing angles), unfortunately, these intensities are regularly used by most existing CGB games for medium and darker colors.

Newer CGB games may avoid this effect by changing palette data when detecting GBA hardware (see how). Based on measurement of GBC and GBA palettes using the 144p Test Suite ROM, a fairly close approximation is GBA = GBC * 3/4 + 8h for each R,G,B intensity. The result isn’t quite perfect, and it may turn out that the color mixing is different also; anyways, it’d be still ways better than no conversion.

This problem with low brightness levels does not affect later GBA SP units and Game Boy Player. Thus ideally, the player should have control of this brightness correction.
