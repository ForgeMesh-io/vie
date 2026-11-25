# VIE — community fork of XCL

VIE is the ForgeMesh community fork of [XCL](https://github.com/xcl-dev/xcl). Its goal is to run lawfully owned Xbox Series X|S games on **Windows, Linux and macOS** through a native host runtime. VIE starts with XCL's Rust runtime, not a separate compatibility layer. It is not affiliated with or endorsed by XCL's maintainers, Microsoft, or game publishers.

| Platform | Status |
| --- | --- |
| Windows x86-64 | Inherited XCL baseline: native x86-64 game execution, with Direct3D 12 and Windows system services. Upstream compatibility reports are not VIE test results. |
| Linux x86-64 | Planned native Linux runtime and graphics/system backends. Running the Windows executable through Wine or Proton does not satisfy this milestone. |
| macOS | Planned native macOS runtime. Intel Macs may be able to execute x86-64 game code directly if host constraints allow it. Apple Silicon cannot run the games' x86-64 instructions directly; it would need an explicitly supported translation path. Neither target has been validated. |

## Windows baseline

The imported code uses Windows 10/11 x64, Rust with MSVC, the Windows SDK and a suitable D3D12 GPU. See the [pinned upstream README](https://github.com/xcl-dev/xcl/blob/e74dda2d4b5c28a81cfa971d14bc417a6be966aa/README.md) for prerequisites, commands and known limits. The workspace and `xcl` executable keep their upstream names. The fork is not a sandbox: do not run untrusted game code on your host.

## Contribute

See the [roadmap](ROADMAP.md), [contribution guide](CONTRIBUTING.md), and [VIE coordination issue](https://github.com/ForgeMesh-io/vie/issues/1). The first porting tasks are read-only and do not require a game or an execution image. Start with a self-built test title; game files, console SDK material and keys stay out of GitHub. The [original research audit](research/MATRIX.md) is historical context, not the fork's roadmap.

## Licence and upstream

**GPL-2.0-only**, like XCL. VIE preserves XCL's history, [licence](LICENSE), and [third-party notices](THIRD_PARTY.md). The initial upstream baseline is [`e74dda2`](https://github.com/xcl-dev/xcl/commit/e74dda2d4b5c28a81cfa971d14bc417a6be966aa). See [LEGAL.md](LEGAL.md) before importing code or moving that baseline forward.
