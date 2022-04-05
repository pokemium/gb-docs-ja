# 🗺 メモリマップ

アドレス | サイズ | 名称 | 用途
---- | ---- | ---- | ----
0000..3FFF | 16KB | ROM Bank00 | バンク0で固定
4000..7FFF | 16KB | ROM Bank01~NN | MBCによってはバンクを1以外に切り替え可能
8000..9FFF | 8KB | VRAM | GBCではバンク0/1で切り替え可能
A000..BFFF | 8KB | RAM Bank00~NN | バンク切り替え可能な場合も存在, SRAMとも。
C000..CFFF | 4KB | WRAM(Bank00) | -- 
D000..DFFF | 4KB | WRAM(Bank01~NN) | GBCではバンク切り替え可能(バンク数はカートリッジに依存)
E000..FDFF | -- | Echo RAM | c000..ddff のミラー 任天堂はこのアドレスを使うことを禁止しています。
FE00..FE9F | 160B | OAM | -- 
FEA0..FEFF | -- | 不使用 | 任天堂はこのアドレスを使うことを禁止しています。
FF00..FF7F | 128B | IOレジスタ | -- 
FF80..FFFE | 127B | HRAM(High RAM) | --
FFFF..FFFF | 1B | IME | -- 

## 🗄 ベクタ

次のアドレスらは特定の処理が起きた時のジャンプ先のベクタとして使われています。

- RST命令: `0000, 0008, 0010, 0018, 0020, 0028, 0030, 0038`
- 割り込み: `0040, 0048, 0050, 0058, 0060`

ただし、RST命令や割り込みを使用しない場合は，このメモリ領域（`0x0000..00FF`）を他の目的に使用することができます。

RSTは1バイトの命令で，宛先アドレスが制限されていることを除けば，3バイトのCALL命令と同様に動作します。命令長が1バイトであるため、3バイトのCALL命令に比べて、若干の高速化も実現しています。

## 🏷 カートリッジヘッダ

`0x0100..014F`のメモリエリアには、カートリッジのヘッダが入っています。

このエリアには、プログラムの情報、エントリーポイント、チェックサム、使用するMBCチップの情報、ROMとRAMのサイズなどが含まれています。このエリアのほとんどのバイトは正しく指定する必要があります。

## 💽 外部メモリとハードウェア

`0x0000..7FFF`および`0xA000..BFFF`のエリアは、カートリッジ(つまり外部ハードウェア)に存在します。

一般的には、ROMやSRAM、あるいはMBC（Memory Bank Controller）などがこれにあたります。

RAMエリアは通常の読み書きが可能で、ROMエリアへの書き込みはMBCを制御します。MBCの中には、このようにして他のハードウェアをRAM領域にマッピングできるものもあります。

カートリッジRAMは、ゲームボーイの電源を切ったときに、ゲームデータを保持するために、バッテリーバッファされている、つまりカートリッジ内蔵のバッテリーを使ってメモリのデータを保持し続けていることが多いです。具体的な情報は、[メモリバンクコントローラ](./cartridge/mbc.md)をお読みください。

## 🔊 Echo RAM

The range E000-FDFF is mapped to WRAM, but only the lower 13 bits of the address lines are connected, with the upper bits on the upper bank set internally in the memory controller by a bank swap register. This causes the address to effectively wrap around. All reads and writes to this range have the same effect as reads and writes to C000-DDFF.

Nintendo prohibits developers from using this memory range. The behavior is confirmed on all official hardware. Some emulators (such as VisualBoyAdvance <1.8) don’t emulate Echo RAM. In some flash cartridges, echo RAM interferes with SRAM normally at A000-BFFF. Software can check if Echo RAM is properly emulated by writing to RAM (avoid values 00 and FF) and checking if said value is mirrored in Echo RAM and not cart SRAM.

## ⌨️ I/O領域

ゲームボーイのI/Oレジスタは次のようになっています。

Start     | End       | First appeared | 用途
----------|-----------|----------------|-------------
`$FF00`   |           |       DMG      | ジョイパッド
`$FF01`   |  `$FF02`  |       DMG      | 通信
`$FF04`   |  `$FF07`  |       DMG      | タイマー
`$FF10`   |  `$FF26`  |       DMG      | サウンド
`$FF30`   |  `$FF3F`  |       DMG      | 波形メモリ
`$FF40`   |  `$FF4B`  |       DMG      | LCD
`$FF4F`   |           |       CGB      | VRAMバンク切り替え
`$FF50`   |           |       DMG      | ブートROMを無効化したい場合に非ゼロ値にセット
`$FF51`   |  `$FF55`  |       CGB      | [VRAM DMA](<#LCD VRAM DMA Transfers>)
`$FF68`   |  `$FF69`  |       CGB      | [BG / OBJ Palettes](<#LCD Color Palettes (CGB only)>)
`$FF70`   |           |       CGB      | WRAMバンク切り替え

## `FEA0..FEFF` について

任天堂はこのアドレス領域を使うことを禁止しています。

この領域は、OAMがブロックされている場合は`0xFF`を返し、それ以外の場合はハードウェアに応じた動作をします。

- On DMG, MGB, SGB, and SGB2, reads during OAM block trigger OAM corruption. Reads otherwise return $00.
- On CGB revisions 0-D, this area is a unique RAM area, but is masked with a revision-specific value.
- On CGB revision E, AGB, AGS, and GBP, it returns the high nibble of the lower address byte twice, e.g. FFAx returns $AA, FFBx returns $BB, and so forth.

