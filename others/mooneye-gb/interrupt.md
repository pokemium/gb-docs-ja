# 割り込み

## 🧨 `Interrupts`

```rust
#[derive(Clone, Debug)]
pub struct Interrupts {
  // 現在の割り込みリクエスト(IF)の状況を表す構造体
  intr_flags: InterruptLine,

  // 割り込み有効無効フラグ(IE)
  intr_enable: u8,
}

impl Interrupts {
  pub fn new() -> Interrupts {
    Interrupts {
      intr_flags: InterruptLine::empty(),
      intr_enable: 0x00,
    }
  }

  pub fn get_interrupt_flag(&self) -> u8 {
    const IF_UNUSED_MASK: u8 = (1 << 5) | (1 << 6) | (1 << 7);

    self.intr_flags.bits() | IF_UNUSED_MASK
  }

  pub fn get_interrupt_enable(&self) -> u8 {
    self.intr_enable
  }
  
  pub fn set_interrupt_flag(&mut self, value: u8) {
    self.intr_flags = InterruptLine::from_bits_truncate(value);
  }

  pub fn set_interrupt_enable(&mut self, value: u8) {
    self.intr_enable = value;
  }

  pub fn get_interrupt(&self) -> InterruptLine {
    self.intr_flags & InterruptLine::from_bits_truncate(self.intr_enable)
  }

  pub fn ack_interrupt(&mut self, mask: InterruptLine) {
    self.intr_flags -= mask;
  }
}

// トレイト:InterruptRequest については後述
impl InterruptRequest for Interrupts {
  fn request_t12_interrupt(&mut self, interrupt: InterruptLine) {
    self.intr_flags |= interrupt;
  }

  fn request_t34_interrupt(&mut self, interrupt: InterruptLine) {
    self.intr_flags |= interrupt;
  }
}
```

## 🚃 `InterruptLine`

現在の割り込みリクエストの状況を表す構造体

```rust
bitflags!(
  pub struct InterruptLine: u8 {
    const VBLANK = 1 << 0;
    const STAT = 1 << 1;
    const TIMER = 1 << 2;
    const SERIAL = 1 << 3;
    const JOYPAD = 1 << 4;
  }
);

// トレイト:InterruptRequest については後述
impl InterruptRequest for InterruptLine {
  fn request_t12_interrupt(&mut self, interrupt: InterruptLine) {
    *self |= interrupt;
  }
  fn request_t34_interrupt(&mut self, interrupt: InterruptLine) {
    *self |= interrupt;
  }
}
```

## 🛎 `InterruptRequest`

このトレイトを実装すると、割り込みリクエストができるようになる

`Interrupts`, `InterruptLine`, `(&mut Interrupts, &mut EmuEvents)`, `InterruptCheck`がこのトレイトを実装している

```rust
pub trait InterruptRequest {
  fn request_t12_interrupt(&mut self, interrupt: InterruptLine);
  fn request_t34_interrupt(&mut self, interrupt: InterruptLine);
}
```

`Timer.tick_cycle`で使われたり、`Ppu.check_compare_interrupt`で割り込みを起こすのに使われたりしている

## `InterruptCheck`

```rust
struct InterruptCheck<'a> {
  check: Interrupts,
  interrupts: &'a mut Interrupts,
  emu_events: &'a mut EmuEvents,
}

impl<'a> InterruptRequest for InterruptCheck<'a> {
  fn request_t12_interrupt(&mut self, interrupt: InterruptLine) {
    self.check.request_t12_interrupt(interrupt);
    self.interrupts.request_t12_interrupt(interrupt);
  }
  fn request_t34_interrupt(&mut self, interrupt: InterruptLine) {
    self.interrupts.request_t34_interrupt(interrupt);
  }
}

impl<'a> CoreContext for InterruptCheck<'a> {
  fn callbacks(&mut self) -> Option<&mut dyn Callbacks> {
    Some(self.emu_events)
  }
}

impl<'a> PeripheralsContext for InterruptCheck<'a> {
  fn interrupts(&self) -> &Interrupts {
    &self.interrupts
  }
  fn interrupts_mut(&mut self) -> &mut Interrupts {
    &mut self.interrupts
  }
}
```

