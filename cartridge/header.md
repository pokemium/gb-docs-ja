# カートリッジヘッダ

<img src="../images/img_03.png" alt="cart" title="https://www.pokemon.co.jp/ex/VCAMAP/dlcard/ より引用" width="240px" ></img>

## 0100-0103 - エントリーポイント

任天堂ロゴが表示された後、内蔵のブート手順ではこのアドレス（`$0100`）にジャンプし、カートリッジ内の実際のメインプログラムにジャンプするはずです。

通常、この4バイトの領域には、`nop`命令と、それに続く`jp $0150`命令が含まれています。(必ずしもそうではありませんが)

## 0104-0133 - 任天堂のロゴ

これらのバイトは、ゲームボーイの電源を入れたときに表示される任天堂ロゴのビットマップを定義します。このビットマップの16進数のダンプは次のようになります。

```
 CE ED 66 66 CC 0D 00 0B 03 73 00 83 00 0C 00 0D
 00 08 11 1F 88 89 00 0E DC CC 6E E6 DD DD D9 99
 BB BB 67 63 6E 0E EC CC DD DC 99 9F BB B9 33 3E
```

ゲームボーイのブート手順では、（表示後に）このビットマップの内容を検証し、このバイトが正しくない場合には動作を停止します。

CGBでは、ビットマップの前半部分（18バイト）のみを検証しますが、ゲームボーイポケットなどでは、ゲームボーイアドバンス同様、30バイトすべてを検証するものもあります。

## 0134-0143 - タイトル

大文字のASCII文字列によるゲームのタイトルです。

16文字に満たない場合は、残りのバイトは`$00`バイトで埋められます。

任天堂はCGBを開発する際に、この領域の長さを15文字にしましたが、その数ヶ月後には11文字にするという素晴らしいアイデアを出しました。

新しいゲームでは後ろの4バイト部分(`0x013f..0143`)は以下のように使われています。

## 013F-0142 - 製造元コード

古いカートリッジでは、この領域はタイトルの一部となっていますが、新しいカートリッジでは、このエリアに4文字で大文字のASCII文字列による製造元コードが入っています。目的や深い意味は不明です。

## 0143 - CGBフラグ

（上述のとおり）古いカートリッジでは、このバイトはタイトルの一部となっています。

CGBカートリッジの場合、上位ビットはCGB機能を有効にするために使用されます。これがないと、CGBは自動的にNon-CGB-Modeに切り替わります。典型的な値は以下の通りです。

- `$80`: ゲームはCGBの機能に対応していますが、古いゲームボーイ(DMG)でも動作します。
- `$c0`: CGBのみで動作するゲームです。

ビット7を設定し、ビット2またはビット3を設定すると、ゲームボーイは「PGBモード」(調査中)と呼ばれる特殊な非CGBモードに切り替わります。

## 0144-0145 - ライセンスコード(新)

ゲームの開発元または販売元を示す2文字のASCIIライセンスコードを指定します。

この2バイトは、新しいゲーム（SGBが発明された後に発売されたゲーム）でのみ使用されます。古いゲームは、代わりに`$014B`のヘッダエントリを使用しています。

Code | Publisher
-----|-----------
`00` | None
`01` | Nintendo R&D1
`08` | Capcom
`13` | Electronic Arts
`18` | Hudson Soft
`19` | b-ai
`20` | kss
`22` | pow
`24` | PCM Complete
`25` | san-x
`28` | Kemco Japan
`29` | seta
`30` | Viacom
`31` | Nintendo
`32` | Bandai
`33` | Ocean/Acclaim
`34` | Konami
`35` | Hector
`37` | Taito
`38` | Hudson
`39` | Banpresto
`41` | Ubi Soft
`42` | Atlus
`44` | Malibu
`46` | angel
`47` | Bullet-Proof
`49` | irem
`50` | Absolute
`51` | Acclaim
`52` | Activision
`53` | American sammy
`54` | Konami
`55` | Hi tech entertainment
`56` | LJN
`57` | Matchbox
`58` | Mattel
`59` | Milton Bradley
`60` | Titus
`61` | Virgin
`64` | LucasArts
`67` | Ocean
`69` | Electronic Arts
`70` | Infogrames
`71` | Interplay
`72` | Broderbund
`73` | sculptured
`75` | sci
`78` | THQ
`79` | Accolade
`80` | misawa
`83` | lozc
`86` | Tokuma Shoten Intermedia
`87` | Tsukuda Original
`91` | Chunsoft
`92` | Video system
`93` | Ocean/Acclaim
`95` | Varie
`96` | Yonezawa/s'pal
`97` | Kaneko
`99` | Pack in soft
`A4` | Konami (Yu-Gi-Oh!)

## 0146 - SGBフラグ

ゲームがSGB機能をサポートしているかどうかを示すフラグです。よく使われるのは次の値です。

- `$00`: SGB機能なし(通常のDMG/CGBのゲームとして動作)
- `$03`: SGB機能をサポートしています

このバイトが `$03` 以外の値に設定されている場合、SGBはそのSGB機能を無効にします。

## 0147 - カートリッジタイプ

カートリッジで使用されているメモリバンクコントローラと、カートリッジにさらに外部ハードウェアが存在するかどうかを指定します。

Code | Type
-----|------
 $00 | ROM ONLY
 $01 | MBC1
 $02 | MBC1+RAM
 $03 | MBC1+RAM+BATTERY
 $05 | MBC2
 $06 | MBC2+BATTERY
 $08 | ROM+RAM <sup>[1](#rom_ram)</sup>
 $09 | ROM+RAM+BATTERY <sup>[1](#rom_ram)</sup>
 $0B | MMM01
 $0C | MMM01+RAM
 $0D | MMM01+RAM+BATTERY
 $0F | MBC3+TIMER+BATTERY
 $10 | MBC3+TIMER+RAM+BATTERY <sup>[2](#mbc30)</sup>
 $11 | MBC3
 $12 | MBC3+RAM <sup>[2](#mbc30)</sup>
 $13 | MBC3+RAM+BATTERY <sup>[2](#mbc30)</sup>
 $19 | MBC5
 $1A | MBC5+RAM
 $1B | MBC5+RAM+BATTERY
 $1C | MBC5+RUMBLE
 $1D | MBC5+RUMBLE+RAM
 $1E | MBC5+RUMBLE+RAM+BATTERY
 $20 | MBC6
 $22 | MBC7+SENSOR+RUMBLE+RAM+BATTERY
 $FC | POCKET CAMERA
 $FD | BANDAI TAMA5
 $FE | HuC3
 $FF | HuC1+RAM+BATTERY

<sup id="rom_ram">1: このオプションを使用しているライセンスカートリッジはありません。正確な動作は不明です。</sup>

<sup id="mbc30">2: RAMサイズが64KByteのMBC3とは、ポケモンクリスタルの日本ROMでのみ使用されたMBC30のことです。</sup>

## 0148 - ROMサイズ

カートリッジのROMサイズを示す領域です。基本的に値が`N`ならROMサイズは`32 KiB << N`になります。

Code | Size      | Amount of banks
-----|-----------|-----------------
 $00 |  32 KB | 2 (ROMバンク無し)
 $01 |  64 KB | 4
 $02 | 128 KB | 8
 $03 | 256 KB | 16
 $04 | 512 KB | 32
 $05 |   1 MB | 64
 $06 |   2 MB | 128
 $07 |   4 MB | 256
 $08 |   8 MB | 512
 $52 | 1.1 MB | 72 <sup>[3](#weird_rom_sizes)</sup>
 $53 | 1.2 MB | 80 <sup>[3](#weird_rom_sizes)</sup>
 $54 | 1.5 MB | 96 <sup>[3](#weird_rom_sizes)</sup>

<sup id="weird_rom_sizes">3: 非公式のドキュメントにのみ記載されています。これらのサイズを使用したカートリッジやROMファイルは知られていません。他のROMサイズはすべて2の累乗なので、これらの値は正確ではないと思われます。これらの値の出典は不明です。</sup>

## 0149 - RAMサイズ

カートリッジのRAMサイズを示す領域です。

Code | Size   | Comment
-----|--------|---------
 $00 | 0      | RAM無し <sup>[4](#mbc2)</sup>
 $01 | -      | 不使用 <sup>[5](#2kib_sram)</sup>
 $02 | 8 KB   | 1バンク
 $03 | 32 KB  | 4バンク
 $04 | 128 KB | 16バンク
 $05 | 64 KB  | 8バンク

 Various "PD" ROMs ("Public Domain" homebrew ROMs generally tagged "(PD)" in the filename) are known to use the $01 RAM Size tag, but this is believed
to have been a mistake with early homebrew tools and the PD ROMs often don't use cartridge RAM at all.

<sup id="mbc2">4: MBC2チップを使用する場合、MBC2には 512×4 bit のRAMが内蔵されていますが、RAM Sizeとして$00を指定する必要があります。</sup>

<sup id="2kib_sram">5: 様々な非公式ドキュメントには2KBと記載されています。しかし、2KBのRAMチップがカートリッジに使われたことはありません。これらの値の出典は不明です。</sup>

## 014A - 販売先コード

このバージョンのゲームが日本で販売されることになっているのか、それ以外の場所で販売されることになっているのかを指定します。2つの値のみが定義されています。

- `$00`: Japanese
- `$01`: Non-Japanese

## 014B - ライセンスコード(旧)

ゲーム開発元/販売元コードを`$00-FF`の範囲で指定します。`$33`を指定すると、新しいほうのライセンスコード（ヘッダの`$0144..0145`）が使用されます。(SGB機能は$33以外の値では無効になります)

ライセンスコードの一覧は[こちら](https://raw.githubusercontent.com/gb-archive/salvage/master/txt-files/gbrom.txt)をみてください。

## 014C - バージョン番号

ゲームのバージョン番号を指定します。たいていのゲームで`$00`です。

## 014D - ヘッダチェックサム

カートリッジのヘッダバイト(`$0134..014C`)の8bitチェックサムが含まれています。

ブートROMは`x`を次のように計算します。

```
x = 0
i = $0134
while i <= $014C
	x = x - [i] - 1
```

`$014D`のバイトが`x`の下位8bitと一致しない場合、ブートROMが停止してしまい、カートリッジプログラムが実行されません。

## 014E-014F - グローバルチェックサム

カートリッジROM全体の16bitチェックサム（上位バイトから順に）が含まれています。

カートリッジの全バイト（チェックサムの2バイトを除く）を加算して生成されます。ゲームボーイはこのチェックサムを検証しません。
