# Awesome MicroVM [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated directory of the **microVM ecosystem** — lightweight virtual machine monitors (VMMs) that boot minimal guests in ~100 milliseconds with tiny memory footprints, plus the orchestration tools, production platforms, benchmarks, and learning resources around them.

A microVM is a hardware-virtualized VM stripped to the essentials: a minimal VMM process, a direct-kernel-boot guest, and a small set of virtio devices — built for multi-tenant serverless functions, strongly-isolated containers, edge workloads, and sandboxed agent code. This list tracks every notable project in the space as of **September 2026**.

**Scope notes:** `gVisor` is included but is explicitly *not* a hardware VM (a userspace application kernel) — it belongs because it competes for the same workloads. Building-block crates ([rust-vmm](#rust-vmm)) and adjacent technologies (Nitro Enclaves tooling, Edera) are listed with clear labels. [Unverified](docs/status-changes.md) items are flagged ⚠️ where license claims weren't re-checked against primary files this pass.

## 2026 Highlights

- **Kata Containers 4.0** made the Rust `runtime-rs` the production default with the built-in **Dragonball** VMM; the Go runtime is deprecated (bug/CVE fixes only).
- **Cloud Hypervisor v53.0** added postcopy/on-demand-paging live migration, mTLS-secured migration, and VFIO migration v2.
- **Hyperlight** entered **CNCF Sandbox** — embeddable microVM library for in-process function sandboxing.
- **Vercel Sandbox** reached GA (Jan 30, 2026) running untrusted/AI-generated code in ephemeral Firecracker microVMs.

## Contents

- [MicroVMs & Lightweight VMMs](#microvms--lightweight-vmms) — the monitors themselves
- [Orchestration & Integration](#orchestration--integration) — lifecycle tooling on top of VMMs
- [MicroVMs on macOS](#microvms-on-macos) — the Mac story: Virtualization.framework sandboxes
- [Managed Platforms & Production Users](#managed-platforms--production-users) — who's running what
- [Benchmarks & Comparisons](#benchmarks--comparisons) — official numbers and community measurements
- [Learning Resources](#learning-resources) — papers, docs, talks
- [Archived / Deprecated](#archived--deprecated) — retired projects
- [Guides](#guides)
- [Contributing](#contributing)
- [License](#license)

---

## MicroVMs & Lightweight VMMs

The monitors, runtimes, and libraries that boot lightweight virtualized guests.

- [Firecracker](https://github.com/firecracker-microvm/firecracker) — The originator of the "microVM" term: purpose-built minimalist KVM VMM from AWS for secure, multi-tenant containers and serverless. One VMM process per microVM with OpenAPI/REST configuration and direct kernel boot (no BIOS); minimalist device model (virtio-net/block, vsock, entropy, pmem, rate limiters) keeps memory footprint and attack surface small; `jailer` process + thread-specific seccomp filters for production isolation; snapshot/restore and memory hotplug; x86_64 and aarch64 hosts, Linux guests. Official spec: ≤125 ms boot, ≤5 MiB VMM overhead (1 vCPU/128 MiB), enforced via CI.
- [Cloud Hypervisor](https://github.com/cloud-hypervisor/cloud-hypervisor) — Rust VMM for modern cloud workloads built on rust-vmm crates, running on KVM and Microsoft Hypervisor (MSHV). Minimal emulation, low-latency/low-footprint, 64-bit only; CPU, memory, PCI, and virtio-{net,block,pmem,fs,vsock} hotplug; machine-to-machine live migration and snapshot/restore (not guaranteed across versions); x86_64 and AArch64 mainline (riscv64 experimental); 64-bit Linux and Windows 10/Server 2019 guests; direct kernel boot (PVH/bzImage) or firmware boot; VFIO passthrough, vDPA and TDX still experimental.
- [Kata Containers](https://github.com/kata-containers/kata-containers) — OCI/CRI-compatible secure container runtime that runs each pod inside a lightweight VM for hardware-enforced isolation. 4.0.0 rewrote the runtime in Rust — `runtime-rs` is the production default (Go runtime deprecated); supported hypervisors QEMU, Cloud Hypervisor, Dragonball (built-in), and a Firecracker configuration; x86_64, aarch64, s390x; deploys via kata-deploy with a Kubernetes RuntimeClass; virtio-fs shared filesystems, EROFS snapshotter, dm-verity verified rootfs, virtio-mem memory hotplug; GPU passthrough tested with QEMU.
- [Dragonball](https://github.com/kata-containers/kata-containers) — Lightweight built-in VMM for Kata's Rust runtime (Alibaba/Ant Group contribution), for users who prefer not to run QEMU or Cloud Hypervisor. Integrated into `runtime-rs` as the default VMM (Kata 4.0.0); virtio-net and virtio-blk support; VFIO device passthrough; multi-queue networking propagated through the runtime.
- [Confidential Containers (CoCo)](https://github.com/confidential-containers/confidential-containers) — CNCF Sandbox stack extending Kata Containers to run workloads inside hardware-TEE-backed confidential VMs with remote attestation. Confidential VMs on AMD SEV-SNP, Intel TDX, IBM Secure Execution; attestation via Trustee/attestation-service with signed and sealed secrets; NVIDIA confidential (Blackwell) GPU attachment in recent releases; v0.19.0 (2026-03-23, based on Kata 3.28.0); CoCo Operator deprecated in favor of Helm charts. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [gVisor (runsc)](https://github.com/google/gvisor) — Userspace application kernel (Go) implementing a Linux-like interface to sandbox containers — **explicitly not a hardware VM** (the project says it is "not a VM in the everyday sense"), but a direct alternative for the same sandbox workloads. `runsc` OCI runtime integrates with Docker/Kubernetes and ships a containerd shim; interposes on syscalls to shrink the host-kernel attack surface, with KVM or systrap (ptrace) platform; x86_64 and ARM64, requires Linux 5.6+; lower footprint and faster startup than full VMs, but incomplete Linux ABI compatibility. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [crosvm](https://github.com/google/crosvm) — Secure, lightweight Rust VMM built for ChromeOS (Crostini Linux guests, ARCVM Android guests), now used across Android, Windows, and Cuttlefish. Process-per-device model — each virtio device can run in its own Minijail-sandboxed process (namespaces, seccomp-BPF, dropped capabilities); hypervisor backends KVM, Gunyah, GenieZone, Halla on Linux/Android, WHPX and HAXM on Windows; x86_64, aarch64, riscv64; broad virtio device model (net/block with qcow2/zstd, 3D GPU via virgl/gfxstream, snd, fs/9p, console, RNG, balloon, vsock, TPM, pmem, video decode/encode); io_uring, vhost, and its own async runtime `cros_async`. License BSD-3-Clause (verified). Contributions via Chromium Gerrit — canonical upstream at `chromium.googlesource.com/crosvm/crosvm`; GitHub is a read-only mirror.
- [QEMU microvm machine type](https://www.qemu.org/docs/master/system/i386/microvm.html) — Firecracker-inspired minimalist machine type inside QEMU for microVM-style direct-kernel-boot workloads. x86-specific; boots kernels directly without BIOS/legacy device enumeration; virtio-mmio transport (up to eight devices); optional legacy timers/serial only when requested; no device hotplug, no live migration across QEMU versions. Language C. ⚠️ License GPL-2.0 reported, not re-verified from primary files.
- [kvmtool (lkvm)](https://github.com/kvmtool/kvmtool) — Minimal, fast KVM-based VMM intended as a simple native virtualization tool for Linux. Lightweight native KVM tool that boots Linux guests quickly; x86, arm64, riscv and others; virtio devices and VFIO passthrough; no live migration or UEFI support; commits as recent as April 2026. Language C. ⚠️ License GPL-2.0 reported, not re-verified from primary files.
- [uhyve](https://github.com/hermit-os/uhyve) — Specialized Rust hypervisor for running HermitOS unikernels as lightweight VMs. Purpose-built for the Hermit unikernel (Rust library OS); networking and filesystem hypercalls instead of full device emulation; HermitOS guests support x86_64, aarch64, riscv64; KVM-backed on Linux; v0.9.0 released 2026-06-30. ⚠️ License Apache-2.0 OR MIT reported, not re-verified from primary files.
- [Solo5](https://github.com/solo5/solo5) — Sandboxed unikernel execution environment running MirageOS-style unikernels via small "tender" backends. `hvt`: hardware-virtualized tender (KVM); `spt`: strict-seccomp unprivileged process tender; `virtio`: QEMU/KVM/bhyve target; startup comparable to process loading (no guest OS boot); minimal auditable codebase used by MirageOS and NanoVMs-style deployments. Language C. ⚠️ License ISC reported, not re-verified from primary files.
- [libkrun](https://github.com/libkrun/libkrun) — Lightweight dynamic KVM library exposing a C API so programs can easily run processes in partially isolated, virtualized environments. Embeddable VMM library — no separate VMM process to manage; backends: KVM on Linux, Hypervisor.framework on Apple Silicon; minimal device model (virtio networking, filesystems, vsock); used for container and confidential-computing integrations; `main` is the unstable 2.0 development line (stable `stable-*` branches recommended for production). ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [krunvm](https://github.com/libkrun/krunvm) — CLI that turns OCI container images into libkrun-based microVMs with no disk-image maintenance. Runs OCI images directly as microVMs — minimal footprint, fast boot; Linux/KVM on x86_64 and aarch64; macOS/HVF on ARM64; host volume mapping and guest port exposure. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [krunkit](https://github.com/containers/krunkit) — libkrun-backed containerd runtime / Kubernetes node agent (`krunkitd`) for running pods inside microVMs. Turns Kubernetes/containerd workloads into libkrun microVMs; reportedly adopted by Podman Machine on macOS as of Podman 6.0 (community sources — no official Podman source re-verified); macOS + Linux support via libkrun backends; v1.3.2 referenced July 2026. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [StratoVirt](https://github.com/openeuler-mirror/stratovirt) — Rust KVM VMM from the openEuler ecosystem (Huawei-originated) with both a `microvm` mode and a standard full-featured mode. Two machine models: lightweight `microvm` and standard; x86_64 and aarch64; virtio-mmio device model in microvm mode; QMP, libvirt, and OCI-compatible management interfaces; snapshot and warm-restore support; VFIO passthrough in standard mode. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [Alioth](https://github.com/google/alioth) — Experimental Google VMM written in Rust, exploring a modern KVM/HVF-based design. KVM backend on Linux; HVF backend on macOS; x86_64 Linux and aarch64 Linux/macOS; virtio net/vsock/blk/rng/fs/balloon devices; VFIO with IOMMUFD; AMD SEV/SNP and Intel TDX confidential-computing hooks. ⚠️ License not disclosed in README — not verified.
- [Hyperlight](https://github.com/hyperlight-dev/hyperlight) — Embeddable microVM VMM library for running untrusted code in milliseconds with no guest OS, designed for in-process function sandboxing. Millisecond startup with no guest kernel; typed host↔guest function calls and a default-deny host access model; backends KVM, MSHV, WHP (Windows Hypervisor Platform); used by Microsoft's Agent Framework CodeAct for agent code execution; CNCF Sandbox project (pre-1.0). Variant: [Hyperlight-Unikraft](https://github.com/hyperlight-dev/hyperlight-unikraft) runs ordinary Linux programs via a Unikraft guest kernel and adds snapshot support. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [AWS Nitro Enclaves tooling](https://github.com/aws/aws-nitro-enclaves-cli) — EC2 feature for carving cryptographically isolated, hardened VMs ("enclaves") out of an instance. The underlying Nitro VMM/service is proprietary; CLI/driver/tooling are open source (Apache-2.0, verified). Enclaves are isolated VMs with no persistent storage (RAM filesystem), no interactive access, and no networking by default; Nitro Secure Module (NSM) with cryptographic attestation; EIF (enclave image format) built from Docker images + signed blobs (kernel, LinuxKit user space, nsm.ko); parent↔enclave communication over vsock with vsock-proxy for controlled egress; kernel driver upstream since Linux 5.10 (x86_64) / 5.16 (arm64).
- [Edera Protect](https://github.com/edera-dev/protos) — ⚠️ **Commercial / partially open — not an open-source VMM.** Hardened container runtime using Xen/PVH-based type-1 isolation ("zones"); the core daemon is proprietary under an EderaON license-key model, with selected open-source components (protobuf APIs, "Am I Isolated" tooling, Falco plugin, [learning materials](https://github.com/edera-dev/learn)). Each container runs in its own Xen PVH "zone" with a minimal guest kernel; Rust Xen service reimplementations, Styrolite, and GPU PCIe passthrough were all in the Trail of Bits 2025 audit scope.

### Building blocks (not runnable VMMs)

- [rust-vmm](https://github.com/rust-vmm/rust-vmm) — Shared Rust virtualization component ecosystem: reusable KVM/virtio/vhost crates, common virtio device implementations, and test infrastructure. Not itself a runnable VMM — the crates it ships are used by Firecracker, Cloud Hypervisor, crosvm, StratoVirt, and Dragonball; crates are consolidating into a monorepo. ⚠️ Licenses mostly Apache-2.0 / BSD-3-Clause reported, not re-verified from primary files.

---
## Orchestration & Integration

Lifecycle management, containerd/Kubernetes integrations, and declarative fleet tooling on top of VMMs.

- [firecracker-containerd](https://github.com/firecracker-microvm/firecracker-containerd) — Containerd integration that runs containers as Firecracker microVMs. Specialized containerd control plugin; `containerd-shim-aws-firecracker` per-VM shim; in-guest agent + runc; devmapper snapshotter; VM lifecycle API; commits in 2026. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [Flintlock](https://github.com/liquidmetal-dev/flintlock) — MicroVM lifecycle management service (formerly Weaveworks), now community-run. Firecracker and Cloud Hypervisor backends; gRPC/HTTP lifecycle API; OCI-sourced kernels/rootfs/initrd; cloud-init and Ignition support; Prometheus metrics; originally paired with Cluster API Provider Microvm; upstream-recommended successor to Weave Ignite. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [battery](https://github.com/liquidmetal-dev/battery) — 2026 warm-pool manager for Flintlock microVMs to cut cold-start latency. Pre-alpha, active implementation in 2026 (README status text may lag). ⚠️ License not verified.
- [firectl](https://github.com/firecracker-microvm/firectl) — Simple CLI for launching raw Firecracker microVMs without containerd. Console access; disk and network configuration; thin wrapper over the Firecracker API; maintenance mode (last tagged release v0.2.0, 2022 — dependency/CVE upkeep only, not archived). ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- [microvm.nix](https://github.com/microvm-nix/microvm.nix) — NixOS-based declarative orchestration for microVM fleets. Declarative NixOS VM definitions; backends: QEMU, Cloud Hypervisor, Firecracker, crosvm, kvmtool, StratoVirt, Alioth, vfkit; shares the host nix store via virtio-fs/9p. ⚠️ License not verified.
- [Apple Containerization](https://github.com/apple/containerization) — Apple's 2025 open-source Swift framework for running Linux containers in lightweight VMs on macOS. Purpose-built lightweight Linux VM per container on Apple Silicon; OCI image support; native macOS integration. ⚠️ Details and license (Apache-2.0) not verified from primary sources this pass.

---

## MicroVMs on macOS

Apple Silicon Macs can't run KVM, so the microVM story on macOS goes through Apple's Virtualization.framework (and Hypervisor.framework for lower-level VMMs). These projects bring microVM-style sandboxes to the Mac:

- [shuru](https://github.com/superhq-ai/shuru) — Local-first microVM sandbox for running AI agents safely on macOS, with experimental Linux support. Boots lightweight Linux VMs via Apple's Virtualization.framework on macOS 14+ (Apple Silicon); a KVM backend for Linux ARM64 hosts is experimental. Every sandbox is ephemeral — the rootfs resets on every run; checkpoints save reusable environments; VirtioFS directory mounts (read-only by default, writes go to a discarded overlay); vsock port forwarding with no network device needed; secrets stay on the host via an HTTPS substitution proxy; per-host network allowlists; TypeScript SDK and an agent skill so Claude Code, Cursor, and Copilot use it automatically. Install via Homebrew (`superhq-ai/tap`). Language Rust. License Apache-2.0 (verified).
- [vfkit](https://github.com/crc-org/vfkit) — Minimal command-line hypervisor and Go API wrapping Apple's Virtualization.framework to run Linux VMs on macOS. Small, auditable Go codebase; adopted by Podman 5.0+, minikube, and CRC. Language Go. License Apache-2.0 (verified).
- [Tart](https://github.com/openai/tart) — Virtualization toolset to build, run, and manage macOS and Linux VMs on Apple Silicon for CI and automation. Near-native performance via Virtualization.framework; push/pull VMs from any OCI-compatible container registry; Packer plugin for VM creation; aimed at CI/CD and reproducible dev environments (macOS 13+). Note: the repo moved from `cirruslabs/tart` to `openai/tart`. Language Swift. ⚠️ License is Fair Source (not OSI-approved; GitHub reports NOASSERTION).
- [Lima](https://github.com/lima-vm/lima) — Lightweight Linux VMs with automatic file sharing and port forwarding (WSL2-style) for running containerd, Docker, Podman, or Kubernetes on a Mac. Uses the vz (Virtualization.framework) driver by default on macOS, with an opt-in krunkit driver. CNCF project. ⚠️ License Apache-2.0 reported, not re-verified from primary files.
- Also on macOS: [Apple Containerization](#orchestration--integration) runs each Linux container in its own lightweight VM on Apple Silicon, and the [libkrun](#microvms--lightweight-vmms) family (libkrun, krunvm, krunkit) supports macOS via Hypervisor.framework on ARM64.

---

## Managed Platforms & Production Users

| Platform | Underlying tech | Confidence |
|---|---|---|
| AWS Lambda, AWS Fargate | Firecracker | ✅ verified — Firecracker README: "developed at AWS to accelerate the speed and efficiency of services like AWS Lambda and AWS Fargate" |
| Fly.io Machines | Firecracker | ✅ verified — fly.io: "Fly.io runs every workload as a Firecracker microVM"; suspend/resume via Firecracker snapshots |
| Vercel Sandbox (`@vercel/sandbox`) | Firecracker | index — "runs untrusted or AI-generated code inside an ephemeral Firecracker microVM"; GA Jan 30, 2026; SDK Apache-2.0, hosted platform proprietary |
| E2B (AI agent cloud) | Firecracker-based infra/fork | index — community sources list E2B under Firecracker applications; no E2B primary doc re-verified |
| Modal (sandboxes/functions) | gVisor (`runsc`) — not a microVM platform | index — Modal security docs (via community citations); gVisor checkpoint/restore for memory snapshots |
| Northflank | Kata Containers + Cloud Hypervisor primary; Firecracker and gVisor options | index — community research; team contributes upstream to Kata/QEMU/Cloud Hypervisor |
| ChromeOS: Crostini (Linux), ARCVM (Android) | crosvm | ✅ verified — crosvm README: "Originally developed for ChromeOS to run Linux (Crostini) and Android guests (ARCVM)" |
| Android: Terminal app, Cuttlefish | crosvm | ✅ verified — crosvm README |
| Google Cloud Run / GKE | gVisor | index — cited in Modal's security materials and gVisor ADOPTERS.md; gVisor README points to ADOPTERS.md for the production-user list |
| Nitro Enclaves users | Nitro (proprietary VMM) + open CLI/driver | ✅ verified (tooling) — EC2 enclave feature; kernel driver upstream |
| Edera customers | Edera Protect zones (Xen/PVH) | index — vendor claims; core not open source |

---

## Benchmarks & Comparisons

### Official / peer-reviewed numbers

- **Firecracker official spec:** ≤125 ms boot and ≤5 MiB VMM overhead for 1 vCPU/128 MiB, enforced via CI (README references the spec docs; numbers from research of the official performance spec). Up to ~150 microVMs launched per second per host.
- **NSDI'20 paper** — Agache et al., *"Firecracker: Lightweight Virtualization for Serverless Applications"* ([usenix.org](https://www.usenix.org/conference/nsdi20/presentation/agache)): ~3 MB memory overhead per microVM, ~125 ms boot, plus fio/iperf3 I/O evaluations. The canonical peer-reviewed microVM evaluation.

### Community-measured comparisons (order-of-magnitude only — NOT authoritative)

| Setup | Cold boot | Memory overhead | Source confidence |
|---|---|---|---|
| Firecracker | ~125 ms | <5 MiB VMM (excluding guest RAM) | consistent with official spec |
| gVisor (`runsc`) sandbox | ~50 ms (process granularity) | ~15–25 MiB sandbox | index, community tables |
| Kata + QEMU | ~150–300 ms (image-dependent) | ~60–120+ MiB | index, community tables |
| Kata + Cloud Hypervisor | — | ~80 MiB (one community table) | index, community tables |

Source for the community rows: [Harness Internals — MicroVM Sandbox Infrastructure](https://github.com/krrish777/ideenkasten/blob/HEAD/02%20-%20Deep%20Dives/Harness-Engineering-Internals/Harness-Internals-MicroVM-Sandbox-Infrastructure.md) (useful but non-authoritative).

### Methodology caveats — read before comparing

- **VMM RSS vs total guest memory:** "memory overhead" figures usually exclude guest RAM — compare like with like.
- **Cold boot vs snapshot restore:** snapshot/resume (Firecracker, Modal/gVisor, Vercel persistent sandboxes) is an order of magnitude faster than cold boot — the numbers are not interchangeable.
- **VM creation vs application-ready:** API-return latency ≠ workload-serving latency (image pull, init, readiness probes).
- Benchmarks from 2020–2023 papers predate Kata runtime-rs/Dragonball, Cloud Hypervisor postcopy migration, and Hyperlight entirely.
- No vendor-published boot/memory figures were found for crosvm, StratoVirt, Alioth, libkrun, uhyve, or Solo5 — community guesses are not quoted here.

---

## Learning Resources

- **NSDI'20 paper** — Agache et al., *"Firecracker: Lightweight Virtualization for Serverless Applications"*: [usenix.org](https://www.usenix.org/conference/nsdi20/presentation/agache) — the canonical microVM paper.
- **Firecracker docs & design** — [firecracker-microvm.github.io](https://firecracker-microvm.github.io/); design doc in-repo at `docs/design.md`.
- **Marc Brooker's Firecracker retrospective** (2025-09-18, AWS principal engineer) — [mbrooker-blog](https://github.com/mbrooker/mbrooker-blog/blob/HEAD/_posts/2025-09-18-firecracker.md): economics of multitenancy, why virtualization over containers for Lambda.
- **crosvm book** — [crosvm.dev](https://crosvm.dev/) — user guide, architecture deep dive, API docs.
- **gVisor docs** — [gvisor.dev](https://gvisor.dev/) — quickstarts, architecture, ADOPTERS.md production-user list.
- **Kata Containers docs** — [kata-containers.github.io/kata-containers](https://kata-containers.github.io/kata-containers/); 4.0.0 release overview at [kata-containers/www.katacontainers.io](https://github.com/kata-containers/www.katacontainers.io).
- **Cloud Hypervisor release notes** — [cloud-hypervisor/release-notes.md](https://github.com/cloud-hypervisor/cloud-hypervisor/blob/HEAD/release-notes.md) — per-release feature tracking, including v53.0 migration work.
- **Nitro Enclaves user guide** — [docs.aws.amazon.com](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave-cli.html).
- **Edera learning materials** — [edera-dev/learn](https://github.com/edera-dev/learn), including "Am I Isolated" open tooling.
- **Community comparison deep-dive** — [krrish777/ideenkasten — MicroVM Sandbox Infrastructure](https://github.com/krrish777/ideenkasten/blob/HEAD/02%20-%20Deep%20Dives/Harness-Engineering-Internals/Harness-Internals-MicroVM-Sandbox-Infrastructure.md): useful but non-authoritative numbers.
- **Field notes on Vercel Sandbox** (Firecracker microVM isolation in practice) — [dev.to](https://dev.to/ahmed_mahmoud360/running-ai-generated-code-safely-field-notes-on-vercel-sandbox-3g4e).

---

## Archived / Deprecated

- [Weave Ignite](https://github.com/weaveworks/ignite) — **ARCHIVED 2023-12-07.** Ran OCI images as Firecracker microVMs (Go, Apache-2.0); upstream recommends [Flintlock](#orchestration--integration).
- [vmm-reference](https://github.com/rust-vmm/vmm-reference) — **ARCHIVED / maintenance ended.** rust-vmm's reference VMM; maintainers lacked bandwidth and pointed users to Cloud Hypervisor.
- [Intel NEMU](https://github.com/intel/nemu) — **ARCHIVED / inactive.** QEMU-derived cloud VMM; Cloud Hypervisor is its successor.

---

## Guides

- [Choosing a microVM](docs/choosing-a-microvm.md) — which VMM fits serverless, OCI containers, confidential computing, in-process sandboxing, or unikernels.
- [Glossary](docs/glossary.md) — VMM, virtio, vsock, jailer, seccomp, snapshot/restore, live migration, TEE/SEV-SNP/TDX, PVH, unikernel.
- [Status changes](docs/status-changes.md) — archived projects and notable 2026 status changes.
- [Machine-readable catalog](data/microvms.json) — every entry with license, verification status, categories, and features.

## Contributing

Entries and corrections are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Every PR is checked by CI: lychee link check over all markdown files, and JSON-schema validation of `data/microvms.json`.

## License

[MIT](LICENSE) © 2026 dakotac1994
