# 16bit算術演算

<pre>
Note: サイクル数は<a href="../cycle.md#マシンサイクル">マシンサイクル</a>単位です。
</pre>

## ADD HL,r16

<pre>
r16の値とHLレジスタの値を足す

サイクル: 2
バイト長: 1
フラグ:
    Z: 不変
    N: 0
    H: bit11からオーバーフローした場合にセット
    C: bit15からオーバーフローした場合にセット
</pre>

## INC r16

<pre>
r16の値をインクリメント(+1)する  
処理結果はr16に格納される

サイクル: 2
バイト長: 1
フラグ: 不変
</pre>

## DEC r16

<pre>
r16の値をデクリメント(-1)する  
処理結果はr16に格納される

サイクル: 2
バイト長: 1
フラグ: 不変
</pre>
