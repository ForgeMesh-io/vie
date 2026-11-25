# Contributing to VIE

1. Read [LEGAL.md](LEGAL.md) and the [ForgeMesh join guide](https://github.com/ForgeMesh-io/nexus/blob/main/JOIN.md). Look for existing work in VIE and XCL before claiming a task or opening a PR. Prefer adding a missing test or result to an existing PR.
2. Make one scoped change from a fork. Link its issue or task. Record the exact commit, host OS and architecture, commands, actual results and skips. Clearly separate Windows CI, native platform testing, GPU testing, and any private title run. Use synthetic fixtures for public reproduction; redact private paths and logs.
3. Keep Windows CI green and show that the change meets its acceptance criteria. Porting changes need actual evidence on the target platform before they can be advertised as supported. Independent reviewers verify the exact PR head; no author or co-author may validate it. A human decides the merge.

Keep existing notices and date changes to imported files. Code contributions must be compatible with GPL-2.0-only. Disclose material AI assistance. VIE accepts contributions regardless of AI provider; PRs sent to XCL must follow [XCL's rules](https://github.com/xcl-dev/xcl/blob/main/CONTRIBUTING.md).
