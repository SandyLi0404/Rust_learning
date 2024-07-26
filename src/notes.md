# 点灯程序

## main.rs

```rust
let cp = cortex_m::Peripherals::take().unwrap();
let dp = pac::Peripherals::take().unwrap();
```

- cortex_m: provides access to Cortex-M specific peripherals
  - 例如：NVIC (Nested Vectored Interrupt Controller), Systick Timer, SCB (System Control Block), MPU
- pac: provides access to all the device-specific peripherals
  - e.g., GPIO, ADC, Timers/Counters, I2C
- `take()`: retrieves singleton instance of peripherals
- `unwrap()`: take之后返回一个option，用unwrap取出Some()里的东西；如果是None直接panic

```rust
let pwr = dp.PWR.constrain(); // using HAL's abstraction
let pwrcfg = pwr.freeze();
```

- `constrain()`: 使用HAL abstraction，把pwr peripheral的所有权转移给HAL
- `freeze()`: finalizes the configuration of the PWR peripheral, freeze后无法再更改

```rust
let rcc = dp.RCC.constrain(); // 设置时钟
let ccdr = rcc.sys_ck(200.MHz()).freeze(pwrcfg, &dp.SYSCFG);
```

- `rcc.sys_ck(200.MHz())`: 设置system clock的频率为200 MHz
- `&dp.SYSCFG`: System Configuration Controller (SYSCFG) peripheral的reference

```rust
let gpioe = dp.GPIOE.split(prec::ccdr.peripheral.GPIOE);
let mut led = gpioe.pe3.into_push_pull_output();
```

- `split()`: 普通单一的外设用constrain, 有多个相似外设的外设(例如GPIO和SPI)使用split
- `prec::ccdr.peripheral.GPIOE`: 给GPIOE设置时钟
- `pe3.into_push_pull_output()`: 将pe3 pin设置为push_pull_output

```rust
let mut delay = cp.SYST.delay(ccdr.clocks);
```

- `cp.SYST`: 通过cp access SysTick timer设置delay

## 设置openocd config

```rust
source [find interface/stlink.cfg]

source [find target/stm32h7x.cfg]
```

- 设置烧录器和所用芯片

## 烧录程序

- `openocd -f openocd.cfg -c "program target/thumbv7em-none-eabihf/rust-embedded-demo preverify verify reset exit 0x08000000"`
  - `-f`选择烧录配置
  - `-c`烧录命令
  - `target/thumbv7em-none-eabihf/rust-embedded-demo`固件地址
  - `preverify`和`verify`烧录前后校验
  - `0x08000000`烧录地址
