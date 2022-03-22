# メモリのロード

メモリロードは`SM83Memory`の`cpuLoad8`と`load8`が使われます。

`cpuLoad8`は`GBLoad8`または`GBCartLoad8`のどちらかで、CPUがプログラムカウンタから命令をフェッチする際に使われます。

`load8`は常に`GBLoad8`を指しており、汎用的なロード関数です。

## GBLoad8

```c++
uint8_t GBLoad8(struct SM83Core* cpu, uint16_t address) {
	struct GB* gb = (struct GB*) cpu->master;
	struct GBMemory* memory = &gb->memory;

	if (gb->memory.dmaRemaining) {
		const enum GBBus* block = gb->model < GB_MODEL_CGB ? _oamBlockDMG : _oamBlockCGB;
		enum GBBus dmaBus = block[memory->dmaSource >> 13];
		enum GBBus accessBus = block[address >> 13];
		if (dmaBus != GB_BUS_CPU && dmaBus == accessBus) {
			return 0xFF;
		}
		if (address >= GB_BASE_OAM && address < GB_BASE_IO) {
			return 0xFF;
		}
	}

	switch (address >> 12) {
    // 0x0000-0x3fff
	case GB_REGION_CART_BANK0:
	case GB_REGION_CART_BANK0 + 1:
	case GB_REGION_CART_BANK0 + 2:
	case GB_REGION_CART_BANK0 + 3:
		if (address >= memory->romSize) {
			memory->cartBus = 0xFF;
		} else {
			memory->cartBus = memory->romBase[address & (GB_SIZE_CART_BANK0 - 1)];
		}
		memory->cartBusPc = cpu->pc;
		return memory->cartBus;

    // 0x6000-0x7fff
	case GB_REGION_CART_BANK1 + 2:
	case GB_REGION_CART_BANK1 + 3:
		if (memory->mbcType == GB_MBC6) {
			memory->cartBus = memory->mbcState.mbc6.romBank1[address & (GB_SIZE_CART_HALFBANK - 1)];
			memory->cartBusPc = cpu->pc;
			return memory->cartBus;
		}
		// Fall through

    // 0x4000-0x5fff
	case GB_REGION_CART_BANK1:
	case GB_REGION_CART_BANK1 + 1:
		if (address >= memory->romSize) {
			memory->cartBus = 0xFF;
		} else if ((memory->mbcType & GB_UNL_BBD) == GB_UNL_BBD) {
			memory->cartBus = memory->mbcRead(memory, address);
		} else {
			memory->cartBus = memory->romBank[address & (GB_SIZE_CART_BANK0 - 1)];
		}
		memory->cartBusPc = cpu->pc;
		return memory->cartBus;

    // 0x8000-0x9fff
	case GB_REGION_VRAM:
	case GB_REGION_VRAM + 1:
		if (gb->video.mode != 3) {
			return gb->video.vramBank[address & (GB_SIZE_VRAM_BANK0 - 1)];
		}
		return 0xFF;

    // 0xa000-0xbfff
	case GB_REGION_EXTERNAL_RAM:
	case GB_REGION_EXTERNAL_RAM + 1:
		if (memory->rtcAccess) {
			memory->cartBus = memory->rtcRegs[memory->activeRtcReg];
		} else if (memory->mbcRead) {
			memory->cartBus = memory->mbcRead(memory, address);
		} else if (memory->sramAccess && memory->sram) {
			memory->cartBus = memory->sramBank[address & (GB_SIZE_EXTERNAL_RAM - 1)];
		} else if (memory->mbcType == GB_HuC3) {
			memory->cartBus = 0x01; // TODO: Is this supposed to be the current SRAM bank?
		} else if (cpu->tMultiplier * (cpu->pc - memory->cartBusPc) >= memory->cartBusDecay) {
			memory->cartBus = 0xFF;
		}
		memory->cartBusPc = cpu->pc;
		return memory->cartBus;

    // 0xc000-0xcfff (+ミラー)
	case GB_REGION_WORKING_RAM_BANK0:
	case GB_REGION_WORKING_RAM_BANK0 + 2:
		return memory->wram[address & (GB_SIZE_WORKING_RAM_BANK0 - 1)];

    // 0xd000-0xdfff
	case GB_REGION_WORKING_RAM_BANK1:
		return memory->wramBank[address & (GB_SIZE_WORKING_RAM_BANK0 - 1)];

	default:
		if (address < GB_BASE_OAM) {
			return memory->wramBank[address & (GB_SIZE_WORKING_RAM_BANK0 - 1)];
		}

		if (address < GB_BASE_UNUSABLE) {
			if (gb->video.mode < 2) {
				return gb->video.oam.raw[address & 0xFF];
			}
			return 0xFF;
		}

		if (address < GB_BASE_IO) {
			mLOG(GB_MEM, GAME_ERROR, "Attempt to read from unusable memory: %04X", address);
			return 0xFF;
		}

		if (address < GB_BASE_HRAM) {
			return GBIORead(gb, address & (GB_SIZE_IO - 1));
		}

		if (address < GB_BASE_IE) {
			return memory->hram[address & GB_SIZE_HRAM];
		}

		return GBIORead(gb, GB_REG_IE);
	}
}
```


## GBCartLoad8

```c++
static uint8_t GBCartLoad8(struct SM83Core* cpu, uint16_t address) {
	// 範囲外に飛び出していた場合はやり直し
	if (UNLIKELY(address >= cpu->memory.activeRegionEnd)) {
		cpu->memory.setActiveRegion(cpu, address);
		return cpu->memory.cpuLoad8(cpu, address);
	}

	// カートリッジからの読み出しを行う
	struct GB* gb = (struct GB*) cpu->master;
	struct GBMemory* memory = &gb->memory;
	memory->cartBusPc = address;
	uint8_t value = cpu->memory.activeRegion[address & cpu->memory.activeMask];
	memory->cartBus = value;
	return value;
}
```

## Region

`GBMemory.setActiveRegion`は`SM83Core.GBSetActiveRegion`を参照しています。

`GBSetActiveRegion`では

- `cpuLoad8`の指す関数を、regionによって切り替える処理
- `SM83Memory`の`activeRegion`関連のデータの設定

を行なっています。カートリッジからの読み取り(`GBCartLoad8`)のための関数のように思われます。

```c++
static void GBSetActiveRegion(struct SM83Core* cpu, uint16_t address) {
	struct GB* gb = (struct GB*) cpu->master;
	struct GBMemory* memory = &gb->memory;

	switch (address >> 12) {
    // 0x0000-0x3fff ROMバンク0
	case GB_REGION_CART_BANK0:
	case GB_REGION_CART_BANK0 + 1:
	case GB_REGION_CART_BANK0 + 2:
	case GB_REGION_CART_BANK0 + 3:
        // cpuLoad8 に GBCartLoad8 をセット
		cpu->memory.cpuLoad8 = GBCartLoad8;

        // activeRegionのデータを設定
		cpu->memory.activeRegion = memory->romBase;
		cpu->memory.activeRegionEnd = GB_BASE_CART_BANK1;
		cpu->memory.activeMask = GB_SIZE_CART_BANK0 - 1;
		if (gb->memory.romSize < GB_SIZE_CART_BANK0) {
			if (address >= gb->memory.romSize) {
				// 無効なROM読み出し
				cpu->memory.activeRegion = _yankBuffer;
				cpu->memory.activeMask = 0;
			} else {
				cpu->memory.activeRegionEnd = gb->memory.romSize;
			}
		}
		break;

    // 0x4000-0x7fff ROMバンクnn
	case GB_REGION_CART_BANK1:
	case GB_REGION_CART_BANK1 + 1:
	case GB_REGION_CART_BANK1 + 2:
	case GB_REGION_CART_BANK1 + 3:
        // cpuLoad8 に GBCartLoad8 をセット
		if ((gb->memory.mbcType & GB_UNL_BBD) == GB_UNL_BBD) {
			cpu->memory.cpuLoad8 = GBLoad8;
			break;
		}
		cpu->memory.cpuLoad8 = GBCartLoad8;

        // activeRegionのデータを設定
		if (gb->memory.mbcType != GB_MBC6) {
			cpu->memory.activeRegion = memory->romBank;
			cpu->memory.activeRegionEnd = GB_BASE_VRAM;
			cpu->memory.activeMask = GB_SIZE_CART_BANK0 - 1;
		} else {
			cpu->memory.activeMask = GB_SIZE_CART_HALFBANK - 1;
			if (address & 0x2000) {
				cpu->memory.activeRegion = memory->mbcState.mbc6.romBank1;
				cpu->memory.activeRegionEnd = GB_BASE_VRAM;
			} else {
				cpu->memory.activeRegion = memory->romBank;
				cpu->memory.activeRegionEnd = GB_BASE_CART_BANK1 + 0x2000;
			}
		}

		if (gb->memory.romSize < GB_SIZE_CART_BANK0 * 2) {
			if (address >= gb->memory.romSize) {
				// 無効なROM読み出し
				cpu->memory.activeRegion = _yankBuffer;
				cpu->memory.activeMask = 0;
			} else {
				cpu->memory.activeRegionEnd = gb->memory.romSize;
			}
		}
		break;

	default:
        // カートリッジ以外のメモリ領域では、 cpuLoad8 に GBLoad8 をセット
		cpu->memory.cpuLoad8 = GBLoad8;
		break;
	}
	
	if (gb->memory.dmaRemaining) {
		const enum GBBus* block = gb->model < GB_MODEL_CGB ? _oamBlockDMG : _oamBlockCGB;
		enum GBBus dmaBus = block[memory->dmaSource >> 13];
		enum GBBus accessBus = block[address >> 13];
		if ((dmaBus != GB_BUS_CPU && dmaBus == accessBus) || (address >= GB_BASE_OAM && address < GB_BASE_UNUSABLE)) {
			cpu->memory.activeRegion = _blockedRegion;
			cpu->memory.activeMask = 0;
		}
	}
}
```

## cartBus, cartBusPc

これらはメモリの細かい動作を再現するために使われているようです。

`cartBusPc`には名前のとおり、現在のpcがメモリの特定の部分を読み取ったときに代入されます。

これはRAMからの読み取りの際に使用されていますが、どの仕様に則ったものかは不明です。

```c++
uint8_t GBLoad8(struct SM83Core* cpu, uint16_t address) {
    // ...

    switch (address >> 12) {
        // 0xa000-0xbfff
        case GB_REGION_EXTERNAL_RAM:
        case GB_REGION_EXTERNAL_RAM + 1:
            if (memory->rtcAccess) {
                memory->cartBus = memory->rtcRegs[memory->activeRtcReg];
            } else if (memory->mbcRead) {
                memory->cartBus = memory->mbcRead(memory, address);
            } else if (memory->sramAccess && memory->sram) {
                memory->cartBus = memory->sramBank[address & (GB_SIZE_EXTERNAL_RAM - 1)];
            } else if (memory->mbcType == GB_HuC3) {
                memory->cartBus = 0x01; // TODO: Is this supposed to be the current SRAM bank?
            } else if (cpu->tMultiplier * (cpu->pc - memory->cartBusPc) >= memory->cartBusDecay) {
                // ここに注目！
                memory->cartBus = 0xFF;
            }
            memory->cartBusPc = cpu->pc;
            return memory->cartBus;
    }
```

`cartBus`に関しては`return memory->cartBus`でしか使っておらず用途不明です。今後の改良のために使われるのかもしれません。

