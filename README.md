# gb-docs-ja

<img src="images/1280px-Nintendo-Game-Boy-Color.png" height="420" />

GameBoy, GameBoyColorについて、技術的な詳細を日本語でまとめたものです。自分のメモの側面もあります。

突然消えたり非公開にする可能性もあるので心配な方はクローンしておくことをお勧めします。

## コンテンツ一覧

### GB

- [仕様](./spec.md)
- [ハードウェア一覧](./hardware/README.md)
- [メモリマップ](./memory.md)
- [割り込み](./interrupt.md)
- [タイマー](./timer.md)
- [シリアル通信](./serial.md)
- [ジョイパッド](./joypad.md)
- [サウンド](./sound.md)
- [外部端子](./connector.md)
- [電源ON時の処理](./powerup.md)
- [ゲームボーイカラー](./cgb/README.md)
  - [HDMA](./cgb/hdma.md)
  - [VRAMバンク](./cgb/vram_bank.md)
  - [WRAMバンク](./cgb/wram_bank.md)
  - [倍速モード](./cgb/double_speed.md)
  - [赤外線通信](./cgb/infrared.md)
- [スーパーゲームボーイ](./sgb/README.md)
  - [SGB機能を利用するには](./sgb/usage.md)
  - [VRAMの転送](./sgb/vram_transfer.md)
  - [カラーパレット](./sgb/palette.md)
  - [コマンド](./sgb/command/README.md)
    - [パレットコマンド](./sgb/command/palette.md)
    - [カラーアトリビュートコマンド](./sgb/command/color_attr.md)
    - [サウンドコマンド](./sgb/command/sound.md)

### CPU

- [命令セット](./cpu/instruction/README.md)
  - [8bit算術論理演算](./cpu/instruction/alu8.md)
  - [16bit算術演算](./cpu/instruction/arith16.md)
  - [bit操作](./cpu/instruction/bit.md)
  - [bitシフト](./cpu/instruction/shift.md)
  - [ロード](./cpu/instruction/load.md)
  - [ジャンプ・コール](./cpu/instruction/jump.md)
  - [スタック操作](./cpu/instruction/stack.md)
  - [その他](./cpu/instruction/misc.md)
- [レジスタとフラグ](./cpu/register.md)
- [クロック/マシンサイクル](./cpu/cycle.md)
- [Z80との比較](./cpu/z80.md)

### グラフィック

- [概要](./video/README.md)
- IOレジスタ
  - [LCDC](./video/register/lcdc.md)
  - [STAT](./video/register/stat.md)
  - [座標/スクロール関連](./video/register/scrolling.md)
  - [パレット関連](./video/register/palette.md)
- [OAM](./video/oam/README.md)
  - [DMA](./video/oam/dma.md)
- [VRAM](./video/vram.md)
  - [タイルデータ](./video/tiledata.md)
  - [タイルマップ](./video/tilemap.md)
- [ピクセルFIFO](./video/pixel_fifo.md)
- [VRAM・OAMへのアクセス](./video/access.md)

### カートリッジ

- [カートリッジヘッダ](./cartridge/header.md)
- [MBC](./cartridge/mbc/README.md)
  - [MBCなし](./cartridge/mbc/no_mbc.md)
  - [MBC1](./cartridge/mbc/mbc1.md)
  - [MBC2](./cartridge/mbc/mbc2.md)
  - [MBC3](./cartridge/mbc/mbc3.md)
  - [MBC5](./cartridge/mbc/mbc5.md)
  - [MBC6](./cartridge/mbc/mbc6.md)
  - [HuC1](./cartridge/mbc/huc1.md)

### ツール

- [rgbds](./tools/rgbds/README.md)
  - [rgbgfx](./tools/rgbds/rgbgfx.md)
- [GB Studio](./tools/gbstudio.md)
- [gbdk-2020](./tools/gbdk2020.md)

### その他

#### 自作ソフト開発

- [電池の消費を少なくするためには](./others/reducing_power_consumption.md)
- [pokegbについて](./others/pokegb/README.md)

#### エミュ開発

- [セーブデータ](./others/emudev/sav.md)

#### コードリーディング

- [mgba-emu/mgba](./others/mgba/)
- [Gekkio/mooneye-gb](./others/mooneye-gb/)
- [sysprog21/jitboy](./others/jitboy/)
- [LIJI32/SameBoy](./others/sameboy/)

#### 雑多

- [ゲームボーイカラーソフトの販売認可](./others/cgb_approval.md)
- [OAMメモリ破壊バグ](./others/oam_corruption_bug.md)

## 関連するレポジトリ

- [gba-docs-ja](https://github.com/pokemium/gba-docs-ja): GameBoy Advanceについて
- [nds-docs-ja](https://github.com/pokemium/nds-docs-ja): Nintendo DSについて
- [snes-docs-ja](https://github.com/pokemium/snes-docs-ja): スーパーファミコンについて

## 参考記事

- [pandocs](https://gbdev.io/pandocs/): 一番参考にしているソースです。大半がここの翻訳になっています。
- [rgbds](https://rgbds.gbdev.io/docs/): CPUの命令周りなど
- [GB Spec](https://w.atwiki.jp/gbspec/): 日本語訳が面倒くさいので間違ってなさそうなところだけコピペしています...
- [The Cutting Room Floor](https://tcrf.net/): さまざま
- [binji's dustbin](https://binji.github.io/)
