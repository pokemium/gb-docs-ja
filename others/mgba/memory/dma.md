# DMA

DMA,HDMAに関する変数は`GBMemory`で定義されています。

```c
struct GBMemory {
	// ...

    uint16_t dmaSource;
	uint16_t dmaDest;
	int dmaRemaining;

	uint16_t hdmaSource;
	uint16_t hdmaDest;
	int hdmaRemaining;
	bool isHdma;

	struct mTimingEvent dmaEvent;
	struct mTimingEvent hdmaEvent;

    // ...
};
```

## DMA(OAM DMA)

`$FF46`に書き込みがあったときに`GBMemoryDMA`が呼び出され、DMAを開始させます。

```c
void GBMemoryDMA(struct GB* gb, uint16_t base) {
	if (base >= 0xE000) {
		base &= 0xDFFF;
	}
	mTimingDeschedule(&gb->timing, &gb->memory.dmaEvent);
	mTimingSchedule(&gb->timing, &gb->memory.dmaEvent, 8 * (2 - gb->doubleSpeed));
	gb->memory.dmaSource = base;
	gb->memory.dmaDest = 0;
	gb->memory.dmaRemaining = 0xA0;
}
```

スケジューラでは`dmaEvent`(`_GBMemoryDMAService`)が呼ばれており、これが逐次転送する処理を担っています。

```c++
void _GBMemoryDMAService(struct mTiming* timing, void* context, uint32_t cyclesLate) {
	struct GB* gb = context;
	int dmaRemaining = gb->memory.dmaRemaining;
	gb->memory.dmaRemaining = 0;
	uint8_t b = GBLoad8(gb->cpu, gb->memory.dmaSource);

	// TODO: Can DMA write OAM during modes 2-3?
	gb->video.oam.raw[gb->memory.dmaDest] = b;
	gb->video.renderer->writeOAM(gb->video.renderer, gb->memory.dmaDest);
	++gb->memory.dmaSource;
	++gb->memory.dmaDest;
	gb->memory.dmaRemaining = dmaRemaining - 1;
	if (gb->memory.dmaRemaining) {
		mTimingSchedule(timing, &gb->memory.dmaEvent, 4 * (2 - gb->doubleSpeed) - cyclesLate);
	}
}
```

