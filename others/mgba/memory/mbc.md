# MBC

```c
struct GBMemory {
    // ...

	enum GBMemoryBankControllerType mbcType;
	GBMemoryBankControllerWrite mbcWrite;       // func(gb *GB, addr uint16, val byte)
	GBMemoryBankControllerRead mbcRead;         // func(mem *GBMemory, addr uint16) byte
	union GBMBCState mbcState;

    // ...
}
```

`GBMemory`の`mbcXXXX`については、`GBMBCInit`でカートリッジをパースすることでMBCを特定し、対応する値やハンドラをセットしています。

`GBMemory.mbcState`の型`GBMBCState`は厄介です。

```c
union GBMBCState {
	struct GBMBC1State mbc1;
	struct GBMBC6State mbc6;
	struct GBMBC7State mbc7;
	struct GBMMM01State mmm01;
	struct GBPocketCamState pocketCam;
	struct GBTAMA5State tama5;
	struct GBPKJDState pkjd;
	struct GBBBDState bbd;
};
```

