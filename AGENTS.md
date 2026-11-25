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
