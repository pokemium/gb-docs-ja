# jitboy

この記事は[jitboy](https://github.com/sysprog21/jitboy/tree/124378c71b6772cfc819f68ae6d755b7cf9df6f8)のコードリーディングから得られたものをまとめたものです。

## GB構造体

ルート構造体は`gb_vm`

```c++
typedef struct {
    gb_state state;
    gb_memory memory;
    gb_block compiled_blocks[MAX_ROM_BANKS][0x4000];  // bank, start address
    gb_block highmem_blocks[0x80];
    gb_lcd lcd;
    gb_audio audio;
    bool draw_frame;
    unsigned next_frame_time;

    int frame_cnt;
    unsigned time_busy;
    unsigned last_time;

    int opt_level;
} gb_vm;

typedef struct {
    uint8_t *mem;
    uint8_t *ram_banks;
    const char *filename;
    char *savname;
    uint8_t max_ram_banks_num;
    int fd;
    enum {
        MBC_NONE = 0x00,
        MBC1 = 0x01,
        MBC1_RAM_BAT = 0x03,
        MBC2 = 0x05,
        ..
        MBC5_RAM_BAT = 0x1b
    } mbc;
    uint8_t mbc_mode, mbc_data;
    uint8_t current_rom_bank, current_ram_bank;
    bool rtc_access;
} gb_memory;

typedef struct {
    // memory
    gb_memory *mem;

    // register
    uint8_t a, b, c, d, e, h, l;
    bool f_subtract;
    uint16_t _sp;
    uint16_t pc;
    uint64_t flags;

    uint16_t last_pc;

    // keys
    gb_keys keys;

    // instruction count
    uint64_t inst_count;

    uint64_t ly_count;
    uint64_t tima_count;
    uint64_t div_count;

    uint64_t next_update;

    // interrupt timers etc
    bool ime;

    // cpu is in halt state
    uint32_t halt;
    uint8_t halt_arg;

    // flag to trace callstack
    enum {
        REASON_OTHER = 0,
        REASON_CALL = 1,
        REASON_RST = 2,
        REASON_INT = 4,
        REASON_RET = 8
    } trap_reason;
} gb_state;
```

## JIT

**compile**

[`gbz80.c`](https://github.com/sysprog21/jitboy/blob/124378c71b6772cfc819f68ae6d755b7cf9df6f8/src/gbz80.c)でコンパイルのための関数`compile`が定義されている

```c++
// `start_address`から始まるブロックを`gb_block`にコンパイルする
bool compile(gb_block *block, gb_memory *mem, uint16_t start_address, int opt_level) {
    GList *instructions = NULL; // GList: glibの提供するリンクリスト

    uint16_t i = start_address;
    for (;;) {
        gbz80_inst *inst = g_new(gbz80_inst, 1);

        uint8_t opcode = mem->mem[i];
        if (opcode != 0xcb) {
            *inst = inst_table[opcode]; // inst_tableは `gbz80_inst[]`
        } else {
            opcode = mem->mem[i + 1];
            *inst = cb_table[opcode];
        }

        // 残りのフィールドを埋める
        inst->args = mem->mem + i;
        inst->address = i;  // オペコードのGBアドレス

        // GBアドレスをインクリメント
        i += inst->bytes;

        if (inst->opcode == ERROR) {
            LOG_ERROR("Invalid Opcode! (%#x)\n", opcode);
            return false;
        }
        LOG_DEBUG("inst: %i @%#x\n", inst->opcode, inst->address);

        instructions = g_list_prepend(instructions, inst);

        if (inst->flags & INST_FLAG_ENDS_BLOCK)
            break;
    }

    instructions = g_list_reverse(instructions);

    if (!optimize_block(&instructions, opt_level))
        return false;

    if (!optimize_cc(instructions))
        return false;

    bool result = emit(block, instructions);

    g_list_free_full(instructions, g_free);

    return result;
}

typedef struct {
    enum {
        NOP,
        LD16,
        LD,
        INC16,
        INC,
        DEC16,
        ..
        BIT,
        RES,
        SET,
        JP_TARGET,
        JP_BWD,
        JP_FWD,
        ERROR,
    } opcode;
    enum {
        NONE,
        REG_A,
        REG_B,
        REG_C,
        ..
        BIT_6,
        BIT_7,
        TARGET_1,
        TARGET_2,
        WAIT_LY,
        WAIT_STAT3
    } op1, op2;
    uint8_t *args;
    uint16_t address;
    uint8_t bytes;
    uint8_t cycles, alt_cycles;
    enum {
        INST_FLAG_NONE = 0x00,
        INST_FLAG_PRESERVE_CC = 0x01,
        INST_FLAG_PERS_WRITE = 0x02,
        INST_FLAG_USES_CC = 0x04,
        INST_FLAG_AFFECTS_CC = 0x08,
        INST_FLAG_ENDS_BLOCK = 0x10,
        INST_FLAG_SAVE_CC = 0x20,
        INST_FLAG_RESTORE_CC = 0x40
    } flags;
} gbz80_inst;
```


**emit**

関数`compile`で生成した`gb_block`を使う

```c++
```

