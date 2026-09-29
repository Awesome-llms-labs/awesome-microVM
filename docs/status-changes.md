# Status Changes

Newest first. Archived projects, deprecations, migrations, and other status events affecting this list.

## 2026

- **Kata Containers 4.0.0** — Rust `runtime-rs` (with built-in Dragonball VMM) became the production default; the Go runtime is deprecated (bug/CVE fixes only, no new features).
- **Cloud Hypervisor v53.0** — postcopy/on-demand-paging live migration, mTLS-secured migration, VFIO migration v2.
- **libkrun moved to the `libkrun` GitHub org** (from `containers/libkrun`); `main` is now the unstable 2.0 development line — stable `stable-*` branches recommended for production.
- **Vercel Sandbox GA (2026-01-30)** — on Firecracker microVMs; SDK Apache-2.0, hosted platform proprietary.
- **Hyperlight entered CNCF Sandbox** (pre-1.0).
- **Confidential Containers v0.19.0 (2026-03-23)** — based on Kata 3.28.0; NVIDIA confidential (Blackwell) GPU attachment; CoCo Operator deprecated in favor of Helm charts.
- **uhyve v0.9.0 (2026-06-30)**; **krunkit v1.3.2** referenced July 2026; **battery** pre-alpha warm-pool manager for Flintlock under active 2026 development.
- **firectl** — maintenance mode (last tagged release v0.2.0, 2022); dependency/CVE upkeep only, not archived.

## Archived / deprecated

- **Weave Ignite — archived 2023-12-07.** Ran OCI images as Firecracker microVMs; upstream recommends Flintlock.
- **vmm-reference — archived / maintenance ended.** rust-vmm's reference VMM; maintainers lacked bandwidth and pointed users to Cloud Hypervisor.
- **Intel NEMU — archived / inactive.** QEMU-derived cloud VMM; Cloud Hypervisor is its successor.
