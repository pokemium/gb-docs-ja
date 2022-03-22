# GB構造体

文字通りGBを表すルート構造体

```c++
struct GB {
	struct mCPUComponent d;

	struct SM83Core* cpu;
	struct GBMemory memory;
	struct GBVideo video;
	struct GBTimer timer;
	struct GBAudio audio;
	struct GBSIO sio;
	enum GBModel model;
  
	struct mTiming timing;

	bool cpuBlocked;
	bool earlyExit;
	struct mTimingEvent eiPending;
	unsigned doubleSpeed;
};

struct GBCartridge {
	uint8_t entry[4];
	uint8_t logo[48];
	union {
		char titleLong[16];
		struct {
			char titleShort[11];
			char maker[4];
			uint8_t cgb;
		};
	};
	char licensee[2];
	uint8_t sgb;
	uint8_t type;
	uint8_t romSize;
	uint8_t ramSize;
	uint8_t region;
	uint8_t oldLicensee;
	uint8_t version;
	uint8_t headerChecksum;
	uint16_t globalChecksum;
	// And ROM data...
};

struct GBTimer {
	struct GB* p;

	struct mTimingEvent event;
	struct mTimingEvent irq;

	uint32_t internalDiv;
	int32_t nextDiv;
	uint32_t timaPeriod;
};
```

