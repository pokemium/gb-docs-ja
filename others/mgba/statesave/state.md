# ステートについて

```c++
// include/mgba/internal/gb/serialize.h

// Total size: 0x11800 (71,680) bytes
struct GBSerializedState {
    /*
     * 0x00000 - 0x00003: Version Magic (0x01000002)
     * 0x00004 - 0x00007: ROM CRC32
     * 0x00008: Game Boy model
     * 0x00009 - 0x0000B: Reserved (leave zero)
     * 0x0000C - 0x0000F: Master cycles
     */
	uint32_t versionMagic;
	uint32_t romCrc32;
	uint8_t model;
	uint8_t reservedHeader[3];
	uint32_t masterCycles;

    /*
     * 0x00010 - 0x0001F: Game title/code (e.g. PM_CRYSTALBYTE)
     */
	char title[16];

    /*
     * 0x00020 - 0x00047: CPU state:
     * | 0x00020: A register
     * | 0x00021: F register
     * | 0x00022: B register
     * | 0x00023: C register
     * | 0x00024: D register
     * | 0x00025: E register
     * | 0x00026: H register
     * | 0x00027: L register
     * | 0x00028 - 0x00029: SP register
     * | 0x0002A - 0x0002B: PC register
     * | 0x0002C - 0x0002F: Cycles since last event
     * | 0x00030 - 0x00033: Cycles until next event
     * | 0x00034 - 0x00035: Reserved (current instruction)
     * | 0x00036 - 0x00037: Index address
     * | 0x00038: Bus value
     * | 0x00039: Execution state
     * | 0x0003A - 0x0003B: Reserved
     * | 0x0003C - 0x0003F: EI pending cycles
     * | 0x00040 - 0x00043: Reserved (DI pending cycles)
     * | 0x00044 - 0x00047: Flags
     *   | bit 0: Is condition met?
     *   | bit 1: Is IRQ pending?
     *   | bit 2: Double speed
     *   | bit 3: Is EI pending?
     *   | bits 4 - 31: Reserved
     */
	struct {
		uint8_t a;
		uint8_t f;
		uint8_t b;
		uint8_t c;
		uint8_t d;
		uint8_t e;
		uint8_t h;
		uint8_t l;
		uint16_t sp;
		uint16_t pc;

		int32_t cycles;
		int32_t nextEvent;

		uint16_t reservedInstruction;
		uint16_t index;
		uint8_t bus;
		uint8_t executionState;

		uint16_t reserved;

		uint32_t eiPending;
		int32_t reservedDiPending;
		GBSerializedCpuFlags flags;
	} cpu;

    /*
     * 0x00048 - 0x0005B: Audio channel 1/framer state
     * | 0x00048 - 0x0004B: Envelepe timing
     *   | bits 0 - 6: Remaining length
     *   | bits 7 - 9: Next step
     *   | bits 10 - 20: Shadow frequency register
     *   | bits 21 - 31: Reserved
     * | 0x0004C - 0x0004F: Next frame
     * | 0x00050 - 0x00053: Next channel 3 fade
     * | 0x00054 - 0x00057: Sweep state
     *   | bits 0 - 2: Timesteps
     *   | bits 3 - 31: Reserved
     * | 0x00058 - 0x0005B: Next event
     * 0x0005C - 0x0006B: Audio channel 2 state
     * | 0x0005C - 0x0005F: Envelepe timing
     *   | bits 0 - 2: Remaining length
     *   | bits 3 - 5: Next step
     *   | bits 6 - 31: Reserved
     * | 0x00060 - 0x00067: Reserved
     * | 0x00068 - 0x0006B: Next event
     * 0x0006C - 0x00093: Audio channel 3 state
     * | 0x0006C - 0x0008B: Wave banks
     * | 0x0008C - 0x0008D: Remaining length
     * | 0x0008E - 0x0008F: Reserved
     * | 0x00090 - 0x00093: Next event
     * 0x00094 - 0x000A3: Audio channel 4 state
     * | 0x00094 - 0x00097: Linear feedback shift register state
     * | 0x00098 - 0x0009B: Envelepe timing
     *   | bits 0 - 2: Remaining length
     *   | bits 3 - 5: Next step
     *   | bits 6 - 31: Reserved
     * | 0x0009C - 0x0009F: Last event
     * | 0x000A0 - 0x000A3: Next event
     * 0x000A4 - 0x000B7: Audio miscellaneous state
     * | TODO: Fix this, they're in big-endian order, but field is little-endian
     * | 0x000A4: Channel 1 envelope state
     *   | bits 0 - 3: Current volume
     *   | bits 4 - 5: Is dead?
     *   | bit 6: Is high?
     *   | bit 7: Reserved
     * | 0x000A5: Channel 2 envelope state
     *   | bits 0 - 3: Current volume
     *   | bits 4 - 5: Is dead?
     *   | bit 6: Is high?
     *   | bit 7: Reserved
     * | 0x000A6: Channel 4 envelope state
     *   | bits 0 - 3: Current volume
     *   | bits 4 - 5: Is dead?
     *   | bits 6 - 7: Current frame (continued)
     * | 0x000A7: Miscellaneous audio flags
     *   | bit 0: Current frame (continuation)
     *   | bit 1: Is channel 1 sweep enabled?
     *   | bit 2: Has channel 1 sweep occurred?
     *   | bit 3: Is channel 3's memory readable?
     *   | bit 4: Skip frame
     *   | bits 5 - 7: Reserved
     * | 0x000A8 - 0x000AB: Left capacitor charge
     * | 0x000AC - 0x000AF: Right capacitor charge
     * | 0x000B0 - 0x000B3: Next sample
     */
	struct {
		struct GBSerializedPSGState psg;
		GBSerializedAudioFlags flags;
		int32_t capLeft;
		int32_t capRight;
		uint32_t nextSample;
	} audio;

    /*
     * 0x000B4 - 0x000153: Video state
     * | 0x000B4 - 0x000B5: Current x
     * | 0x000B6 - 0x000B7: Current y (ly)
     * | 0x000B8 - 0x000BB: Next frame
     * | 0x000BC - 0x000BF: Reserved
     * | 0x000C0 - 0x000C3: Next mode
     * | 0x000C4 - 0x000C7: Dot cycle counter
     * | 0x000C8 - 0x000CB: Frame counter
     * | 0x000CC: Current VRAM bank
     * | 0x000CD: Palette flags
     *   | bit 0: BCP increment
     *   | bit 1: OCP increment
     *   | bits 2 - 3: Mode
     *   | bits 4 - 7: Reserved
     * | 0x000CE - 0x000CF: Reserved
     * | 0x000D0 - 0x000D1: BCP index
     * | 0x000D1 - 0x000D3: OCP index
     * | 0x000D4 - 0x00153: Palette entries
     */
	struct {
		int16_t x;
		int16_t ly;
		uint32_t nextFrame;
		uint32_t reserved;
		uint32_t nextMode;
		int32_t dotCounter;
		int32_t frameCounter;

		uint8_t vramCurrentBank;
		GBSerializedVideoFlags flags;
		uint16_t reserved2;

		uint16_t bcpIndex;
		uint16_t ocpIndex;

		uint16_t palette[64];
	} video;

    /*
     * 0x00154 - 0x000167: Timer state
     * | 0x00154 - 0x00157: Next event
     * | 0x00158 - 0x0015B: Next IRQ
     * | 0x0015C - 0x0015F: Next DIV
     * | 0x00160 - 0x00163: Inernal DIV
     * | 0x00164: TIMA period
     * | 0x00165: Flags
     *   | bit 0: Is IRQ pending?
     * | 0x00166 - 0x00167: Reserved
     */
	struct {
		uint32_t nextEvent;
		uint32_t nextIRQ;

		uint32_t nextDiv;
		uint32_t internalDiv;
		uint8_t timaPeriod;
		GBSerializedTimerFlags flags;
		uint16_t reserved;
	} timer;

    /*
     * 0x000168 - 0x000197: Memory state
     * | 0x00168 - 0x00169: Current ROM bank
     * | 0x0016A: Current WRAM bank
     * | 0x0016B: Current SRAM bank
     * | 0x0016C - 0x0016F: Next DMA
     * | 0x00170 - 0x00171: Next DMA source
     * | 0x00172 - 0x00173: Next DMA destination
     * | 0x00174 - 0x00177: Next HDMA
     * | 0x00178 - 0x00179: Next HDMA source
     * | 0x0017A - 0x0017B: Next HDMA destination
     * | 0x0017C - 0x0017D: HDMA remaining
     * | 0x0017E: DMA remaining
     * | 0x0017F - 0x00183: RTC registers
     * | 0x00184 - 0x00193: MBC state
     * | 0x00194 - 0x00195: Flags
     *   | bit 0: SRAM accessable
     *   | bit 1: RTC accessible
     *   | bit 2: RTC latched
     *   | bit 3: IME
     *   | bit 4: Is HDMA active?
     *   | bits 5 - 7:  Active RTC register
     * | 0x00196: Cartridge bus value
     * | 0x00197: Reserved (leave zero)
     */
	struct {
		uint16_t currentBank;
		uint8_t wramCurrentBank;
		uint8_t sramCurrentBank;

		uint32_t dmaNext;
		uint16_t dmaSource;
		uint16_t dmaDest;

		uint32_t hdmaNext;
		uint16_t hdmaSource;
		uint16_t hdmaDest;

		uint16_t hdmaRemaining;
		uint8_t dmaRemaining;
		uint8_t rtcRegs[5];

		union {
			struct {
				uint8_t mode;
				uint8_t multicartStride;
				uint8_t bankLo;
				uint8_t bankHi;
			} mbc1;
			struct {
				uint64_t lastLatch;
			} rtc;
			struct {
				uint8_t state;
				GBMBC7Field eeprom;
				uint8_t address;
				uint8_t access;
				uint8_t latch;
				uint8_t srBits;
				uint16_t sr;
				uint32_t writable;
			} mbc7;
			struct {
				uint8_t locked;
				uint8_t bank0;
			} mmm01;
			struct {
				uint8_t dataSwapMode;
				uint8_t bankSwapMode;
			} bbd;
			struct {
				uint8_t reserved[16];
			} padding;
		};

		GBSerializedMemoryFlags flags;
		uint8_t cartBus;
		uint8_t reserved;
	} memory;

    /*
     * 0x00198 - 0x0019F: Global cycle counter
     * 0x001A0 - 0x001A1: Program counter for last cartridge read
     * 0x001A2 - 0x0025F: Reserved (leave zero)
     */
	uint64_t globalCycles;
	uint16_t cartBusPc;
	uint16_t reserved[95];

    /*
     * 0x00260 - 0x002FF: OAM
     * 0x00300 - 0x0037F: I/O memory
     * 0x00380 - 0x003FE: HRAM
     * 0x003FF: Interrupts enabled
     * 0x00400 - 0x043FF: VRAM
     * 0x04400 - 0x0C3FF: WRAM
     */
	uint8_t oam[GB_SIZE_OAM];
	uint8_t io[GB_SIZE_IO];
	uint8_t hram[GB_SIZE_HRAM];
	uint8_t ie;
	uint8_t vram[GB_SIZE_VRAM];
	uint8_t wram[GB_SIZE_WORKING_RAM];

    /*
     * 0x0C400 - 0x0C77F: Reserved
     */
	uint32_t reserved2[0xC4];

    /*
     * 0x0C780 - 0x117FF: Super Game Boy
     * | 0x0C780 - 0x0C7D9: Current attributes
     * | 0x0C7DA: Current command
     * | 0x0C7DB: Current bit count
     * | 0x0C7DC - 0x0C7DF: Flags
     *   | bits 0 - 1: Current P1 bits
     *   | bits 2 - 3: Current render mode
     *   | bit 4: Is a mode event not scheduled?
     *   | bit 5: Is a frame event not scheduled?
     *   | bits 6 - 31: Reserved (leave 0)
     * | 0x0C7E0 - 0x0C7EF: Current packet
     * | 0x0C7F0 - 0x0C7FF: Reserved
     * | 0x0C800 - 0x0E7FF: Character VRAM
     * | 0x0E800 - 0x0F7FF: Tile map VRAM
     * | 0x0F800 - 0x107FF: Palette VRAM
     * | 0x10800 - 0x117FF: Attribute file
     */
	struct {
		uint8_t attributes[90];
		uint8_t command;
		uint8_t bits;
		GBSerializedSGBFlags flags;
		uint8_t inProgressPacket[16];
		uint8_t packet[128];
		uint8_t charRam[SGB_SIZE_CHAR_RAM];
		uint8_t mapRam[SGB_SIZE_MAP_RAM];
		uint8_t palRam[SGB_SIZE_PAL_RAM];
		uint8_t atfRam[SGB_SIZE_ATF_RAM];
	} sgb;
};
```

