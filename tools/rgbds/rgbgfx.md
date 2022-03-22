# rgbgfx

```
Note: このドキュメントは[RGBGFX(1)](https://rednex.github.io/rgbds/rgbgfx.1.html)を日本語に翻訳・意訳したドキュメントです
```

png画像をGameBoyで利用できる[グラフィックフォーマット](./2bpp.md)にする変換ツールです

## 概要

rgbgfxはpng画像をGameBoyで利用できる[グラフィックフォーマット](./2bpp.md)にする変換ツールです

入力で与えたpng画像の色によって、変換の結果生じる色やパレット番号は様々なものになります
 - もしpngファイルがファイル内にパレットデータを持っているなら、そのパレットカラーと順番が利用されます
 - パレットデータを持っていないかつ画像がグレースケールなら、rgbgfxは画像に適切にパレット番号を割り当てます。未定のインデックスは、それぞれデフォルトの灰色の濃淡に設定されます。たとえば、ビット深度が2で、画像に明るい灰色と黒色が含まれている場合、それらは2番目と4番目の色になり、1番目と3番目の色はデフォルトの白色と暗い灰色に設定されます。 同じインデックスにマップする複数のシェードがイメージにある場合、代わりに、イメージにカラーがあるかのようにパレットが決定されます。
 - もし画像がグレースケールでない色を持った画像なら、色は 明->暗 の順にソートします

入力画像には、選択したビット深度が許可するよりも多くの色を含めることはできません。また透明ピクセルはパレット番号0に設定されます。

## 使い方

```sh
rgbgfx	[-CDhmuVv] [-f | -F] [-a attrmap | -A] [-d depth] [-o out_file] [-p pal_file | -P] [-t tilemap | -T] [-x tiles] file
```

### オプション一覧

オプションは一意に定まる範囲でなら省略が可能になっています  
例: --verboseは --verbまでなら省略が可能だが--verは--versionとの識別ができないので不可能

 ショート | ロング |  内容
 ----  | ---- | ----
 -a  | --attr-map | Generate a file of tile mirroring attributes for OAM or (CGB-only) background tiles. For each tile in the input file, a byte is written representing the dimensions that the associated tile in the output file should be mirrored. Useful in combination with -m to keep track the mirror direction of mirrored duplicate tiles.
 -A  | --output-attr-map | -aオプションと同じだが、入力ファイルの拡張子を`.attrmap`に変えたファイルをattrmapとして出力する
 -C  | --color-curve | パレットを生成する際にGameBoy Colorのcolor curveを利用する
 -D  | --debug | デバッグオプションを有効にする
 -d  | --depth | 出力データのビット深度 デフォルトでは2(2ビットで1pxを表す)
 -f  | --fix | 入力で与えられたpngファイルを正しくインデックス付けされた画像に修正する
 -F  | --fix-and-save | -fオプションと同じだが、PNGにコマンドライン引数を保存し、次に利用されるときそれをロードして利用するようにする
 -h  | --horizontal | タイルを垂直ではなく水平にレイアウトする
 -m  | --mirror-tiles | Truncate tiles by checking for tiles that are mirrored versions of others and omitting these from the output file. Useful with tilemaps and attrmaps together to keep track of the duplicated tiles and the dimension mirrored. Tiles are checked for horizontal, vertical, and horizontal-vertical mirroring. Implies -u.
 -o  | --output | 出力先のファイル名
 -p  | --palette | Output the image's palette in standard GBC palette format: bytes (8 bytes for two bits per pixel, 4 bytes for one bit per pixel) containing the RGB15 values in little-endian byte order. If the palette contains too few colors, the remaining entries are set to black.
 -P  | --output-palette | -pオプションと同じだが、入力ファイルの拡張子を`.pal`に変えたファイルをパレットファイルとして出力する
 -t  | --tilemap | Generate a file of tile indices. For each tile in the input file, a byte is written representing the index of the associated tile in the output file. Useful in combination with -u or -m to keep track of duplicate tiles.
 -T  | --output-tilemap | -tオプションと同じだが、入力ファイルの拡張子を`.tilemap`に変えたファイルをタイルマップファイルとして出力する
 -u  | --unique-tiles | 他のタイルとまったく同じですタイルをチェックし、出力ファイルからこれらを除外することにより、タイルを切り捨てる。 タイルマップと併用すると、どのタイルが複製されたかを追跡できる
 -V  | --version | rgbgfxのバージョンを出力
 -v  | --verbose | Verboseオプションを有効にする コマンドライン引数と入力pngファイルのパラメータが一致しないときにエラーを出力する
 -x  | --trim-end | このオプションで指定したタイル数で出力ファイルの末尾をトリミングする

## 例

```sh
# 次のコマンドはpngファイルをビット深度1か2か8で解釈して2bppフォーマットに変換する
rgbgfx -o out.2bpp in.png

# 次のコマンドは一意のタイルのみを含む平面2bppファイルを作成し、out.tilemapにタイルマップを出力する
rgbgfx -T -u -o out.2bpp in.png

# The following creates a planar 2bpp file with only unique tiles accounting for tile mirroring and its associated tilemap out.tilemap and attrmap out.attrmap:
rgbgfx -A -T -m -o out.2bpp in.png

# 次のコマンドは何もしない
rgbgfx in.png
```

## 参考

- [RGBGFX(1)](https://rednex.github.io/rgbds/rgbgfx.1.html)
- [PNGの規格を簡単に説明する](https://dawn.hateblo.jp/entry/2017/10/22/205417)

