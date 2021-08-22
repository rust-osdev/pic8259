# `pic_8259`

Abstractions for 8259 and 8259A Programmable Interrupt Controllers (PICs).

This project is a fork of the [`pic8259_simple` crate](https://github.com/emk/toyos-rs/tree/master/crates/pic8259_simple) created by [@emk](https://github.com/emk).

**Things we may not handle very well yet include:**

1. Dealing with spurious interrupts.
2. Non-standard configurations.

This code is based on the [OSDev Wiki PIC notes][PIC], but it's not a
complete implementation of everything they discuss.  Also note that if you
want to do more sophisticated interrupt handling, especially on
multiprocessor systems, you'll probably want to read about the newer
[APIC] and [IOAPIC] interfaces.

[PIC]: http://wiki.osdev.org/8259_PIC
[APIC]: http://wiki.osdev.org/APIC
[IOAPIC]: http://wiki.osdev.org/IOAPIC

## Using

This is a very basic interface to the 8259 and 8259A interrupt controllers,
which are used on single processor systems to pass hardware interrupts to
the CPU.

To use this crate, add it to your `Cargo.toml` file, along with an
appropriate kernel-space mutex implementation such as `spin`:

```toml
[dependencies]
pic8259 = "0.10.0"
spin = "0.9.0"
```

You can then declare a global, lockable `ChainedPics` object as follows:

```rust

use pic8259::ChainedPics;
use spin::Mutex;

// Map PIC interrupts to 0x20 through 0x2f.
static PICS: Mutex<ChainedPics> =
    Mutex::new(unsafe { ChainedPics::new(0x20, 0x28) });
```

To perform runtime PIC intialization, call `initialize` before enabling
interrupts:

```rust
PICS.lock().initialize();
```

When you've finished handling an interrupt, run:

```rust
PICS.lock().notify_end_of_interrupt(interrupt_id);
```

It's safe to call `notify_end_of_interrupt` after every interrupt; the
`notify_end_of_interrupt` function will try to figure out what it needs to
do.

All public PIC interfaces are `unsafe`, because it's really easy to trigger
undefined behavior by misconfiguring the PIC or using it incorrectly.

## Crate Feature Flags

- `nightly` - Uses features that are only usable on nightly Rust. Enabled by default.
- `stable` - Enable this feature flag to build this crate on stable Rust. You have to add `default-features = false, features = ["stable"]` to your `Cargo.toml`.

## Licensing

Licensed under the [Apache License, Version 2.0][LICENSE-APACHE] or the
[MIT license][LICENSE-MIT], at your option.

[LICENSE-APACHE]: http://www.apache.org/licenses/LICENSE-2.0
[LICENSE-MIT]: http://opensource.org/licenses/MIT
