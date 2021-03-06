# bit操作

<pre>
Note: サイクル数は<a href="../cycle.md#マシンサイクル">マシンサイクル</a>単位です。
</pre>

## BIT u3,r8

<pre>
r8の下位u3目のbitをチェックする(u3=0~7)  
bitが1ならZフラグをクリア、0ならZフラグをセットする

サイクル: 2
バイト長: 2
フラグ:
    Z: bitが0ならセット
    N: 0
    H: 1
    C: 不変
</pre>

## BIT u3,[HL]

<pre>
HLが指すメモリアドレスの値の下位u3目のbitをチェックする(u3=0~7)  
bitが1ならZフラグをクリア、0ならZフラグをセットする

サイクル: 3
バイト長: 2
フラグ:
    Z: bitが0ならセット
    N: 0
    H: 1
    C: 不変
</pre>

## SET u3,r8

<pre>
r8の下位u3目のbitをセットする(u3=0~7)  

サイクル: 2
バイト長: 2
フラグ: 不変
</pre>

## SET u3,[HL]

<pre>
HLが指すメモリアドレスの値の下位u3目のbitをセットする(u3=0~7)  

サイクル: 4
バイト長: 2
フラグ: 不変
</pre>

## RES u3,r8

<pre>
r8の下位u3目のbitをクリアする(u3=0~7)  

サイクル: 2
バイト長: 2
フラグ: 不変
</pre>

## RES u3,[HL]

<pre>
HLが指すメモリアドレスの値の下位u3目のbitをクリアする(u3=0~7)  

サイクル: 4
バイト長: 2
フラグ: 不変
</pre>

## SWAP r8

<pre>
r8の上位4bit(bit4-7)と下位4bit(bit0-3)を入れ替える

サイクル: 2
バイト長: 2
フラグ:
    Z: 入れ替えた結果の値が0ならセット
    N: 不変
    H: 不変
    C: 不変
</pre>

## SWAP [HL]

<pre>
HLが指すメモリアドレスの値について、上位4bit(bit4-7)と下位4bit(bit0-3)を入れ替える

サイクル: 4
バイト長: 2
フラグ:
    Z: 入れ替えた結果の値が0ならセット
    N: 不変
    H: 不変
    C: 不変
</pre>
