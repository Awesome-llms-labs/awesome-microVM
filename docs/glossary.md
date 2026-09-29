# Glossary

Terms that recur across microVM projects and docs.

- **VMM (Virtual Machine Monitor)** — the userspace process that creates and manages a virtual machine: vCPU threads, guest memory, and emulated/paravirtual devices. In microVMs the VMM is a single lean process per VM (Firecracker's model), shrinking attack surface.
- **microVM** — a hardware-virtualized VM stripped to essentials: minimal VMM, direct kernel boot, a handful of virtio devices. The term was coined by Firecracker; the rough profile is ~100 ms boot with single-digit-MiB VMM overhead.
- **KVM** — the Linux kernel's hardware-virtualization interface (`/dev/kvm`). Nearly every project in this list is a KVM *frontend*; alternatives include Microsoft Hypervisor (MSHV), Windows Hypervisor Platform (WHP), Gunyah, and Apple Silicon's Hypervisor.framework.
- **virtio** — the standard paravirtual device interface for guests (net, block, vsock, fs, console, balloon, RNG…). virtio-mmio is the memory-mapped transport used by microVM-style machine types; virtio-pci is the PCI transport with hotplug support.
- **vsock** — a socket address family for host↔guest communication that bypasses the network stack. Used by Firecracker guests, Nitro Enclaves (parent↔enclave), and libkrun.
- **jailer** — Firecracker's companion process that drops privileges, sets up namespaces/cgroups, and applies seccomp filters *before* the VMM starts, so a compromised VMM still can't reach the host.
- **seccomp (-BPF)** — Linux syscall filtering. MicroVMs layer seccomp on the VMM process (Firecracker's thread-specific filters, crosvm's Minijail, Solo5's `spt` tender) so even a VMM exploit has few syscalls to work with.
- **snapshot / restore** — serializing a VM's memory and device state to disk and resuming it later. Cold boot → ~125 ms; restore from snapshot → an order of magnitude faster. Snapshot *format* compatibility across VMM versions is generally **not** guaranteed (Cloud Hypervisor states this explicitly).
- **live migration** — moving a running VM between hosts: *precop* (copy memory while running, brief stop) vs *postcopy* (start on target and page memory on demand — added in Cloud Hypervisor v53.0). Usually secured with mTLS.
- **TEE (Trusted Execution Environment)** — hardware that encrypts guest memory and attests the guest's identity to remote parties, so the host/hypervisor can't read it. The basis of confidential computing.
- **SEV-SNP** — AMD's Secure Encrypted Virtualization with Secure Nested Paging: per-VM memory encryption with integrity protection and attestation, used by Confidential Containers and Alioth's hooks.
- **TDX (Trust Domain Extensions)** — Intel's confidential-VM technology: hardware-isolated "trust domains" with memory encryption and attestation; supported by CoCo, experimental in Cloud Hypervisor, hooked in Alioth.
- **PVH** — a paravirtualized boot mode for x86 guests (no BIOS/legacy emulation). QEMU's `microvm` machine type and Edera's Xen PVH "zones" boot this way; direct kernel boot is the microVM norm.
- **unikernel** — a single-address-space machine image where the application is linked directly against a library OS — no separate guest OS, no process boundary inside the VM. Solo5 (MirageOS), uhyve (HermitOS), and Hyperlight-Unikraft (Unikraft) target this model; startup is comparable to loading a process.
