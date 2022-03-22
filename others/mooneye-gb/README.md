# mooneye-gb

この記事は[mooneye-gb](https://github.com/Gekkio/mooneye-gb/tree/2d52008228557f9e713545e702d5b7aa233d09bb)のコードリーディングから得られたものをまとめたものです。

## コンテンツ一覧

- [データ構造](struct.md)
- [割り込み](interrupt.md)

## 実行

`Machine.emulate`で行っている

イベントが起きるか、所定のサイクルを消費し終えるまで実行を続け、返り値として、生じたイベントとエミュの時間を返す

CPUからhardwareの情報をいじる必要があるのでhardwareをmutableとして借用している

CPUが直接hardwareのmutable参照を保持しないのはなぜ？

```rust
impl Machine {
  pub fn emulate(&mut self, target_time: EmuTime) -> (EmuEvents, EmuTime) {
    let mut step = self.step;
     
     // イベントが起こるか、target_timeのサイクルを消費し終えるまで execute_step を実行し続ける
    loop {
      step = self.cpu.execute_step(&mut self.hardware, step);   // ここでCPUに 実行に必要なhardware(Hardware)を貸している
      if !self.hardware.emu_events().is_empty() || self.hardware.emu_time() >= target_time {
        break;
      }
    }
    
    self.step = step;
    (self.hardware.ack_emu_events(), self.hardware.emu_time())
  }
}

impl CPU {
fn prefetch_next<H: CpuContext>(&mut self, ctx: &mut H, addr: u16) -> Step {
    let (interrupts, opcode) = ctx.read_cycle_intr(addr);
    self.opcode = opcode;
    if self.ime && !interrupts.is_empty() {
      Step::InterruptDispatch
    } else {
      self.regs.pc = addr.wrapping_add(1);
      Step::Running
    }
  }
  
  pub fn execute_step<H: CpuContext>(&mut self, ctx: &mut H, step: Step) -> Step {
    match step {
      Step::Running => self.decode_exec_fetch(ctx),
      
      Step::InterruptDispatch => {
        self.ime = false;
        ctx.tick_cycle();

        let [lo, hi] = u16::to_le_bytes(self.regs.pc);
        ctx.tick_cycle();

        self.regs.sp = self.regs.sp.wrapping_sub(1);
        ctx.write_cycle(self.regs.sp, hi);

        self.regs.sp = self.regs.sp.wrapping_sub(1);
        let interrupts = ctx.write_cycle_intr(self.regs.sp, lo);

        let interrupt = interrupts.highest_priority();
        ctx.ack_interrupt(interrupt);
        self.regs.pc = match interrupt {
          InterruptLine::VBLANK => 0x0040,
          InterruptLine::STAT => 0x0048,
          InterruptLine::TIMER => 0x0050,
          InterruptLine::SERIAL => 0x0058,
          InterruptLine::JOYPAD => 0x0060,
          _ => 0x0000,
        };
        self.opcode = self.fetch_imm8(ctx);
        Step::Running
      }
      
      Step::Halt => {
        if ctx.has_interrupt() {
          self.prefetch_next(ctx, self.regs.pc)
        } else {
          ctx.tick_cycle();
          Step::Halt
        }
      }
    }
  }
}
```

## OAM DMA

```rust
pub struct Peripherals {
  ...
  oam_dma: OamDma,
}

struct OamDma {
  bus: Option<ExternalBus>,
  source: u8,
  requested: Option<u8>,
  starting: Option<u8>,
  addr: u16,
}
```

所有権システムのため、OamDma自体はOamにアクセスすることはできず、OamDma転送に必要なプロパティを保持しているのみになっている

実際の転送は、Oam(`.ppu.oam`)と`OamDma`両方の所有権を持っている`Peripherals`が`Peripherals.emulate_oam_dma`で転送を行っている

## Callbacks と EmuEvents

**Callbacks**

用途不明のトレイト。実装しているのは`EmuEvents`のみ。

```rust
pub trait Callbacks {
  fn debug_opcode(&mut self);
  fn bootrom_disabled(&mut self);
  fn trigger_emu_events(&mut self, emu_events: EmuEvents);
}

impl Callbacks for EmuEvents {
  fn debug_opcode(&mut self) {
    self.insert(EmuEvents::DEBUG_OP);
  }
  fn bootrom_disabled(&mut self) {
    self.insert(EmuEvents::BOOTROM_DISABLED);
  }
  fn trigger_emu_events(&mut self, emu_events: EmuEvents) {
    self.insert(emu_events);
  }
}
```

**EmuEvents**

もしかすると、GBとしての実行には関係ない？

```rust
bitflags!(
  pub struct EmuEvents: u8 {
    const DEBUG_OP         = 0b_0000_0001;  // ld b, b (opcode=0x40) でトリガーされることがある デバッグ用？
    const VSYNC            = 0b_0000_0010;  // frontendのInGameStateで使われている
    const BOOTROM_DISABLED = 0b_0000_0100;  // 0xff50 と何か関係あり
  }
);
```

## コンテキスト

存在するのは`CoreContext`, `CpuContext`, `PeripheralsContext` の3つ

`PeripheralsContext`は実装していると、割り込みフラグの取得・更新、割り込みリクエストが可能になる

基本的に、コンテキストは全部、サブトレイトを使って`CoreContext`を実装するようにしている

**コンテキストを実装している構造体一覧**

- `CoreContext`: `Hardware`, `InterruptCheck`, `(T, &mut EmuEvents)`
- `CpuContext`: `Hardware`
- `PeripheralsContext`: `(&mut Interrupts, &mut EmuEvents)`, `InterruptCheck`

```rust
pub trait CoreContext {
  fn callbacks(&mut self) -> Option<&mut dyn Callbacks>;
}

pub trait CpuContext: CoreContext {
  fn read_cycle(&mut self, addr: u16) -> u8;
  fn read_cycle_high(&mut self, addr: u8) -> u8 {
    self.read_cycle(0xff00 | (addr as u16))
  }
  fn read_cycle_intr(&mut self, addr: u16) -> (InterruptLine, u8);
  fn write_cycle(&mut self, addr: u16, data: u8);
  fn write_cycle_high(&mut self, addr: u8, data: u8) {
    self.write_cycle(0xff00 | (addr as u16), data);
  }
  fn write_cycle_intr(&mut self, addr: u16, data: u8) -> InterruptLine;
  fn tick_cycle(&mut self);
  fn has_interrupt(&self) -> bool;
  fn ack_interrupt(&mut self, mask: InterruptLine);
}

pub trait PeripheralsContext: CoreContext + InterruptRequest {
  fn interrupts(&self) -> &Interrupts;
  fn interrupts_mut(&mut self) -> &mut Interrupts;
}
```

## グラフィック

`Peripherals.generic_cycle`で`Ppu.emulate`

当然、`Peripherals`は`Ppu`の所有権を持っている

## タイマー

```rust
pub struct Timer {
  internal_counter: u16,
  tac: TacReg,
  counter: u8,
  modulo: u8,
  overflow: bool,
  enabled: bool,
}
```

生のバイナリデータではなく、意味のあるデータとして持っている

GBの場合は、TimerからやるのはせいぜいIRQを起こすくらい

GBAはDMAが絡んでくるので設計を考える

**所有権**

Timerのメソッドから見るとInterruptsは次のような関係になっているので、HardwareがInterruptsを内包した`PeripheralsContext`をPeripheralsに貸すことでTimerからアクセスできるようになっている

```sh
Hardware
├── Peripherals
│    └── Timer
└── Interrupts
```

`Hardware.write_cycle` > `Peripherals.write` > `Timer.tac_write_cycle` > `Timer.tick_cycle`

