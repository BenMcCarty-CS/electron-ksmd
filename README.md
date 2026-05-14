# electron-ksmd

> A Linux daemon that reduces Electron app RAM usage via kernel same-page merging (KSM)

**Status: early development** — contributions and feedback welcome.

---

## The problem

Slack, VS Code, Discord, Obsidian. If you're a developer, you're probably running several Electron apps at once. Each one ships its own bundled copy of Chromium — meaning 3–5 separate browser engines loaded into memory simultaneously, consuming gigabytes of RAM for pages that are largely identical across processes.

This daemon doesn't patch binaries or require kernel modules. It uses a feature already in the Linux kernel — **Kernel Same-page Merging (KSM)** — to deduplicate those identical memory pages across all running Electron processes transparently.

---

## How it works

KSM is a Linux kernel subsystem that scans memory pages across processes, identifies identical ones, and merges them into a single copy-on-write page. Applications see no difference; the kernel just hands them the same physical page.

`electron-ksmd` automates the setup:

1. Polls `/proc` to detect running Electron processes
2. Calls `madvise(MADV_MERGEABLE)` on their memory regions to opt them into KSM
3. Tunes `/sys/kernel/mm/ksm/` parameters for optimal deduplication
4. Tracks savings and exposes them via a local REST API

---

## Project status

This project is under active development. The following is the planned roadmap:

- [ ] Electron process detection via `/proc`
- [ ] KSM manager — read/write `/sys/kernel/mm/ksm/`
- [ ] `madvise` integration
- [ ] Metrics collection
- [ ] REST API (`axum`)
- [ ] systemd service file
- [ ] Benchmarks
- [ ] Config file support
- [ ] CLI (`clap`)

---

## Building from source

**Prerequisites:** Rust 1.78+, Linux kernel 3.16+, KSM enabled in your kernel.

```bash
git clone https://github.com/you/electron-ksmd
cd electron-ksmd
cargo build --release
```

Enable KSM in your kernel (one-time, requires root):

```bash
sudo modprobe ksm
echo 1 | sudo tee /sys/kernel/mm/ksm/run
```

Run the daemon:

```bash
./target/release/electron-ksmd start
```

---

## Tech stack

Built in Rust. Key crates:

| Crate | Purpose |
|---|---|
| `tokio` | Async runtime |
| `sysinfo` | Process monitoring |
| `nix` | Unix syscalls (`madvise`, signals) |
| `axum` | REST API |
| `clap` | CLI |
| `serde` | Config and API serialization |
| `tracing` | Structured logging |
| `anyhow` | Error handling |

---

## Limitations

- **Linux only** — KSM is a Linux kernel feature. macOS and Windows are not supported.
- **KSM must be enabled once at boot** — requires a one-time root command (or a systemd unit).
- **Savings vary** — results depend on which Electron apps are running, their versions, and your workload. More apps = more savings.
- **Early stage** — this project is a work in progress. Expect rough edges.

---

## Why not just use Tauri?

[Tauri](https://tauri.app/) solves the same problem differently — it's a framework for building *new* apps that use the OS's native WebView instead of bundling Chromium. That's the right long-term answer, but it requires app developers to migrate.

`electron-ksmd` works on *existing* Electron apps you already have installed, with no changes required from the app developer.

---

## Contributing

This project is in early development and good first areas to contribute are:

- Improving Electron process detection heuristics
- Testing on different Linux distributions
- Adding support for detecting more Electron app signatures

Please run `cargo fmt` and `cargo clippy` before submitting a PR.

---

## License

MIT © 2025