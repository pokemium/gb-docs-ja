# MBC6

対応カートリッジ: 特殊

MBC6は、個別に切り替え可能な2つのROMバンク（\$4000と\$6000）とRAMバンク（\$A000と\$B000）、SRAM、8Mbit(1MB)の`Macronix MX29F008TC-14`フラッシュメモリチップを搭載した珍しいMBCです。

MBC6が使われているゲームは、[ネットでゲット ミニゲーム@100](https://tcrf.net/Net_de_Get:_Minigame_@100)というゲームだけです。

<img src="../../images/mbc6.jpeg" width="320px" alt="Net_de_Get:_Minigame_@100" />

このゲームは、[モバイルアダプタGB](https://ja.wikipedia.org/wiki/%E3%83%A2%E3%83%90%E3%82%A4%E3%83%AB%E3%82%A2%E3%83%80%E3%83%97%E3%82%BFGB)を使ってウェブに接続し、ミニゲームをローカルのフラッシュメモリにダウンロードします。

2つのROMバンク領域はそれぞれ同じROMを共有しており、それぞれ別のバンクを指定可能です。2つのRAMバンク領域も同じことが言えます。

## メモリ

### 0000-3FFF - ROMバンク 00 (R)

ROMの最初の16KiB

### 4000-5FFF - ROM/フラッシュバンクA 00-7F (フラッシュ:R/W, ROM:R)

8KiBの領域で、ROMまたはフラッシュメモリとして使われます。どちらの用途で使うかは[\$2800..2fff](#2800-2fff---romフラッシュバンクa選択レジスタ-w)で決定します。

ROMバンクBと同じROM領域を共有していますが、ROMバンクBと別々のバンクに切り替え可能です。

### 6000-7FFF - ROM/フラッシュバンクB 00-7F (フラッシュ:R/W, ROM:R)

8KiBの領域で、ROMまたはフラッシュメモリとして使われます。どちらの用途で使うかは[\$3800..3fff](#3800-3fff---romフラッシュバンクb選択レジスタ-w)で決定します。

ROMバンクAと同じROM領域を共有していますが、ROMバンクAと別々のバンクに切り替え可能です。

### A000-AFFF - RAMバンクA 00-07 (R/W)

4KiBのRAM領域です。

RAMバンクBと同じRAM領域を共有していますが、RAMバンクBと別々のバンクに切り替え可能です。

### B000-BFFF - RAMバンクB 00-07 (R/W)

4KiBのRAM領域です。

RAMバンクAと同じRAM領域を共有していますが、RAMバンクAと別々のバンクに切り替え可能です。

## レジスタ

### 0000-03FF - RAM有効フラグ (W)

MBC1のそれと同じで、`0Ah`の値は外部RAMへの読み書きを有効にします。値が`00h`の場合は、無効になります。

### 0400-07FF - RAMバンクA番号 (W)

RAMバンクA(`$A000..AFFF`)のバンク番号です。

### 0800-0BFF - RAMバンクB番号 (W)

RAMバンクB(`$B000..BFFF`)のバンク番号です。

### 0C00-0FFF - フラッシュメモリ有効フラグ (W)

フラッシュチップへのアクセスを有効または無効にします。最下位ビット(0=無効, 1=有効)のみが使用されます。

このフラグを変更するには、Flash Write Enableをアクティブにする必要があります。

### 1000 - Flash Write Enable (W)

Enable or disable write mode for the flash chip. Only the lowest bit (0 for disable, 1 for enable) is used. 

Note that this maps to the /WE pin on the flash chip, not whether or not writing to the bus is enabled; some flash commands (e.g. JEDEC ID query) still work with this off so long as Flash Enable is on.

### 2000-27FF - ROM/フラッシュバンクA番号 (W)

The number for the active bank in ROM/Flash Bank A.

### 2800-2FFF - ROM/フラッシュバンクA選択レジスタ (W)

Selects whether the ROM or the Flash is mapped into ROM/Flash Bank A. A value of 00 selects the ROM and 08 selects the flash.

### 3000-37FF - ROM/フラッシュバンクB番号 (W)

The number for the active bank in ROM/Flash Bank B.

### 3800-3FFF - ROM/フラッシュバンクB選択レジスタ (W)

Selects whether the ROM or the Flash is mapped into ROM/Flash Bank B. A value of 00 selects the ROM and 08 selects the flash.

## フラッシュコマンド

フラッシュチップは、ROMバンクAまたはROMバンクBのアドレス空間に直接マッピングされているため、標準的なフラッシュアクセスコマンドが使用されます。

To issue a command, you must write the value $AA to $5555 then $55 to $2AAA and, which are mapped as 2:5555/1:4AAA for bank A or 2:7555/1:6AAA for bank B followed by the command at either 2:5555/2:7555, or a relevant address, depending on the command.

The commands and access sequences are as follows, were X refers to either 4 or 6 and Y to 5 or 7, depending on the bank region:

```
------------- ------------- ------------- ------------- ------------- ------------- ------------------------------------------------
2:Y555=$AA    1:XAAA=$55    2:Y555=$80    2:Y555=$AA    1:XAAA=$55    ?:X000=$30    Erase sector\* (set 8 kB region to $FFs)
2:Y555=$AA    1:XAAA=$55    2:Y555=$80    2:Y555=$AA    1:XAAA=$55    ?:Y555=$10    Erase chip\* (set entire flash to $FFs)
2:Y555=$AA    1:XAAA=$55    2:Y555=$90                                                 ID mode (reads out JEDEC ID (C2,81) at $X000)
2:Y555=$AA    1:XAAA=$55    2:Y555=$A0                                                 Program mode\*
2:Y555=$AA    1:XAAA=$55    2:Y555=$F0                                                 Exit ID/erase chip mode
2:Y555=$AA    1:XAAA=$55    ?:X000=$F0                                                 Exit erase sector mode
?:????=$F0                                                                               Exit program mode
------------- ------------- ------------- ------------- ------------- ------------- ------------------------------------------------
```

Commands marked with * require the Write Enable bit to be 1. These will make the flash read out status bytes instead of values. A status of $80 means the operation has finished and you should exit the mode using the appropriate command. A status of $10 indicates a timeout.

Programming must be done by first erasing a sector, activating write mode, writing out 128 bytes (aligned), then writing a 0 to the final address to commit the write, waiting for the status to indicate completion, and writing $F0 to the final address again to exit program mode. If a sector is not erased first programming will not work properly. In some cases it will only allow the stored bytes to be anded instead of replaced; in others it just won’t work at all. The only way to set the bits back to 1 is to erase the sector entirely. It is recommended to check the flash to make sure all bytes were written properly and re-write (without erasing) the 128 byte block if some bits didn’t get set to 0 properly. After writing all blocks in a sector Flash Write Enable should be set to 0.

