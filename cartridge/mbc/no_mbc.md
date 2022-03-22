# MBCなし

対応カートリッジ: 32KiB ROMのみ

ROMが32KiB以下の小型ゲームでは、ROMバンク用のMBCチップは必要ありません。

ROMは`$0000..7FFF`のメモリに直接マッピングされます。オプションで、最大8KiBのRAMを`$A000..BFFF`に接続することができます。この場合、完全なMBCチップの代わりに`discrete logic decoder`を使用します。

