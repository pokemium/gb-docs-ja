# その他

<pre>
Note: サイクル数は<a href="../cycle.md#マシンサイクル">マシンサイクル</a>単位です。
</pre>

## CCF

<pre>
キャリーフラグ(C)を反転させる

サイクル: 1
バイト長: 1
フラグ:
    Z: 不変
    N: 0
    H: 0
    C: 反転
</pre>

## CPL

<pre>
Aレジスタを反転させる  
A = ~A

サイクル: 1
バイト長: 1
フラグ:
    Z: 不変
    N: 1
    H: 1
    C: 不変
</pre>

## DAA

<pre>
Decimal Adjust Accumulator to get a correct BCD representation after an arithmetic instruction.

サイクル: 1
バイト長: 1
フラグ:
    Z: 結果が0ならセット
    N: 不変
    H: 0
    C: Set or reset depending on the operation.
</pre>

## DI

<pre>
IMEをクリアして、割り込みを無効にする

サイクル: 1
バイト長: 1
フラグ: 不変
</pre>

## EI

<pre>
IMEをセットして、割り込みを有効にする  
IMEがセットされるまで<a href="../../interrupt.md#ime---全割り込み有効フラグ-w">少々のラグ</a>があることに注意

サイクル: 1
バイト長: 1
フラグ: 不変
</pre>

## HALT

<pre>
割り込みが起きるまで、CPUを低電力モードにする
HALT時のIMEの値によって挙動が変わってくる

IMEがセットされている(1)場合:
    割り込みが処理されるまで、CPUは低電力モードになる
    割り込みハンドラは正常に実行され、HALTから復帰するとCPUは実行を再開する

IMEがクリアされている(0)場合:
    割り込みが保留中([IE] & [IF] != 0)かどうかで挙動が変わる
    保留中:
        CPUは低電力モードに移行せず、実行を継続する
        ただし、ハードウェアのバグにより、HALTの次のバイトが2回連続して読み込まれることに注意
    保留中でない:
        割り込みが保留中になると、すぐにCPUは実行を再開する
        割り込みハンドラが呼び出されないことを除けば、上記のIMEがセットされている場合と同じ挙動になる

サイクル: -
バイト長: 1
フラグ: 不変
</pre>

## NOP

<pre>
何もしない

サイクル: 1
バイト長: 1
フラグ: 不変
</pre>

## SCF

<pre>
キャリーフラグ(C)をセットする

サイクル: 1
バイト長: 1
フラグ:
    Z: 不変
    N: 0
    H: 0
    C: 1
</pre>

## STOP

<pre>
Enter CPU very low power mode. Also used to switch between double and normal speed CPU modes in GBC.

サイクル: -
バイト長: 2
フラグ: 不変
</pre>


