# tpt-fat32 TODO

Maintained by **TPT Solutions**. Licensed `MIT OR Apache-2.0`. Design: see [SPEC.md](SPEC.md).
Tick a box (`[x]`) when the task is done.

## Phase 0: Repo setup
- [ ] Workspace `Cargo.toml` (members: `crates/*`, resolver 2, shared package metadata)
- [x] `LICENSE-MIT` and `LICENSE-APACHE` (copyright TPT Solutions)
- [x] `SPEC.md` (from spec.txt, with the write-order fix)
- [ ] `README.md` (overview, goals, status, dual-license notice)
- [ ] `.gitignore`, `rustfmt.toml`, `clippy.toml`, `rust-toolchain.toml`
- [ ] `git init` and first commit
- [ ] Decide MSRV and pin it in CI
- [ ] Create the 3 crates: `tpt-fat32`, `tpt-fat32-repair`, `tpt-fat32-fuzz`
- [ ] `#![no_std]` and `#![forbid(unsafe_code)]` in `tpt-fat32` (parsing code)
- [ ] Feature flags: `alloc` (optional cache), `async`, `std` (tests only)
- [ ] Choose a zero-copy approach (`zerocopy` crate or safe manual parsing)

## Phase 1: Foundation and zero-copy parsing (Weeks 1-3)
- [ ] `error.rs`: error enum (`Corrupted`, `Io`, `NotFound`, `NoSpace`, ...), no panics
- [ ] `block.rs`: `BlockDevice` trait (read, write, flush), sync version
- [ ] `block.rs`: async version of the trait (feature `async`)
- [ ] RAM-disk `BlockDevice` implementation for tests
- [ ] `bpb.rs`: zero-copy BPB parser with full validation (sector size, cluster count, FAT32 detection)
- [ ] `bpb.rs`: FSInfo parser (signatures, free count and next-free hints treated as untrusted)
- [ ] `fat.rs`: FAT entry parsing (mask the high 4 bits; free, bad, EOF and data markers)
- [ ] `dir.rs`: 32-byte directory entry parser
- [ ] `dir.rs`: LFN entries and 8.3 short-name checksum
- [ ] `dir.rs`: short-name generation (`~N` tail, collision handling)
- [ ] Time module: FAT 1980 epoch to Unix timestamps (both directions, range checks)
- [ ] Unit tests for every parser (valid, boundary and malformed inputs)
- [ ] Test images made with `mkfs.vfat` (various cluster sizes)

## Phase 2: Core driver and allocator (Weeks 4-6)
- [ ] Volume manager: mount, BPB validation, FAT geometry, FSInfo handling
- [ ] LRU sector cache for FAT and directory sectors (fixed-size buffers; optional `alloc` variant)
- [ ] Cache write-back and flush ordering (reverse order)
- [ ] Sequential/contiguous cluster allocator
- [ ] FSInfo write minimization (update on clean unmount or at intervals only)
- [ ] FAT chain walker (bounded by `total_clusters`)
- [ ] Directory walker (including LFN and `.` / `..` entries)
- [ ] File handle: read, write, seek, truncate, set length
- [ ] Dir handle: create file, create dir, delete, rename, list
- [ ] Write-ordering state machine: data, FAT chain (EOF), directory entry, flush in reverse
- [ ] Mirror both FAT copies correctly
- [ ] Timestamps on create, modify and access
- [ ] `lib.rs` public API (`Volume`, `File`, `Dir`)
- [ ] `tests/loopback.rs`: full read/write/create/delete tests on a RAM disk
- [ ] Interop check: images written by tpt-fat32 pass `fsck.vfat`

## Phase 3: Verification and repair (Weeks 7-9)
### Kani proofs
- [ ] Set up Kani (harness layout, CI job)
- [ ] Proof: FAT chain walker terminates and stays in bounds
- [ ] Proof: LFN checksum matches the 8.3 short name
- [ ] Proof: parsers never panic
- [ ] Proof: write-ordering state machine never leaves FAT and directory inconsistent if interrupted

### Repair crate (`tpt-fat32-repair`)
- [ ] `checker.rs`: invariant checker (FAT vs directory mismatches, cycles, bad clusters)
- [ ] Detect cross-linked files
- [ ] `reclaimer.rs`: reclaim lost chains into a `FOUND.000` directory
- [ ] Safe truncation of corrupted files
- [ ] `main.rs`: CLI (`check`, `repair`, `--dry-run`, image or raw device path)
- [ ] Repair never modifies valid data (tests)

### Fuzzing and crash testing
- [ ] `tpt-fat32-fuzz`: `fuzz_bpb.rs`
- [ ] Fuzz targets for FAT entries, directory entries, LFN and the full mount path
- [ ] Fuzz corpus seeded from real images
- [ ] Power-loss simulation: a block device that fails at every write, then remount and check
- [ ] Confirm the driver returns `Error::Corrupted` and never panics on malformed media

## CI (GitHub Actions)
- [ ] Matrix: Linux, Windows, macOS
- [ ] `cargo fmt --check` and `cargo clippy -D warnings`
- [ ] `cargo test --workspace`
- [ ] `no_std` embedded target build (e.g. `thumbv7em-none-eabihf`)
- [ ] Kani job
- [ ] Fuzz smoke run (short time budget)
- [ ] `cargo deny` / license check (MIT OR Apache-2.0 compatible only)

## Release
- [ ] `CONTRIBUTING.md` (including the dual-license contribution clause)
- [ ] `CHANGELOG.md`
- [ ] Rustdoc for all public APIs and `#![deny(missing_docs)]`
- [ ] Examples (RAM disk, SD card via `embedded-sdmmc`-style adapter)
- [ ] Interop tests against Windows, Linux and macOS images
- [ ] Crate metadata: `license = "MIT OR Apache-2.0"`, authors, repository, keywords
- [ ] `cargo publish --dry-run` for each crate
- [ ] Publish to crates.io and tag v0.1.0
