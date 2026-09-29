# Choosing a microVM

Pick the VMM by the workload, not the hype. All five scenarios below are about *lightweight, fast-start isolation* — but they differ in the interface they expose (OCI? functions? a C API?) and in how deep the hardware trust boundary goes.

## 1. Serverless functions / ephemeral compute

**Profile:** thousands of short-lived, stateless invocations per minute; cold-start latency and per-instance memory dominate cost.

- **Firecracker** — the default answer. ≤125 ms boot and ≤5 MiB VMM overhead per microVM (official spec, enforced via CI), one VMM process per instance, `jailer` + seccomp for multitenant isolation, snapshot/restore for warm starts. Powers AWS Lambda and Fargate.
- **Cloud Hypervisor** — a heavier but more featureful alternative when you need live migration, virtio-fs, or Windows guests.
- **Orchestration:** [Flintlock](../README.md#orchestration--integration) (Firecracker/Cloud Hypervisor lifecycle service) or `firecracker-containerd` if you want containerd to schedule the VMs; **battery** adds warm-pool management to cut cold starts in 2026.
- **Managed:** AWS Lambda, Vercel Sandbox (GA Jan 2026), Fly.io Machines, E2B.

## 2. OCI containers with hardware-enforced isolation

**Profile:** run unmodified OCI images/Kubernetes pods, but each pod gets its own lightweight VM instead of a shared kernel.

- **Kata Containers** — the standard. 4.0.0 made the Rust `runtime-rs` the production default with the built-in **Dragonball** VMM; QEMU and Cloud Hypervisor backends remain for GPU passthrough and live migration. Deploy via kata-deploy with a Kubernetes RuntimeClass.
- **krunvm** — turn any OCI image directly into a libkrun microVM from the CLI, with no VM image maintenance. Best for ad-hoc or dev-loop use.
- **krunkit** — libkrun-backed containerd runtime / Kubernetes node agent (`krunkitd`); reportedly adopted by Podman Machine on macOS (community sources — no official Podman source re-verified).
- **firecracker-containerd** — run containers as Firecracker microVMs from containerd.
- **gVisor** (`runsc`) — if you want container-native semantics with a smaller footprint and can live with incomplete Linux ABI coverage; note it's a userspace kernel, *not* a hardware VM.

## 3. Confidential computing

**Profile:** workloads whose memory must stay encrypted from the host/hypervisor, with remote attestation.

- **Confidential Containers (CoCo)** — extends Kata with hardware-TEE-backed confidential VMs: AMD SEV-SNP, Intel TDX, IBM Secure Execution, attestation via Trustee, signed/sealed secrets, confidential NVIDIA GPU attachment (Blackwell). v0.19.0 (2026-03-23); deploy with Helm charts (CoCo Operator deprecated).
- **AWS Nitro Enclaves** — EC2 feature carving cryptographically isolated VMs out of an instance. Underlying Nitro VMM is proprietary, but the CLI/driver/tooling are open source (Apache-2.0); EIF images built from Docker images; NSM attestation; vsock parent↔enclave communication.
- **Alioth** — experimental Google VMM with SEV-SNP/TDX confidential-computing hooks; interesting for research, not production.

## 4. Embedded / in-process sandboxing

**Profile:** untrusted *code*, not containers — millisecond starts, callable from a host process, no guest OS.

- **Hyperlight** — the purpose-built answer. Millisecond startup, no guest kernel; typed host↔guest function calls with a default-deny host access model; KVM/MSHV/WHP backends; CNCF Sandbox; used by Microsoft's Agent Framework CodeAct. The **Hyperlight-Unikraft** variant runs ordinary Linux programs via a Unikraft guest kernel and adds snapshot support.
- **gVisor** (`runsc`) — a middle ground: process-granularity sandboxes with ~50 ms startup, lower memory than VMs, but not hardware-isolated.

## 5. Unikernels

**Profile:** single-address-space library-OS guests where the application *is* the kernel image; process-like startup.

- **Solo5** — `hvt` (KVM), `spt` (strict-seccomp process), and `virtio` tenders for MirageOS-style unikernels; startup comparable to process loading.
- **uhyve** — specialized hypervisor for HermitOS (Rust library OS) unikernels; network/filesystem hypercalls instead of device emulation; x86_64, aarch64, riscv64 guests.
- **Hyperlight-Unikraft** — ordinary Linux programs on a Unikraft guest kernel under Hyperlight, with snapshot support.

## 6. macOS hosts

**Profile:** Apple Silicon Mac — no KVM, so everything goes through Apple's Virtualization.framework (or Hypervisor.framework for lower-level VMMs).

- **shuru** — the microVM-native answer for AI agent sandboxes on the Mac: ephemeral Linux microVMs with checkpoints, VirtioFS mounts, vsock port forwarding, and a secrets proxy.
- **Apple Containerization** — one lightweight Linux VM per container; the OCI-native route.
- **vfkit** — minimal Virtualization.framework wrapper (CLI + Go API); what Podman and minikube build on.
- **Tart** — full macOS *and* Linux VMs on Apple Silicon, OCI-registry-distributed, CI-focused. Note the Fair Source license.
- **Lima** — WSL2-style Linux dev VMs with file sharing and port forwarding; vz driver by default.
- **libkrun family** (libkrun, krunvm, krunkit) — Hypervisor.framework backend on ARM64 Macs.

## Building blocks

- **rust-vmm** — the shared Rust crates (KVM bindings, virtio, vhost, device models) underneath Firecracker, Cloud Hypervisor, crosvm, StratoVirt, and Dragonball. Start here if you're *writing* a VMM.

## Caveats

- Benchmark numbers across projects are mostly community-measured and order-of-magnitude only — see [Benchmarks](../README.md#benchmarks--comparisons) for methodology caveats (VMM RSS vs guest RAM; cold boot vs snapshot restore; API-return vs application-ready).
- Licenses marked ⚠️ in the README were reported by search/index sources and not re-verified against primary files this pass — check `license_verified` in [data/microvms.json](../data/microvms.json).
- Cloud Hypervisor does **not** guarantee snapshot/migration compatibility across versions; assume the same caution for Firecracker snapshots across releases unless documented.
