# 命令の実行

命令の実行の核となるのは`SM83Run`とそれが呼び出している`_SM83TickInternal`です。

ちなみに`SM83Tick`も`_SM83TickInternal`を呼び出している関数ですがデバッグ用の関数なので処理には関係ありません。

```c++
void SM83Run(struct SM83Core* cpu) {
	bool running = true;
	while (running || cpu->executionState != SM83_CORE_FETCH) {
		if (cpu->cycles < cpu->nextEvent) {
			running = _SM83TickInternal(cpu) && running;
		} else {
			cpu->irqh.processEvents(cpu); // GBProcessEvents
			running = false;
		}
	}
}
```

```c++
static inline bool _SM83TickInternal(struct SM83Core* cpu) {
	bool running = true;
	
	// 前処理
	_SM83Step(cpu);
	
	// イベントが前処理の時点で生じたか
	int t = cpu->tMultiplier;
	if (cpu->cycles + t * 2 >= cpu->nextEvent) {
		if (cpu->cycles >= cpu->nextEvent) {
			cpu->irqh.processEvents(cpu);
		}
		cpu->cycles += t;
		++cpu->executionState;
		if (cpu->cycles >= cpu->nextEvent) {
			cpu->irqh.processEvents(cpu);
		}
		cpu->cycles += t;
		++cpu->executionState;
		if (cpu->cycles >= cpu->nextEvent) {
			cpu->irqh.processEvents(cpu);
		}
		running = false;
	} else {
		cpu->cycles += t * 2;
	}
	
	// 本処理
	cpu->executionState = SM83_CORE_FETCH;
	cpu->instruction(cpu);
	cpu->cycles += t;
	return running;
}

static void _SM83Step(struct SM83Core* cpu) {
	cpu->cycles += cpu->tMultiplier;
	enum SM83ExecutionState state = cpu->executionState;
	cpu->executionState = SM83_CORE_IDLE_0;
	switch (state) {
	case SM83_CORE_FETCH:
		if (cpu->irqPending) {
			cpu->index = cpu->sp;
			cpu->irqPending = false;
			cpu->instruction = _SM83InstructionIRQ;
			cpu->irqh.setInterrupts(cpu, false);
			break;
		}
		cpu->bus = cpu->memory.cpuLoad8(cpu, cpu->pc);
		cpu->instruction = _sm83InstructionTable[cpu->bus];
		++cpu->pc;
		break;
	case SM83_CORE_MEMORY_LOAD:
		cpu->bus = cpu->memory.load8(cpu, cpu->index);
		break;
	case SM83_CORE_MEMORY_STORE:
		cpu->memory.store8(cpu, cpu->index, cpu->bus);
		break;
	case SM83_CORE_READ_PC:
		cpu->bus = cpu->memory.cpuLoad8(cpu, cpu->pc);
		++cpu->pc;
		break;
	case SM83_CORE_STALL:
		cpu->instruction = _sm83InstructionTable[0]; // NOP
		break;
	case SM83_CORE_HALT_BUG:
		if (cpu->irqPending) {
			cpu->index = cpu->sp;
			cpu->irqPending = false;
			cpu->instruction = _SM83InstructionIRQ;
			cpu->irqh.setInterrupts(cpu, false);
			break;
		}
		cpu->bus = cpu->memory.cpuLoad8(cpu, cpu->pc);
		cpu->instruction = _sm83InstructionTable[cpu->bus];
		break;
	default:
		break;
	}
}

void GBProcessEvents(struct SM83Core* cpu) {
	struct GB* gb = (struct GB*) cpu->master;
	do {
		int32_t cycles = cpu->cycles;
		int32_t nextEvent;

		cpu->cycles = 0;
		cpu->nextEvent = INT_MAX;

		nextEvent = cycles;
		do {
			nextEvent = mTimingTick(&gb->timing, nextEvent);
		} while (gb->cpuBlocked);
		// This loop cannot early exit until the SM83 run loop properly handles mid-M-cycle-exits
		cpu->nextEvent = nextEvent;

		if (cpu->halted) {
			cpu->cycles = cpu->nextEvent;
			if (!gb->memory.ie || !gb->memory.ime) {
				break;
			}
		}
		if (gb->earlyExit) {
			break;
		}
	} while (cpu->cycles >= cpu->nextEvent);
	gb->earlyExit = false;
	if (gb->cpuBlocked) {
		cpu->cycles = cpu->nextEvent;
	}
}
```


