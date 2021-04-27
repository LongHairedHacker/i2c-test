Stuck I2C Reproducer
====================

Minimal example of using blocking I2C with the stm32f1xx_hal crate,
that gets stuck reliably if run with `cargo run --release`, but works fine with `cargo run`.

This example tries to write the output control register of a si5351,
because that's how I ran into the issue initially.
However I could als reproduce it with a SSD1306 display module, so the actual device should not matter.
The example also behaves consistently across my multiple different bluepill boards.

The most of the settings in the repo have been taken from https://github.com/knurling-rs/app-template

Versions on my machine:
```
$ cargo -V
cargo 1.51.0 (43b129a20 2021-03-16)
$ rustc -V
rustc 1.51.0 (2fd73fabe 2021-03-23)
$ probe-run -V
0.2.1
supported defmt version: 0.2
```


Without release
---------------
```
$ cargo run
    Finished dev [optimized + debuginfo] target(s) in 0.01s
     Running `probe-run --chip STM32F103CB target/thumbv7m-none-eabi/debug/i2c-test`
  (HOST) INFO  flashing program (10.18 KiB)
  (HOST) INFO  success!
────────────────────────────────────────────────────────────────────────────────
       0 INFO  Clock Setup done
└─ i2c_test::__cortex_m_rt_main @ src/main.rs:54
       1 INFO  I2C Setup done
└─ i2c_test::__cortex_m_rt_main @ src/main.rs:78
       2 INFO  Done
└─ i2c_test::__cortex_m_rt_main @ src/main.rs:85
^Cstack backtrace:
   0: i2c_test::__cortex_m_rt_main
   1: main
        at src/main.rs:32
   2: Reset
        at /home/sebastian/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs:526
```

With release
------------
```
$ cargo run --release
$ cargo run --release
    Finished release [optimized + debuginfo] target(s) in 0.01s
     Running `probe-run --chip STM32F103CB target/thumbv7m-none-eabi/release/i2c-test`
  (HOST) INFO  flashing program (7.64 KiB)
  (HOST) INFO  success!
────────────────────────────────────────────────────────────────────────────────
       0 INFO  Clock Setup done
└─ i2c_test::__cortex_m_rt_main @ src/main.rs:54
       1 INFO  I2C Setup done
└─ i2c_test::__cortex_m_rt_main @ src/main.rs:78
^Cstack backtrace:
   0: stm32f1xx_hal::i2c::I2c<I2C,PINS>::wait_after_sent_start
        at /home/sebastian/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f1xx-hal-0.7.0/src/i2c.rs:353
   1: stm32f1xx_hal::i2c::BlockingI2c<I2C,PINS>::send_start_and_wait
        at /home/sebastian/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f1xx-hal-0.7.0/src/i2c.rs:424
   2: stm32f1xx_hal::i2c::BlockingI2c<I2C,PINS>::write_without_stop
        at /home/sebastian/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f1xx-hal-0.7.0/src/i2c.rs:436
   3: <stm32f1xx_hal::i2c::BlockingI2c<I2C,PINS> as embedded_hal::blocking::i2c::Write>::write
        at /home/sebastian/.cargo/registry/src/github.com-1ecc6299db9ec823/stm32f1xx-hal-0.7.0/src/i2c.rs:463
   4: i2c_test::__cortex_m_rt_main
        at src/main.rs:80
   5: main
        at src/main.rs:32
   6: Reset
        at /home/sebastian/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.13/src/lib.rs:526
```
