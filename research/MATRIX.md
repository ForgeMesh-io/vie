# VIE research matrix: Xbox Series X|S compatibility layers and recompilers

> Historical audit from the former research project, published under its [original Apache-2.0 licence](https://github.com/ForgeMesh-io/vie/blob/a5e8a5b312e5a2a607dfdc171922d73b891b813b/LICENSE). Its recommendations do not govern the XCL fork; see [LEGAL.md](../LEGAL.md) and [ROADMAP.md](../ROADMAP.md).

Audited: 2026-11-25. Read-only audit through the GitHub REST API (`gh api`, account FRGMesh) and the freedesktop GitLab API. No binary was downloaded or run. Per-repo details and sources are in [`repos.json`](repos.json).

**Reading rules**

- Status, performance and shader-count claims come from each project's README. They were **not verified** by building or running anything. Treat them as claims.
- "Findings" are facts taken from fetched data. Text marked **Recommendation** is the auditor's judgement for VIE.
- Star, commit and contributor counts are as reported by the API on the audit date. Commit and contributor counts are estimates taken from pagination `Link` headers.

---

## 1. Summary table

### 1.1 Requested repositories

| Repo (resolved) | Exists | License | Approach | Maturity (claimed) | Best reusable parts for VIE | Red flags |
|---|---|---|---|---|---|---|
| xcl-dev/xcl (`ivall/xcl` redirects here) | yes | GPL-2.0-only | Native x64 + HLE; RDNA2 → HLSL → DXIL; Windows D3D12 | working-partial (Minecraft Preview and Unicorn Overlord playable; a newer retail game ~60%) | Behaviour reference only (license). Its THIRD_PARTY.md points to the MIT upstreams it used: AMD machine-readable ISA and Mesa addrlib | Repo transferred to an org created 2026-11-24; 8.1k stars in 5 days; ports AnyPS5 (GPL-2.0-only) code; Windows-only |
| mewiof/Holy-Interoperability | yes | MIT | In-memory import takeover + one runtime DLL; RDNA → host bytecode; Windows | early (first frame presented, nothing drawn) | Clean-room hygiene checker idea (`tools/hygiene/check.py`), import-coverage `holy inspect` | 1 author, about 1 day of history |
| californiaAlmonds/OpenScarlett | yes | MIT | Native x64 + 33 shim DLLs; RDNA2 SSA IR → **SPIR-V 1.6** and DXIL; D3D12 and **Vulkan 1.4** back ends | working-partial (claims Minecraft Preview at 60 fps) | Only MIT code found with a direct RDNA2 → SPIR-V emitter, a Vulkan back end and a CPU wave32/64 interpreter oracle (`src/gpu/rdna2_*`, `vulkan_backend.cpp`) | Hype-heavy README; monolithic 200–760 KB source files; ships prebuilt win64 zips; provenance not documented |
| OnlyMoisties/GTA6-BootTest | yes | **none** (all rights reserved) | GTA VI-specific stand-in DLLs + debugger harness; Python RDNA2 → HLSL | prototype (first UI screen) | Nothing (no license) | No license; prebuilt DLL zip; hard-coded game offsets |
| brcly/TuxBox | yes | **Apache-2.0** | Native x64 + minimal HLE on **stock Valve Proton** (vkd3d-proton) + Vulkan CBV layer; RDNA2 → HLSL → DXIL → (dxil-spirv) SPIR-V | working-partial (Matrix Awakens reaches the cinematic; GTA VI reaches the story intro on Linux, not playable) | Directly reusable: runtime crates (`runtime/*`), `tuxbox-xshader` RDNA decoder and HLSL emitters, the Vulkan CBV layer, Proton workflow, evidence/test methodology | `legacy-hlsl` default feature derives from TomBox, which has no LICENSE file; fast-growing history |
| R0Y00/Xbox2Windows | yes | MIT | Static x64 → C++ AOT research + offline RDNA → DXIL + shims; Windows | early (full-game startup unverified) | **Only real x64 static recompiler** (`tools/cpu_*_aot.py`, `tests/cpu`); differential test methodology; SM 6.0 mesh fallback drawn on GTX 1060 | 4,460 commits and 112 contributors in 5 days; GpuCheck binary release |
| OpenCommunityEdition/OpenScarlett | yes | AGPL-3.0-or-later (layer) + MIT (tools/tests/ABI data, per-file SPDX) | Replacement DLLs; GFX10 → HLSL → DXIL; Linux via Wine + patched vkd3d-proton/dxil-spirv | working-partial (Minecraft Preview ~60 fps, also on Linux) | MIT-tagged tools and tests; `docs/provenance.md` practice; patches against MIT dxil-spirv and LGPL vkd3d-proton as references | AGPL core; no description; 2 contributors |
| ThatRecompDev/five-star-recomp | yes | GPL-3.0-or-later | GTA VI 1.0.2.1 loader + RDNA → HLSL + own DXIL back end; C#/.NET 10; Windows | early ("does not run yet"; cutscene at 30 fps, lighting missing) | Behaviour reference only | Title-specific; uses Leonida facts (Leonida is now 404) |
| fablinoo/palmwave | **no (404)** | n/a | n/a | none | n/a | 404; a same-named XCL derivative exists at schizoidcock/palmwave |
| Marvisak/VIctory (`marvisak/VIctory`) | yes | GPL-3.0-or-later | Native x64 + Rust HLE; D3D12.X → D3D12; console GPU code → DXIL (AOT cache); Windows | working-partial (GTA VI world shown at 1–2 s/frame, unlit; tested on RX 6700 XT) | Behaviour reference only; crate split (`victory-{pe,loader,hle,gpu,shader,formats,pipeline}`) is a useful layout model | 12k stars and 105 contributors in about 2 days; README warns of GPU hangs and TDR |
| LooperSalty/AnyXboxX | yes | GPL-3.0 | Python/Unicorn emulation prototype + launcher around a pinned VIctory fork | working-partial via VIctory fork (4.8–7 fps world claimed); own emulator is a prototype | Only accessible copy of Leonida's RDNA2 → SPIR-V tools (`third_party/leonida`, GPL-3.0), as a reference | Hub of copied GPL code; claims rest on a third-party fork |
| fxceless/victory-fork (`fxceless/victory-gta6` redirects here) | yes | GPL-3.0-or-later | VIctory fork with perf/render fixes | working-partial (2–5 fps world on RTX 4070) | Behaviour reference only | Not flagged as a GitHub fork; README says changes were developed with Claude Code |
| boykopovar/AnyPS5 (PS5 reference) | yes | GPL-2.0-only | PS5 ELF relinker (no emulation) + prx reimplementations + **RDNA2 → SPIR-V** recompiler → Vulkan | mature (by size and activity) | Architecture reference for RDNA2 → SPIR-V (`core/shader/recompiler/{RdnaDecoder,IntermediateRepresentation,SpirvBackend}`) and for AMD-only CPU instruction lowering (`--to-intel`) | 61k stars and 50k commits in about 4 months; prebuilt relinker binaries; PS5, not Xbox |

### 1.2 Reference projects

| Project | License | Where the relevant code lives | What VIE can reuse |
|---|---|---|---|
| shadps4-emu/shadPS4 | GPL-2.0-or-later | `src/shader_recompiler/{frontend,ir,backend/spirv}`, `src/video_core`, `src/core/cpu_patches.cpp` | Design reference only (GCN → SPIR-V pipeline, structured control flow, CPU patching). No code |
| Mesa RADV + ACO (gitlab.freedesktop.org/mesa/mesa; no GitHub mirror found) | Mostly MIT (per-file SPDX) | `src/amd/vulkan` (RADV), `src/amd/compiler` (ACO, `aco_opcodes.py`, `README-ISA.md`), `src/amd/addrlib` | Runtime driver. MIT code is reusable as RDNA semantics/encoding reference and for addrlib detiling tables |
| HansKristian-Work/vkd3d-proton | LGPL-2.1 | `libs/vkd3d`, dxil-spirv submodule | Use unmodified as a separate DLL (D3D12 → Vulkan) if VIE targets desktop D3D12 |
| HansKristian-Work/dxil-spirv (discovered) | MIT (`LICENSE.MIT`) | `cfg_structurizer*.cpp`, `dxil_converter.cpp` | CFG structurizer reusable for unstructured → structured SPIR-V control flow |
| CachyOS/proton-cachyos | Multiple (BSD-style `LICENSE.proton` + component licenses; GitHub: NOASSERTION) | Code on versioned branches `cachyos_<ver>_<date>/main`; default `cachyos_main` holds only docs. Submodules: wine-cachyos, vkd3d-proton, vkd3d, dxvk | Host runtime (Wine PE loader + vkd3d-proton). Pin an exact branch or release |

### 1.3 Discovered projects (verified lightly)

| Repo | Exists | License | Note |
|---|---|---|---|
| ItsNotRexed/Leonida | **no (404)** | GPL-3.0 (per copies) | RDNA2 → SPIR-V (Python). Copies: LooperSalty/AnyXboxX `third_party/leonida`, clevervi/palmetto |
| clevervi/palmetto | yes | GPL-3.0 | "Private fork" of Leonida. Claims all 26,503 GTA VI shaders → SPIR-V pass `spirv-val`; README: "Early. The game does not boot yet." |
| schizoidcock/palmwave | yes | GPL-2.0 | XCL derivative branded "Moon AI / Stellarr Studio" |
| drycheetah84/TomBox | yes | **No LICENSE file**; crate manifests say `MIT OR Apache-2.0` | Origin of TuxBox `legacy-hlsl` |
| omonkeyz/xbox-emulator | yes | GPL-3.0 | TomBox + rage9 notes; AnyXboxX submodule |
| WinDurango/WinDurango | yes | LGPL-3.0 | Xbox One layer; reference only per XCL |
| xodus-gaming/xodus / xgameruntime | yes | GPL-3.0 / LGPL-2.1 | xgameruntime.dll for Wine (could load as a separate DLL) |
| Freminet/ScarlettX | yes | none | Not reusable |
| p-portela/SCARAB | yes | GPL-3.0 | 16 commits |
| Ahmet-Dedeler/Percy6Stars (`6Stars` redirects here) | yes | Custom non-commercial | Not reusable |
| realdody/americas-recomp | **no (404)** | n/a | Referenced by TuxBox as an AnyPS5-based experiment |

---

## 2. Per-component matrix

"Apache-OK" means the code can be incorporated into Apache-2.0 VIE (Apache-2.0, MIT, or `MIT OR Apache-2.0`).

| Component | Who provides it (license) | Best Apache-OK candidate | Best overall reference (any license) | License constraint |
|---|---|---|---|---|
| CPU: native x64 execution / PE loading | Every Xbox layer runs title code natively. PE loading: XCL, VIctory, Holy (MIT), five-star; TuxBox delegates it to Wine/Proton (Apache-2.0) | **TuxBox approach** (let Proton/Wine load the PE) | VIctory `victory-pe`/`victory-loader`; XCL `pe.rs`/`loader.rs` | GPL refs: facts only |
| CPU: recompilation / instruction patching | Xbox2Windows x64 → C++ AOT (MIT); californiaAlmonds VEH emulation of Zen 2-only instructions (MIT); AnyPS5 `--to-intel` lowering (GPL-2.0); shadPS4 `cpu_patches` (GPL-2.0+) | Xbox2Windows (MIT), californiaAlmonds VEH list (MIT) | AnyPS5 relinker | Not needed for AMD Zen hosts per the native-execution designs; relevant only for non-Zen or Polaris-era CPUs (open question) |
| GPU: RDNA2 decode / ISA tables | AMD machine-readable ISA (MIT, used by XCL/TuxBox); Mesa ACO `aco_opcodes.py` (MIT); OCE generates tables from LLVM; TuxBox decoder (Apache-2.0); californiaAlmonds disassembler (MIT) | **TuxBox `tuxbox-xshader` decoder + AMD MIT ISA XML** | AnyPS5 `RdnaDecoder` | — |
| GPU: RDNA2 → **SPIR-V** | californiaAlmonds (MIT, claimed); AnyPS5 (GPL-2.0-only); Leonida via AnyXboxX/palmetto (GPL-3.0); OCE and TuxBox reach SPIR-V only indirectly through dxil-spirv | **californiaAlmonds `src/gpu/rdna2_lifter.cpp` + SPIR-V emitter** (quality unverified) | AnyPS5 `SpirvBackend` (most mature); palmetto (claims full GTA VI corpus passes `spirv-val`) | AnyPS5/Leonida code unusable; behaviour and test corpora only |
| GPU: RDNA2 → HLSL/DXIL | XCL, VIctory, five-star, OCE, Xbox2Windows (MIT), TuxBox (Apache-2.0), Holy (MIT), GTA6-BootTest (none) | TuxBox HLSL emitters | XCL (claims 26,698/26,698) | — |
| GPU: control-flow structurization | dxil-spirv `cfg_structurizer` (MIT); shadPS4 `structured_control_flow` (GPL); Leonida `cfg_structure.py` (GPL-3.0); VIctory EXEC-free structurization (GPL-3.0) | **dxil-spirv cfg_structurizer (MIT)** | shadPS4 | — |
| GPU: wave64 on wave32 | XCL `docs/wave64-on-wave32.md` (GPL); Leonida `wave64_*.py` (GPL-3.0); californiaAlmonds "Wave64-on-32" (MIT); palmetto says wave64 is exact on AMD 64-wide subgroups | californiaAlmonds | XCL doc | RADV: maxSubgroupSize = 64 on all GFX; minSubgroupSize = 64 before GFX10, so Polaris is wave64-native |
| Graphics API: D3D12.X → host | D3D12.X → D3D12: XCL, VIctory, five-star, OCE, TuxBox (Apache-2.0), Holy (MIT), Xbox2Windows (MIT). D3D12.X → **Vulkan direct**: californiaAlmonds only (MIT) | **TuxBox `runtime/d3d12`** (on vkd3d-proton) | VIctory `victory-gpu` | vkd3d-proton LGPL-2.1: use as a separate DLL |
| Graphics: texture detiling | Mesa addrlib (MIT, used by XCL); californiaAlmonds `xg_gpu_untile.cpp` (MIT); TuxBox `xg-layout` (Apache-2.0) | **Mesa addrlib + TuxBox xg-layout** | — | — |
| Graphics: ray tracing / BVH | XCL `bvh.rs` (GPL); TuxBox `runtime/bvh` (Apache-2.0); californiaAlmonds ray-query SPIR-V (MIT) | TuxBox `runtime/bvh` | XCL | RADV ray query requires gfx10.3+ or RT emulation (drirc `emulate_rt`) |
| OS HLE (kernelx, xmem, XGameRuntime, acphal) | All layers. Apache-OK: TuxBox, Holy, californiaAlmonds, Xbox2Windows. Separate-DLL option: xodus xgameruntime (LGPL-2.1) | **TuxBox runtime crates** | XCL (GDK table of 396 functions), OCE per-edition ABI layouts | — |
| Filesystem / packages / DirectStorage / BCPack | XCL, VIctory (GPL); TuxBox `xcrdapi` (Apache-2.0); californiaAlmonds CoW VFS + BCPack (MIT); Holy (MIT); OCE uses the SDK DLL for BCPack | TuxBox `xcrdapi`; californiaAlmonds BCPack | VIctory `victory-formats` | VIctory BCPack is GPL-3.0 (XCL obtained an author exception for GPL-2.0) |
| Audio (XAudio2, XMA, Opus) | XCL (FFmpeg XMA, LGPL-2.1+); VIctory (GPL); TuxBox forwards to Proton XAudio2 (Apache-2.0); californiaAlmonds (MIT) | **TuxBox forward-to-Proton** | XCL | FFmpeg LGPL: link as a separate library only |
| Input (GameInput) | XCL, OCE (SDL/DualSense), VIctory, TuxBox `runtime/gameinput` (Apache-2.0), californiaAlmonds (MIT) | TuxBox `runtime/gameinput` | OCE | — |
| Tooling / tests | Xbox2Windows differential native/mapped/AOT tests (MIT); TuxBox evidence JSON + Lanes homebrew + GPU self-test (Apache-2.0); californiaAlmonds CPU wave interpreter oracle (MIT); OCE `provenance.md`; Holy hygiene checker (MIT); AnyPS5 hw-oracle (GPL) | **TuxBox evidence pipeline + Xbox2Windows differential methodology** | AnyPS5 hw-oracle | — |

---

## 3. License strategy

### Findings

- **Directly reusable in Apache-2.0 VIE:** TuxBox (Apache-2.0); Holy-Interoperability, californiaAlmonds/OpenScarlett and Xbox2Windows (MIT); Mesa RADV/ACO/addrlib (MIT per checked files); dxil-spirv (MIT); AMD machine-readable ISA (MIT, per XCL and TuxBox notices); OCE files that carry MIT SPDX tags.
- **Usable only as separate components, never merged into VIE source:** vkd3d-proton (LGPL-2.1), Wine/Proton-CachyOS components (LGPL and others), xodus xgameruntime (LGPL-2.1), FFmpeg (LGPL-2.1+).
- **Code not reusable (facts and behaviour only):** XCL, palmwave, AnyPS5 (GPL-2.0-only); shadPS4 (GPL-2.0+); VIctory, victory-fork, five-star-recomp, AnyXboxX, Leonida copies, palmetto, xodus, SCARAB, omonkeyz (GPL-3.0); OCE layer DLLs (AGPL-3.0).
- **No license, so not reusable:** GTA6-BootTest, Freminet/ScarlettX. Percy6Stars uses a custom non-commercial license.
- **Ambiguous:** drycheetah84/TomBox has no LICENSE file but declares `MIT OR Apache-2.0` in crate manifests. TuxBox's default `legacy-hlsl` feature depends on it.
- XCL's own THIRD_PARTY.md states Apache-2.0 is incompatible with its GPL-2.0-only license.
- Cross-contamination risk: XCL ports AnyPS5 code, palmwave derives from XCL, AnyXboxX vendors Leonida and VIctory, and omonkeyz relicenses TomBox-derived code to GPL-3.0. "Inspired by" lineages are common across the field.

### Recommendations

1. **Recommendation:** Base VIE on Apache/MIT sources only: TuxBox, californiaAlmonds, Xbox2Windows, Holy, Mesa, dxil-spirv and the AMD ISA. Keep a `THIRD_PARTY.md` listing the origin, license and commit of every imported file, following OCE's `provenance.md` practice.
2. **Recommendation:** Treat every GPL/AGPL repo as off-limits for copying. Anyone who has read a GPL implementation should document facts (ABI layouts, behaviour) in prose specs, and those specs should be reimplemented separately, following the clean-room rule TuxBox REFERENCES.md already states.
3. **Recommendation:** Do not ship TuxBox's `legacy-hlsl` feature until the TomBox authors confirm the license in writing or a LICENSE file is added. Build with `--no-default-features` in the meantime.
4. **Recommendation:** Load LGPL components (vkd3d-proton, Wine, FFmpeg-based decoders) only as separate, unmodified or separately published libraries.
5. **Recommendation:** Run a provenance check (license header, SPDX tag, contributor history) on californiaAlmonds code before importing it, because the repo does not document where its GDK ABI knowledge came from.

---

## 4. Recommended architecture skeleton for VIE

> Everything in this section is **Recommendation**. Each recommendation cites the finding it is based on.

```
user's game package (outside repo)
        │
        ▼
vie-launcher (Linux)  ── runs title under pinned Proton-CachyOS branch
        │                 [finding: TuxBox runs on stock Proton; CachyOS bundles wine-cachyos + vkd3d-proton]
        ▼
Wine PE loader  ──►  title x86-64 code, native        [finding: all layers execute natively]
        │ imports
        ▼
vie runtime DLLs (Rust, Apache-2.0)
  kernelx / xmem / xgameruntime / acphal / gameinput / xcrdapi / xapu
        [start from TuxBox runtime/* (Apache-2.0); forward to Proton APIs where possible]
        │ D3D12.X
        ▼
vie-d3d12x ─┬─ Phase 1: → desktop D3D12 → vkd3d-proton → RADV   [TuxBox path; proven to GTA VI intro]
            └─ Phase 2: → Vulkan directly (RADV)                [californiaAlmonds has an MIT Vulkan back end;
                                                                 TuxBox ARCHITECTURE.md suggests this later]
        │ shaders
        ▼
vie-shader: RDNA2 decoder (AMD MIT ISA XML; TuxBox decoder)
          → SSA IR → structurizer (dxil-spirv cfg_structurizer, MIT)
          → SPIR-V emitter (new, or reviewed californiaAlmonds code)
          → spirv-val + disk cache
          + CPU wave32/64 interpreter as reference oracle      [californiaAlmonds pattern, MIT]
```

1. **Recommendation, host runtime:** Proton-CachyOS, pinned to an exact `cachyos_<ver>_<date>/main` branch or release tag, because the default branch carries docs only.
2. **Recommendation, phase 1 graphics:** Use TuxBox's D3D12.X → D3D12 layer on vkd3d-proton to reach first frames quickly. It needs the HLSL/DXIL detour and the CBV-offset Vulkan layer (`tools/reference/cbv-layer`).
3. **Recommendation, phase 2 graphics:** Emit SPIR-V directly from RDNA2, as VIE's brief requires. Mature references for this exist only under GPL (AnyPS5, Leonida), so the emitter has to be new code or reviewed MIT code from californiaAlmonds. Use palmetto's claimed corpus result (26,503/26,503 passing `spirv-val`) as a target to match, not as code.
4. **Recommendation, wave size:** Run wave64 shaders as native 64-wide subgroups on AMD, using `VK_EXT_subgroup_size_control` (RADV: DONE; maxSubgroupSize 64). Keep wave64-on-32 emulation only for non-AMD GPUs.
5. **Recommendation, Polaris (RX 580), experimental:** RADV gates `VK_EXT_mesh_shader` behind `gfx_level >= GFX10_3` + NGG and ray query behind hardware BVH or RT emulation, and NGG needs GFX10+. GFX8 therefore has no mesh shaders or native ray query. VIE needs a mesh → compute + vertex fallback (Xbox2Windows reports an SM 6.0 fallback drawn on a GTX 1060) and either RT emulation or RT disabled. Polaris is wave64-native, which keeps wave64 shaders simple.
6. **Recommendation, CPU:** No recompiler in v1. Execute natively, as every working layer does. Keep californiaAlmonds' list of Zen 2-only instructions and Xbox2Windows' AOT tools as fallbacks for non-Zen hosts.
7. **Recommendation, tests:** Adopt TuxBox's evidence-JSON status files and homebrew test title, Xbox2Windows' differential (native vs translated) testing, and a CPU interpreter oracle for every translated RDNA2 instruction.
8. **Recommendation, GTA VI validation:** Track the same milestones the other projects report (boot → menus → story intro → world → playable), with measured fps on a named AMD GPU, so results can be compared with TuxBox, VIctory and XCL.

---

## 5. Open questions / follow-up research tasks

| # | Task | Acceptance criteria |
|---|---|---|
| 1 | Confirm TomBox licensing | A LICENSE file exists in drycheetah84/TomBox, **or** a written statement from its authors licenses `tombox-xshader` under MIT OR Apache-2.0. Linked in VIE THIRD_PARTY.md |
| 2 | Provenance review of californiaAlmonds RDNA2 → SPIR-V and Vulkan code | For `src/gpu/rdna2_lifter.cpp`, `rdna2_ir.cpp`, `vulkan_backend.cpp`: git blame/author list exported, no file contains GPL headers or verbatim matches with AnyPS5/Leonida (checked with a code-similarity tool such as `jscpd` or `dolos`, report attached) |
| 3 | Build-only check of TuxBox on Linux (no game) | `cargo build` and `cargo test` of the TuxBox workspace pass from source with `--no-default-features` on an x86-64 Linux host. Command log committed |
| 4 | Measure the californiaAlmonds SPIR-V emitter on a homebrew corpus | Built from source; ≥ 100 RDNA2 shaders from a self-built GDK sample or TuxBox Lanes; result table with counts of `spirv-val` pass/fail |
| 5 | Polaris feasibility | On an RX 580 with current RADV: `vulkaninfo` output saved; confirms `meshShader=false`, `minSubgroupSize=64`, ray query absent unless `emulate_rt` is set. A compute-based mesh fallback renders one test mesh correctly (image diff ≤ 1%) |
| 6 | Proton-CachyOS pinning | One release tag/branch chosen. VIE CI downloads it by checksum and runs the TuxBox Lanes homebrew title to first presented frame on an RDNA2 GPU |
| 7 | Leonida status | Determine whether ItsNotRexed/Leonida was deleted, renamed or made private (GitHub redirect check repeated weekly for 4 weeks). Result recorded with dates |
| 8 | Independent verification of headline claims | For TuxBox, XCL, VIctory and OCE: each README claim mapped to evidence that can be checked (log, video, evidence JSON, issue). Claims without evidence marked "unverified" in repos.json |
| 9 | Zen 2-only CPU instructions in GTA VI | List of non-baseline x86-64 instructions in the title executable (static scan of the user's own copy, run locally). Each marked supported/unsupported on a target AMD Zen host and on an Intel host |
| 10 | D3D12.X → Vulkan direct vs via vkd3d-proton | Written decision record comparing frame time and shader-compile time on one GDK sample through both paths, measured on one RDNA2 GPU |
| 11 | Check AMD machine-readable ISA license and coverage | Upstream URL, license text and GFX10.3 XML file identified. Opcode count compared with TuxBox decoder table (diff list attached) |
| 12 | Discovered repos not audited in depth (WinDurango, xodus, ScarlettX, SCARAB, omonkeyz, Percy6Stars) | Each gets a README + tree review with `confidence` raised to at least `medium` in repos.json |
