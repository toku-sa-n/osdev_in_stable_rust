# Write a UEFI bootloader in stable Rust

This article is based on [my blog post](https://tokuchan3515.hatenablog.com/entry/2021/08/14/163511).

We follow the official target called `x86_64-unknown-uefi`. See [this](https://github.com/rust-lang/rust/blob/d0a10b205608ad91280fb10495c18eb1c9110c89/compiler/rustc_target/src/spec/uefi_msvc_base.rs) and [this](https://github.com/rust-lang/rust/blob/d0a10b205608ad91280fb10495c18eb1c9110c89/compiler/rustc_target/src/spec/x86_64_unknown_uefi.rs). Also, please check [the discussion](https://github.com/rust-lang/rust/pull/56769) of the pull request which added the target.

## Installation of the Windows target

To create a PE32+ binary, we install a Windows target.

```
rustup target add x86_64-pc-windows-gnu
```

## `.cargo/config.toml`

```toml
[build]
target = "x86_64-pc-windows-gnu"

[target.x86_64-pc-windows-gnu]
rustflags = [
    "-C", "link-args=/entry:efi_main /subsystem:efi_application",
    "-C", "no-redzone=y",
    "-C", "linker=lld-link",
]
```
Strictly speaking, `/entry:efi_main` is not necessary. However, The official `x86_64-unknown-uefi` target [spcfiy this](https://github.com/rust-lang/rust/blob/d0a10b205608ad91280fb10495c18eb1c9110c89/compiler/rustc_target/src/spec/uefi_msvc_base.rs#L21), so we follow it.

We specify `linker=lld-link` to use `lld-link` as a linker to use options selected in `link-args`. On Ubuntu, run these commands to install it.
```
sudo apt-get install lld
sudo ln -s /usr/bin/lld /usr/bin/lld-link
```

## `src/main.rs`
```rust
#![no_std]
#![no_main]

#[no_mangle]
fn efi_main(h: uefi::Handle, mut st: bootx64::SystemTable) -> ! {
    // ...
}
```
We define `efi_main` as a pseudo entry point using `#![no_main]` and `#![no_mangle]`.

## Building
Running `cargo build` generates `bootx64.exe` in `target/x86_64-pc-windows-gnu/debug`. You can use this binary as a bootloader by renaming it to `bootx64.efi`.

## Appendix: libraries
The [`uefi`](https://crates.io/crates/uefi) crate is one of the crates which is helpful to handle UEFI features. However, it uses features available only in nightly Rust, so we cannot build it in stable Rust.

The [`r-efi`](https://crates.io/crates/r-efi) crate is one of the crates we can build on stable Rust. This crate defines functions and definitions specified in the UEFI specification as it is. The crate authors do not intend to use the crate directly, and the users of this crate are supposed to [write some wrappers](https://github.com/r-efi/r-efi/blob/5b63ca81d8d5f16b5d89cddf9d3254db323aa2e4/src/lib.rs#L9-L14).
