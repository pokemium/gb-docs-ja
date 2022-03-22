# デバッグ処理

デバッグ処理の大本は`GB_run`内の`GB_debugger_run`で行われています

`GB_run`までの処理は[処理の流れ](flow.md)を参照してください。

`GB_run`ごとに呼ばれるこの関数はデバッグモード、つまり`gb->debug_disable = false`ならその内容を実行します。

```c
// Core/debugger.c
void GB_debugger_run(GB_gameboy_t *gb) {
    if (gb->debug_disable) return;
    
    if (!gb->undo_state) {
        gb->undo_state = malloc(GB_get_save_state_size_no_bess(gb));
        GB_save_state_to_buffer_no_bess(gb, gb->undo_state);
    }

    char *input = NULL;
    if (gb->debug_next_command && gb->debug_call_depth <= 0 && !gb->halted) {
        gb->debug_stopped = true;
    }
    if (gb->debug_fin_command && gb->debug_call_depth == -1) {
        gb->debug_stopped = true;
    }
    if (gb->debug_stopped) {
        GB_cpu_disassemble(gb, gb->pc, 5);
    }
next_command:
    if (input) {
        free(input);
    }
    if (gb->breakpoints && !gb->debug_stopped && should_break(gb, gb->pc, false)) {
        gb->debug_stopped = true;
        GB_log(gb, "Breakpoint: PC = %s\n", value_to_string(gb, gb->pc, true));
        GB_cpu_disassemble(gb, gb->pc, 5);
    }

    if (gb->breakpoints && !gb->debug_stopped) {
        uint16_t address = 0;
        jump_to_return_t jump_to_result = test_jump_to_breakpoints(gb, &address);

        bool should_delete_state = true;
        if (gb->nontrivial_jump_state && should_break(gb, gb->pc, true)) {
            if (gb->non_trivial_jump_breakpoint_occured) {
                gb->non_trivial_jump_breakpoint_occured = false;
            }
            else {
                gb->non_trivial_jump_breakpoint_occured = true;
                GB_log(gb, "Jumping to breakpoint: PC = %s\n", value_to_string(gb, gb->pc, true));
                GB_cpu_disassemble(gb, gb->pc, 5);
                GB_load_state_from_buffer(gb, gb->nontrivial_jump_state, -1);
                gb->debug_stopped = true;
            }
        }
        else if (jump_to_result == JUMP_TO_BREAK) {
            gb->debug_stopped = true;
            GB_log(gb, "Jumping to breakpoint: PC = %s\n", value_to_string(gb, address, true));
            GB_cpu_disassemble(gb, gb->pc, 5);
            gb->non_trivial_jump_breakpoint_occured = false;
        }
        else if (jump_to_result == JUMP_TO_NONTRIVIAL) {
            if (!gb->nontrivial_jump_state) {
                gb->nontrivial_jump_state = malloc(GB_get_save_state_size_no_bess(gb));
            }
            GB_save_state_to_buffer_no_bess(gb, gb->nontrivial_jump_state);
            gb->non_trivial_jump_breakpoint_occured = false;
            should_delete_state = false;
        }
        else {
            gb->non_trivial_jump_breakpoint_occured = false;
        }

        if (should_delete_state) {
            if (gb->nontrivial_jump_state) {
                free(gb->nontrivial_jump_state);
                gb->nontrivial_jump_state = NULL;
            }
        }
    }

    if (gb->debug_stopped && !gb->debug_disable) {
        gb->debug_next_command = false;
        gb->debug_fin_command = false;
        gb->stack_leak_detection = false;
        input = gb->input_callback(gb);

        if (input == NULL) {
            /* Debugging is no currently available, continue running */
            gb->debug_stopped = false;
            return;
        }

        if (GB_debugger_execute_command(gb, input)) {
            goto next_command;
        }

        free(input);
    }
}
```

## ブレークポイント

```c
// Core/debugger.c
struct GB_breakpoint_s {
    union {
        struct {
        uint16_t addr;
        uint16_t bank; /* -1 = any bank*/
        };
        uint32_t key; /* ソートと比較のためのID */
    };
    char *condition;
    bool is_jump_to;
};
```

ブレークポイントに到達したかどうかの判定は`should_break`で行っています。

上の`GB_debugger_run`を見ると確かにブレークポイントの判定が行われています。

```c++
void GB_debugger_run(GB_gameboy_t *gb) {
    // ...

    if (gb->breakpoints && !gb->debug_stopped && should_break(gb, gb->pc, false)) {
        gb->debug_stopped = true;
        GB_log(gb, "Breakpoint: PC = %s\n", value_to_string(gb, gb->pc, true));
        GB_cpu_disassemble(gb, gb->pc, 5);
    }

    // ...
}
```

## GB_cpu_disassemble

デバッグコンソールに

```
  ->0100: NOP
    0101: JP $016e
    0104: ADC $ed
    0106: LD h, [hl]
    0107: LD h, [hl]
```

のような逆アセンブル結果を表示します。`count`引数で直近いくつの命令を逆アセンブルするか指定できるようです。

