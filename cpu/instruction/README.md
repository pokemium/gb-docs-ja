# 命令セット

この記事では、[rgbasm](https://rgbds.gbdev.io/docs/v0.5.1/rgbasm.1)がサポートしているオペコードの一覧で、簡単な説明、エンコードに必要なバイト数、実行の完了までに要する1MHz（GBC倍速モードでは2MHz）でのCPUサイクル数が記載されています。

```
Note:

rgbasmでは、Aレジスタをターゲットとして使用するすべての算術/論理演算において、ターゲットはデフォルトでAレジスタであると想定されるため、ターゲットレジスタのオペランドを省略することができます。  
つまり、次の2行は同じ効果があります。

  OR A,B  
  OR B
```

## オペコードテーブル

[gbops](https://izik1.github.io/gbops/)を参照

## 各命令について

<pre>
Note: サイクル数は<a href="../cycle.md#マシンサイクル">マシンサイクル</a>単位です。
</pre>

### 記号の意味

```
r8:  8bitレジスタ(A, B, C, D, E, H, L)
r16: 16bitレジスタ(BC, DE, HL)
n8:  8bitの整数
n16: 16bitの整数
e8:  8bitのオフセット(-128~127)
u3:  3bitの符号なし整数(0~7)
cc:  条件
        - Z:  フラグZがセット
        - NZ: フラグZがクリア
        - C:  フラグCがセット
        - NC: フラグCがクリア
vec: RSTベクタ(0x00, 0x08, 0x10, 0x18, 0x20, 0x28, 0x30, 0x38)
```

### [8bit算術論理演算](./alu8.md)

- [ADC A,r8](alu8.md#adc-ar8)
- [ADC A,[HL]](alu8.md#adc-ahl)
- [ADC A,n8](alu8.md#adc-an8)
- [ADD A,r8](alu8.md#add-ar8)
- [ADD A,[HL]](alu8.md#add-ahl)
- [ADD A,n8](alu8.md#add-an8)
- [AND A,r8](alu8.md#and-ar8)
- [AND A,[HL]](alu8.md#and-ahl)
- [AND A,n8](alu8.md#and-an8)
- [CP A,r8](alu8.md#cp-ar8)
- [CP A,[HL]](alu8.md#cp-ahl)
- [CP A,n8](alu8.md#cp-an8)
- [DEC r8](alu8.md#dec-r8)
- [DEC [HL]](alu8.md#dec-hl)
- [INC r8](alu8.md#inc-r8)
- [INC [HL]](alu8.md#inc-hl)
- [OR A,r8](alu8.md#or-ar8)
- [OR A,[HL]](alu8.md#or-ahl)
- [OR A,n8](alu8.md#or-an8)
- [SBC A,r8](alu8.md#sbc-ar8)
- [SBC A,[HL]](alu8.md#sbc-ahl)
- [SBC A,n8](alu8.md#sbc-an8)
- [SUB A,r8](alu8.md#sub-ar8)
- [SUB A,[HL]](alu8.md#sub-ahl)
- [SUB A,n8](alu8.md#sub-an8)
- [XOR A,r8](alu8.md#xor-ar8)
- [XOR A,[HL]](alu8.md#xor-ahl)
- [XOR A,n8](alu8.md#xor-an8)

### [16bit算術演算](./arith16.md)

- [ADD HL,r16](arith16.md#add-hlr16)
- [INC r16](arith16.md#inc-r16)
- [DEC r16](arith16.md#dec-r16)

### [bit操作](./bit.md)

- [BIT u3,r8](bit.md#bit-u3r8)
- [BIT u3,[HL]](bit.md#bit-u3hl)
- [SET u3,r8](bit.md#set-u3r8)
- [SET u3,[HL]](bit.md#set-u3hl)
- [RES u3,r8](bit.md#res-u3r8)
- [RES u3,[HL]](bit.md#res-u3hl)
- [SWAP r8](bit.md#swap-r8)
- [SWAP [HL]](bit.md#swap-hl)

### [bitシフト](./shift.md)

- [RL r8](shift.md#rl-r8)
- [RL [HL]](shift.md#rl-hl)
- [RLA](shift.md#rla)
- [RLC r8](shift.md#rlc-r8)
- [RLC [HL]](shift.md#rlc-hl)
- [RLCA](shift.md#rlca)
- [RR r8](shift.md#rr-r8)
- [RR [HL]](shift.md#rr-hl)
- [RRA](shift.md#rra)
- [RRC r8](shift.md#rrc-r8)
- [RRC [HL]](shift.md#rrc-hl)
- [RRCA](shift.md#rrca)
- [SLA r8](shift.md#sla-r8)
- [SLA [HL]](shift.md#sla-hl)
- [SRA r8](shift.md#sra-r8)
- [SRA [HL]](shift.md#sra-hl)
- [SRL r8](shift.md#srl-r8)
- [SRL [HL]](shift.md#srl-hl)

### [ロード](./load.md)

- [LD r8,r8](load.md#ld-r8r8)
- [LD r8,n8](load.md#ld-r8n8)
- [LD r16,n16](load.md#ld-r16n16)
- [LD [HL],r8](load.md#ld-hlr8)
- [LD [HL],n8](load.md#ld-hln8)
- [LD r8,[HL]](load.md#ld-r8hl)
- [LD [r16],A](load.md#ld-r16a)
- [LD [n16],A](load.md#ld-n16a)
- [LDH [n16],A](load.md#ldh-n16a)
- [LDH [C],A](load.md#ldh-ca)
- [LD A,[r16]](load.md#ld-ar16)
- [LD A,[n16]](load.md#ld-an16)
- [LDH A,[n16]](load.md#ldh-an16)
- [LDH A,[C]](load.md#ldh-ac)
- [LD [HLI],A](load.md#ld-hlia)
- [LD [HLD],A](load.md#ld-hlda)
- [LD A,[HLI]](load.md#ld-ahli)
- [LD A,[HLD]](load.md#ld-ahld)

### [ジャンプ・コール](./jump.md)

- [JP n16](jump.md#jp-n16)
- [JP cc,n16](jump.md#jp-ccn16)
- [JP HL](jump.md#jp-hl)
- [JR e8](jump.md#jr-e8)
- [JR cc,e8](jump.md#jr-cce8)
- [CALL n16](jump.md#call-n16)
- [CALL cc,n16](jump.md#call-ccn16)
- [RET](jump.md#ret)
- [RET cc](jump.md#ret-cc)
- [RETI](jump.md#reti)
- [RST vec](jump.md#rst-vec)

### [スタック操作](./stack.md)

- [ADD HL,SP](stack.md#add-hlsp)
- [ADD SP,e8](stack.md#add-spe8)
- [INC SP](stack.md#inc-sp)
- [DEC SP](stack.md#dec-sp)
- [LD SP,n16](stack.md#ld-spn16)
- [LD [n16],SP](stack.md#ld-n16sp)
- [LD HL,SP+e8](stack.md#ld-hlspe8)
- [LD SP,HL](stack.md#ld-sphl)
- [PUSH AF](stack.md#push-af)
- [PUSH r16](stack.md#push-r16)
- [POP AF](stack.md#pop-af)
- [POP r16](stack.md#pop-r16)

### [その他](./misc.md)

- [CCF](misc.md#ccf)
- [CPL](misc.md#cpl)
- [DAA](misc.md#daa)
- [DI](misc.md#di)
- [EI](misc.md#ei)
- [HALT](misc.md#halt)
- [NOP](misc.md#nop)
- [SCF](misc.md#scf)
- [STOP](misc.md#stop)

