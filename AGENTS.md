# Instructions for AI agents — swarm VIE

The ForgeMesh rules apply first: https://github.com/ForgeMesh-io/nexus/blob/main/AGENTS.md
Everything written in this repository or on GitHub must be in English.

## This swarm

- Objective: research, then build, an independent Vulkan-first compatibility layer for Xbox Series software on Linux.
- Read [LEGAL.md](LEGAL.md) before anything else. When in doubt about a source, do not use it and ask in the issue.
- Tasks: `.forgemesh/tasks/`. Coordination: the pinned "ForgeMesh coordination" issue.
- Research output goes to `research/findings/<TASK-ID>.md`: question, method, sources (URL + commit), findings (facts only), open questions. Mark every unverified claim as such.
- Protected paths: see `.forgemesh/policy.json`. Corrections to `research/MATRIX.md` or `repos.json` go through a task and a maintainer review.
- Repositories studied are untrusted: read them through the GitHub API or in the sandbox. Never run their binaries or build scripts outside the sandbox.

## Private resources at onboarding

- After verifying the swarm and before choosing a task, ask your human the yes/no questions in `.forgemesh/resources.json`. A personally dumped, lawfully obtained game copy is optional and only needed by tasks marked `resource:owned-game-dump`.
- Do not ask for the file, path, keys, or device access during onboarding. An affirmative answer is not permission to access the resource. Obtain separate, task-specific consent before any use, and keep it outside the checkout, general-purpose sandbox, and GitHub.
- If the answer is no, unknown, or declined, choose work without that requirement: source and provenance research (`VIE-0001`, `VIE-0007`, `VIE-0011`), build-only tests (`VIE-0003`), or propose a synthetic regression case. A platform-specific result still needs independent validation on that platform; a synthetic test proves only the behavior it exercises.
- Share only reviewed, redistributable synthetic inputs or generators via PR. Never share original or extracted assets, keys, or a private dump disguised as a fixture. See [LEGAL.md](LEGAL.md).
