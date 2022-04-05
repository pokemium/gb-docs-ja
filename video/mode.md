# モードについて

[STAT](./register/stat.md)も合わせて読むことをお勧めします。

モード | 内容 | 期間(`1dot=1C-cycle`) | アクセス可能なビデオメモリ
----- | ----- | ----- | -----
2 | OAM Search | 80 dots (19 µs) | VRAM, CGBパレット
3 | Pixel Transfer | 168\~291 dots (40\~60 µs) スキャンライン上のスプライトの数によって変動 | None
0 | HBlank | 85~208 dots (20\~49 µs) モード3に費やした時間に依存 | VRAM, OAM, CGBパレット
1 | VBlank | 4560 dots (1087 µs, 10スキャンライン分の合計) | VRAM, OAM, CGBパレット

## モード2 - OAM Serach

このモードでは、XY座標が現在のスキャンラインと重なるOBJをOAMの中から検索します。

## モード3 - Pixel Transfer

[ピクセルFIFO](./pixel_fifo.md)を参照

## モード0 - HBlank

TODO

## モード1 - VBlank

TODO
