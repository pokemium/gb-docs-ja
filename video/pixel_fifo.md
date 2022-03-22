# ピクセルFIFO

ピクセルFIFOとは文字通り、ピクセルを保持するFIFOのことです。これはSTATのモード3(Pixel Transfer)で転送に使われるFIFOです。

```
Note:

このページでのサイクルとは、クロックサイクル(4.19MHz)のことで、倍速モードのCGBではサイクル数が2倍になります。  
ある動作によってモード3が長くなるというのは、次の図のように、モード0（HBlank）が短くなって、モード3の追加時間を補うことを意味します。
```

![](../images/ppu_modes_timing.svg)

## 前置き

ゲームボーイでは、このピクセルFIFOを2つ持っています。1つは背景ピクセル用(背景FIFO)、もう1つはOAMピクセル用(スプライトFIFO)です。

この2つのFIFOの内容は共有されません。お互いに独立しています。

2つのFIFOのデータが混在するのは、データをポップするときだけです。スプライトは、透明（色番号0）でない限り、背景よりも優先されますが、これについては後で詳しく説明します。

各FIFOには最大で16個のピクセルを格納できます。FIFOとピクセルフェッチャー(後述)は連携して、FIFOには常に8ピクセル以上のピクセルが格納されるようになっています。各FIFOは、モード3（Pixel Transfer）のときのみ操作されます。

FIFO内のピクセルは各々が次の4つのプロパティを保持しています。

- 色番号: 0~3の値となっています。
- パレット: CGBでは0～7の値を取ります。DMGではスプライトにのみ適用されます。
- OBJ優先度: CGBではこれがスプライトのOAMインデックスとなります。DMGではこのプロパティは存在しません。
- BG優先度: OAMの3バイト目のbit7

## ピクセルフェッチャー

ピクセルフェッチャー(以後、フェッチャー)は、8個の、背景またはウィンドウピクセルの列(`8x1`)をフェッチし、それらをキューに入れて、スプライトピクセルと混合します。

フェッチャーの処理は5つのステップで構成されています。最初の4つのステップはそれぞれ2サイクルかかり、5つ目のステップは成功するまで毎サイクル試行されます。ステップの順番は以下の通りです。

1. タイルの取得
2. 下位タイルデータの取得
3. 上位タイルデータの取得
4. スリープ
5. FIFOへのプッシュ

### タイルの取得

このステップでは、ピクセルを取得する背景/ウィンドウのタイルを決定します。デフォルトでは、\$9800のタイルマップが使用されますが、特定の条件で変更されることがあります。

LCDCのbit3がセットされていて、現在のスキャンラインのX座標がウィンドウ内にない場合、\$9C00のタイルマップが使用されます。また、LCDCのbit6がセットされていて、現在のスキャンラインのX座標がウィンドウ内にある場合は、\$9C00のタイルマップが使用されます。

フェッチャーは、自分(ピクセル)が乗っているタイルのXYの座標を、以下のように計算して記録しています。

まずX座標について。現在のタイルがウィンドウタイルの場合は、ウィンドウタイルのX座標が使用され、そうでない場合は、次の式でタイルのX座標が計算されます。

```go
// fetcherX: フェッチャーのいるタイルのX座標(タイル単位)
// fetcherXCoord: フェッチャーのX座標(ピクセル単位)
fetcherX = ((SCX / 8) + fetcherXCoord) & 0x1F
```

この計算式により、fetcherXは0〜31の間になります。

次にY座標について。現在のタイルがウィンドウタイルの場合は、ウィンドウタイルのY座標が使用され、そうでない場合は、次の式でタイルのY座標が計算されます。

```go
// fetcherY: フェッチャーのいるタイルのY座標(ピクセル単位)
fetcherY = (currentScanline + SCY) & 255
```

この計算式により、fetcherYは0～159の間になります。

フェッチャーが計算したタイルのX、Y座標を使って、VRAMのタイルマップからタイルIDを取得します。ただし、PPUのVRAMへのアクセスがブロックされている場合は、タイルIDの値は$FFとして読み込まれます。

CGBは、[タイルID](./tilemap.md#タイル番号タイルid-タイルインデックス)と[背景マップ属性](./tilemap.md#背景マップ属性cgbモードのみ)の両方にアクセスする必要がありますが、これは並行に行われます。

### 下位タイルデータの取得

LCDCのbit4で使用するタイルデータを確認します。(0なら`0x8800..97FF`、 1なら`0x8000..8FFF`)

CGBは、このステップでは、使用するVRAMバンクを確認し、タイルが垂直方向に反転しているかどうかを確認する必要があります。

タイルマップ、VRAM、垂直反転のデータが揃うと、アドレスが一意に定まるため、VRAMからタイルデータが取り出されます。しかし、PPUのVRAMへのアクセスがブロックされている場合、タイルデータは\$FFとして読み込まれます。

このステップで取得したタイルデータは、[プッシュ](#fifoへのプッシュ)のステップで使用されます。

### 上位タイルデータの取得

タイルデータのアドレスとして1だけインクリメントされたものを使う以外は、[下位タイルデータの取得](#下位タイルデータの取得)と同じです。

下位タイルデータ同様、このステップで取得したタイルデータは、[プッシュ](#fifoへのプッシュ)のステップで使用されます。

This also pushes a row of background/window pixels to the FIFO. This extra push is not part of the 8 steps, meaning there’s 3 total chances to push pixels to the background FIFO every time the complete fetcher steps are performed.

### FIFOへのプッシュ

背景/ウィンドウのピクセルの列(`8x1 pixel`)をFIFOにプッシュします。タイルの横幅は8ピクセルなので、前のステップで計算したXとYの座標に基づいて、レンダリングするタイルを起点として横8ピクセルの列ができます。

ピクセルが背景FIFOにプッシュされるのは背景FIFOが空のときに限られます。

ここで、直近の2ステップ(下位/上位タイルデータの取得)で取得したタイルデータが役に立ちます。タイルが水平方向に反転しているかどうかによって、ピクセルの背景FIFOへのプッシュの方法が変わってきます。タイルが水平方向に反転している場合、ピクセルはLSBから順に押されます。それ以外の場合は、MSBが先に押し出されます。

### スリープ

なにもしません。

### VRAMアクセス

At various times during PPU operation read access to VRAM is blocked and the value read is $FF:

- LCD turning off
- At scanline 0 on CGB when not in double speed mode
- When switching from mode 3 to mode 0
- On CGB when searching OAM and index 37 is reached

At various times during PPU operation read access to VRAM is restored:

- At scanline 0 on DMG and CGB when in double speed mode
- On DMG when searching OAM and index 37 is reached
- After switching from mode 2 (oam search) to mode 3 (pixel transfer)

Note: These conditions are checked only when entering STOP mode and the PPU's access to VRAM is always restored upon leaving STOP mode.

## モード3の処理内容

前述のとおり、ピクセルFIFOはモード3の間のみ動作します。モード3の開始時には、背景FIFOとスプライトFIFOの両方がクリアされます。

### ウィンドウ

ウィンドウを描画すると、背景FIFOがクリアされ、フェッチャーが[ステップ1](#タイルの取得)にリセットされます。WXが0で`SCX&7 > 0`の場合、モード3の期間が1サイクル短縮されます。

ウィンドウのレンダリングが既に始まっている場合、スキャンラインの途中でWXを変更するとバグが発生します。ウィンドウのレンダリング開始後にWXの値が変化し、新たに設定されたWXにフェッチャーのX座標が到達すると、色値が0で優先順位が最も低いピクセルが背景FIFOに押し込まれます。

### スプライト

LCDCのbit1がセットされていて（CGBではこの条件は無視されます）、かつ現在のスキャンラインのX座標にスプライトがある場合、現在のスキャンライン上の各スプライトに対して以下の処理が行われます。これらの条件が満たされていない場合、スプライトの取得は中止されます。

この時点で、[フェッチャー](#ピクセルフェッチャー)は[ステップ5](#fifoへのプッシュ)になるか、または背景FIFOに値が入るまで、1ステップずつ進みます。

ここでフェッチャーを1ステップ進めるごとに、モード3が1サイクル長くなります。この処理は、フェッチャーが1ステップ進んだ後に中止することができます。

`SCX&7 > 0`で、現在のスキャンラインのX座標0にスプライトがある場合、モード3の期間が長くなります。

このペナルティによってモード3が何サイクル長くなるかは、SCXの下位3ビットの値によって決まります。

このペナルティが適用された後、スプライトのフェッチが中止されることがあります。なお、このペナルティが適用されるタイミングは確定していません。フェッチャーを待つ前に起こるかもしれませんし、フェッチャーを待った後に起こるかもしれません。さらなる調査が必要です。

X座標0でスプライトをチェックした後、フェッチャーを2ステップ進めます。1回目のステップではモード3が1サイクル長くなり、2回目のステップではモード3が3サイクル長くなります。各フェッチャーのステップ進行後には、スプライトのフェッチャーが中止されることがあります。

The lower address for the row of pixels of the target object tile is now
retrieved and lengthens mode 3 by 1 cycle. Once the address is retrieved
this is the last chance for sprite fetch abortion to occur. Exiting
object fetch lengthens mode 3 by 1 cycle. The upper address for the
target object tile is now retrieved and does not shorten mode 3.

At this point [VRAM Access](<#VRAM Access>) is checked for the lower and
upper addresses for the target object. Before any mixing is done, if the
OAM FIFO doesn't have at least 8 pixels in it then transparent pixels
with the lowest priority are pushed onto the OAM FIFO. Once this is done
each pixel of the target object row is checked. On CGB, horizontal flip
is checked here. If the target object pixel is not white and the pixel in
the OAM FIFO *is* white, or if the pixel in the OAM FIFO has higher
priority than the target object's pixel, then the pixel in the OAM FIFO
is replaced with the target object's properties.

Now it's time to [render a pixel](<#Pixel Rendering>)! The same process
described in Sprite Fetch Abortion is performed: a pixel is rendered and
the fetcher is advanced one step. This advancement lengthens mode 3 by 1
cycle if the X coordinate of the current scanline is not 160. If the X
coordinate is 160 the PPU stops processing sprites (because they won't be
visible).

Everything in this section is repeated for every sprite on the current
scanline unless it was decided that fetching should be aborted or the
X coordinate is 160.

### ピクセルの描画

ここでは、背景FIFOとスプライトFIFOが混在しています。条件に応じて、背景ピクセルとスプライトピクセルのどちらかが優先的に表示されます。

背景FIFOとスプライトFIFOにピクセルがある場合は、それぞれのピクセルがFIFOからポップされます。スプライトのピクセルが透明でなく、LCDCのbit1がセットされている場合、スプライトピクセルは、自身のBG優先度プロパティが、背景ピクセルのBG優先度プロパティと同じかそれ以上の優先度であれば使用されます。

背景FIFOに何もない場合や、現在のピクセルのX座標が160px以上の場合、ピクセルはLCDに押し出されません。

LCDCのbit0を無効にすると、DMGでは背景が無効になり、CGBでは背景ピクセルが優先されなくなります。このとき、背景ピクセルが無効になるDMGの場合、ピクセルの色番号は0になり、CGBの場合は、ピクセルの色番号は背景FIFOからポップされたピクセルの色になります。背景FIFOからポップされたピクセルの色番号が0以外で、そのピクセルに優先権がある場合、そのピクセルでのスプライトピクセルは破棄されます。

この時点で、DMGでは、色番号とBGPレジスタからピクセルの色データが取得され、LCDにプッシュされます。CGBでは、[パレットへのアクセス](#cgbパレットアクセス)がブロックされると、黒の色データがLCDにプッシュされます。

スプライトピクセルに優先権がある場合、色番号はスプライトFIFOからポップされたピクセルから取得されます。DMGでは、ピクセルの色データは、ピクセルのパレットプロパティに応じて、OBP1またはOBP0レジスタから取得されます。パレットのプロパティが1の場合はOBP1が、そうでない場合はOBP0が使用されます。そして、そのピクセルはLCDにプッシュされます。CGBでは、パレットへのアクセスがブロックされると、黒の色データがLCDにプッシュされます。

このようにしてピクセルの色が液晶(LCD)に描画されます。

### CGBパレットアクセス

PPUの動作中に、CGBパレットへのリードアクセスがブロックされ、画素のレンダリング時に黒の色のピクセルがLCDに押し出され描画されることがあります。

- LCD turning off
- First HBlank of the frame
- When searching OAM and index 37 is reached
- After switching from mode 2 (oam search) to mode 3 (pixel transfer)
- When entering HBlank (mode 0) and not in double speed mode, blocked 2 cycles later no matter what

PPUが動作している間に、CGBパレットへのリードアクセスが回復し、ピクセルをレンダリングする際に通常通りピクセルがLCDにプッシュされます。

- At the end of mode 2 (oam search)
- For only 2 cycles when entering HBlank (mode 0) and in double speed mode

```
Note: これらの条件は、STOPモードに入るときにのみチェックされ、STOPモードを抜けるときには、PPUのCGBパレットへのアクセスは常に回復します。
```

### スプライトフェッチャーの処理中止

PPUがOAMからスプライトをフェッチしているときにLCDCのbit1がクリアされると、スプライトのフェッチが中止されることがあります。

この処理中止は、前の命令が要したサイクル数と、PPUが処理するために残っている残余サイクル分だけ、モード3を長くします。

OAMフェッチが中止され、[ピクセルがレンダリング](#ピクセルの描画)されると、[フェッチャー](#ピクセルフェッチャー)が1ステップ進みます。

現在のピクセルのX座標が160でない場合、このステップの進行によりモード3が1サイクル長くなります。

現在のピクセルが160の場合、PPUはスプライトが見えなくなるので処理を停止します。
