# OAMメモリ破壊バグ

```
Note: 海外では一般に「OAM Corruption Bug」と呼ばれています。(Corruption=メモリ破壊)
```

ゲームボーイのハードウェアには、以下の命令の16ビットの内容（演算前）が0xFE00..FEFFの範囲にあり、かつPPUがモード2の状態で使用した場合、OAMにゴミが書き込まれるという不具合があります。

```asm
 inc rr         dec rr       ; rr = bc, de, or hl
 ld a, [hli]    ld a, [hld]
 ld [hli], a    ld [hld], a
```

スプライト1,2(`$FE00`, `$FE04`)はこのバグの影響を受けないようです。

またゲームボーイカラーとゲームボーイアドバンスの場合、モノクロのゲームソフトを使ったとしても、このバグは起きません。

## Accurate Description

OAMメモリ破壊バグは、正確には次の2種類の現象が組み合わさって生じるバグです。

- PPUがモード2（OAM Search）の時に、OAM（\$FEA0-\$FEFF領域を含む）から読み書きしようとすると、対象のメモリの内容が破損してしまいます。
- 16bitレジスタ（BC、DE、HL、SP、PC）がOAMの範囲（0xFE00..FEFF）にあるときに、レジスタの増減操作を行うと、OAMへのメモリ書き込みが発生し、破損の原因となります。

## Affected Operations

このバグの影響を受けるのは以下の処理たちです。

**OAMにアクセスする命令**

説明不要です。

**`inc rr`, `dec rr`**

もし`rr`がOAMを指している16bitレジスタであれば、書き込みのトリガーとなり、OAMを破壊してしまいます。

**`ld [hli], a`, `ld [hld], a`, `ld a, [hli]`, `ld a, [hld]`**

もしHLががOAMを指していれば、書き込みのトリガーとなり、OAMを破壊してしまいます。

**`pop rr`, `ret`系**

スタックからのpop(SPのデクリメント)を行うこれらの処理はOAMメモリ破壊バグを3回引き起こします。

バグを引き起こす3回の内訳は、読み出し、(不具合による)書き込み、読み出し です。

後述のスタックへのpushの場合と違ってバグは3回だけです。

**`push rr`, `call`系, `rst xx`, 割り込みハンドラ**

スタックへのpush(SPのインクリメント)を行うこれらの処理はOAMメモリ破壊バグを4回引き起こします。

バグを引き起こす4回の内訳は、書き込みx2、(不具合による)書き込みx2 です。

However, since one glitched write occurs in the same cycle as a actual write, this will effectively behave like 3 writes.

**OAM内のコードの実行**

If PC is inside OAM (reading $FF, that is, `rst $38`) the bug will trigger twice, once for increasing PC inside OAM (triggering a write), and once for reading from OAM. If a multi-byte opcode is executed from $FDFF or $FDFE, and bug will similarly trigger twice for every read from OAM.

## Corruption Patterns

The OAM is split into 20 rows of 8 bytes each, and during mode 2 the PPU
reads those rows consecutively; one every 1 M-cycle. The operations
patterns rely on type of operation (read/write/both) used on OAM during
that M-cycle, as well as the row currently accessed by the PPU. The
actual read/write address used, or the written value have no effect.
Additionally, keep in mind that OAM uses a 16-bit data bus, so all
operations are on 16-bit words.

### Write Corruption

A write corruption corrupts the currently access row in the following
manner, as long as it's not the first row (containing the first two
sprites):

-   The first word in the row is replaced with this bitwise expression:
    `((a ^ c) & (b ^ c)) ^ c`, where `a` is the original value of that
    word, `b` is the first word in the preceding row, and `c` is the
    third word in the preceding row.
-   The last three words are copied from the last three words in the
    preceding row.

### Read Corruption

A read corruption works similarly to a write corruption, except the
bitwise expression is `b | (a & c)`.

### Write During Increase/Decrease

If a register is increased or decreased in the same M-cycle of a write,
this will effectively trigger two writes in a single M-cycle. However,
this case behaves just like a single write.

### Read During Increase/Decrease

If a register is increased or decreased in the same M-cycle of a write,
this will effectively trigger both a read **and** a write in a single
M-cycle, resulting in a more complex corruption pattern:

-   This corruption will not happen if the accessed row is one of the
    first four, as well as if it's the last row:
    -   The first word in the row preceding the currently accessed row
        is replaced with the following bitwise expression:
        `(b & (a | c | d)) | (a & c & d)` where `a` is the first word
        two rows before the currently accessed row, `b` is the first
        word in the preceding row (the word being corrupted), `c` is the
        first word in the currently accessed row, and `d` is the third
        word in the preceding row.
    -   The contents of the preceding row is copied (after the
        corruption of the first word in it) both to the currently
        accessed row and to two rows before the currently accessed row
-   Regardless of wether the previous corruption occurred or not, a
    normal read corruption is then applied.
