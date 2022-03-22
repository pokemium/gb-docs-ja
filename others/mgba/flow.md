# 処理の流れ

```c++
void mGUIRun(struct mGUIRunner* runner, const char* path) {
    // ...
    while(running) {
        runner->fps = 0;
        int frame = 0;
        // ...
        while (running) {
            // ...
            runner->core->runFrame(runner->core);
            if (runner->drawFrame) {
                // ...
                ++frame;
            }
        }
    }
}
```

```c++
static void _GBCoreRunFrame(struct mCore* core) {
	struct GB* gb = core->board;
	int32_t frameCounter = gb->video.frameCounter;
	while (gb->video.frameCounter == frameCounter) {
		SM83Run(core->cpu);
	}
}

static void _GBCoreRunLoop(struct mCore* core) {
	SM83Run(core->cpu);
}
```