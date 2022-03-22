# ステートセーブ

## セーブステートのデータ構造

[こちら](state.md)を参照

## core

```c++
// src/gb/core.c
struct mCore* GBCoreCreate(void) {
    // ...

    core->loadState = _GBCoreLoadState;
    core->saveState = _GBCoreSaveState;

    // ...
}
```

## セーブ処理

```c++
// src/gb/core.c
static bool _GBCoreSaveState(struct mCore* core, void* state) {
	struct SM83Core* cpu = core->cpu;
	while (cpu->executionState != SM83_CORE_FETCH) {
		SM83Tick(cpu);
	}
	GBSerialize(core->board, state);
	return true;
}
```

## ロード処理

```c++
// src/gb/core.c
static bool _GBCoreLoadState(struct mCore* core, const void* state) {
	return GBDeserialize(core->board, state);
}
```
