# VIE roadmap

1. **Windows x86-64 baseline — VIE-0013.** Preserve XCL's Windows build and tests. Establish exact-SHA evidence for a self-built title, including launch and the first rendered frame on a capable GPU. Keep GPU and game tests distinct from hosted CI.
2. **Linux x86-64 — VIE-0014.** Separate shared Rust runtime code from Win32 host services. Plan a native loader and a graphics backend, with Vulkan as a candidate. First prove native process startup with a synthetic title, then render a frame without Wine or Proton.
3. **macOS — VIE-0015.** Measure the loader, ABI, memory, and graphics constraints for Intel and Apple Silicon separately. Consider Metal for graphics. An arm64 host binary is native to Apple Silicon; an x86-64 game running via translation is not direct native game execution. Prove startup and a rendered frame on each supported host separately.

Keep the imported Windows path working as ports land. Before calling a game playable on a platform, record the VIE commit, host/CPU/GPU category, and observed progression (boot, menus, in-game, playable). A synthetic title does not establish retail compatibility.
