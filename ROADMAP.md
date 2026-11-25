# VIE roadmap

Milestones are reached only with evidence (see LEGAL.md rule 6).

## M0 — Research campaign (current)

- [x] Audit of existing projects and licences (`research/MATRIX.md`)
- [ ] Licence and provenance questions resolved (VIE-0001, VIE-0002, VIE-0011)
- [ ] Headline claims of existing projects mapped to evidence (VIE-0008)
- [ ] Build-only reproduction of the most reusable permissive project (VIE-0003)
- [ ] Hardware feasibility on Polaris and RDNA2 (VIE-0005, VIE-0006)

## M1 — Architecture decision

- [ ] Decision record: runtime host, graphics path (via vkd3d-proton first or direct Vulkan), shader pipeline (VIE-0010)
- [ ] `THIRD_PARTY.md` and repository layout
- [ ] Test harness with a homebrew/self-built sample only

## M2 — First frame

- [ ] Homebrew sample presents a frame through VIE on an RDNA2 GPU, in CI where possible
- [ ] RDNA2 → SPIR-V translation validated by `spirv-val` and a CPU reference interpreter

## M3 — Broader compatibility

- [ ] Milestones per title tracked as boot → menus → in-game → playable, with fps on named hardware
- [ ] Polaris (gfx803) fallbacks: mesh shaders through compute, ray tracing disabled or emulated
