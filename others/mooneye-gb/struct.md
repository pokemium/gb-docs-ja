# データ構造

```sh
├── core
│    ├── src
│    └── Cargo.toml
├── src
│    ├── frontend
│    └── xxxx.rs
├── Cargo.lock
└── Cargo.toml
```

`mooneye-gb`自体がアプリ、`core/`がクレートライブラリとなっている

`mooneye-gb/src`はエミュレータのフロントエンド、`core/`がGBをエミュレートするバックエンド部分になっている

サウンド機能はなさそう

## core

```sh
Machine
├── Cpu
├── Hardware
│    ├── Peripherals
│    ├── Interrupts
│    ├── EmuEvents
│    └── EmuTime
└── Step
```

```rust
pub struct Machine {
  cpu: Cpu,
  hardware: Hardware,
  step: Step, // Machineの状態(Running | Halt | InterruptDispatch)
}

pub struct Cpu {
  pub regs: RegisterFile,
  pub ime: bool,
  pub opcode: u8,
}

pub struct Hardware {
  pub peripherals: Peripherals,
  interrupts: Interrupts,
  emu_events: EmuEvents,
  emu_time: EmuTime,
}

pub struct Peripherals {
  pub bootrom: Bootrom,
  pub cartridge: Cartridge,
  work_ram: WorkRam,
  hiram: HiramData,
  ppu: Ppu,
  apu: Apu,
  joypad: Joypad,
  serial: Serial,
  pub timer: Timer,
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

