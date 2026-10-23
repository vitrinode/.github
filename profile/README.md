# Vitrinode

**Secure, centralised management for Linux infrastructure.**

Vitrinode is an identity-first infrastructure management platform that brings Linux host inventory, device trust, controlled remote operations and infrastructure visibility into one management plane.

Built for infrastructure teams managing Debian, Ubuntu, SUSE Linux Enterprise and Red Hat Enterprise Linux, including Proxmox VE environments.

Every managed host has its own cryptographic identity. Administrators authenticate using WebAuthn/FIDO2 security keys, and privileged operations follow a constrained, signed and audited execution path.

**Vitrinode is in active development towards v1.0.**

[Website](https://vitrinode.com) · [Product](https://vitrinode.com/product) · [Architecture](https://vitrinode.com/architecture) · [Security](https://vitrinode.com/security) · [Development builds](https://vitrinode.com/screenshots)

---

## Why Vitrinode?

Managing a growing Linux estate often means switching between SSH sessions, scripts, hypervisor interfaces and monitoring tools to understand what is running and what has changed.

Vitrinode brings those activities together without introducing unrestricted remote shell execution or replacing established configuration-management workflows.

The platform is designed around three principles:

- **Visibility:** Know which hosts are enrolled, reachable, trusted and associated with virtual infrastructure.
- **Control:** Execute explicitly defined operations with cryptographic authorisation, expiry and replay protection.
- **Accountability:** Maintain auditable records of administrative access, device trust decisions and remote operations.

Vitrinode complements SSH, Ansible and existing Linux administration practices rather than attempting to replace them.

## Capabilities

The following capabilities are implemented and tested in the development build. Implementation does not imply production readiness.

### Device identity and access

- Single-use device enrolment with client-generated private keys.
- Mutual TLS authentication between agents and the control plane.
- Per-request device and certificate revocation enforcement.
- Passwordless administrator authentication using WebAuthn/FIDO2.
- Server-enforced security-key step-up for sensitive administrative actions.

### Host inventory and visibility

- Device registration, heartbeats, connectivity and last-seen status.
- CPU, memory, load, uptime, storage and network inventory.
- systemd unit state and service health.
- APT package and update information for Debian and Ubuntu.
- Docker daemon, container and image inventory.
- Device trust and attestation status.
- Read-only Proxmox VE VM and container correlation.

Inventory facts are collected and stored. The comprehensive per-host inventory interface and historical change timeline remain planned.

### Controlled remote operations

Vitrinode uses signed, expiring and replay-protected jobs with a fixed operation vocabulary.

Currently implemented operations:

| Operation | Purpose |
|---|---|
| `system.status` | Retrieve host status |
| `systemd.list` | Enumerate systemd units |
| `systemd.status` | Inspect service state |
| `apt.list_updates` | Retrieve available APT updates |
| `systemd.restart` | Restart an authorised service |

Privileged operations require additional authorisation and a trusted target device.

The network-facing agent runs without elevated privileges. A separate local executor independently validates signed jobs before performing permitted privileged operations.

Executor hardening, including device binding, protected identity material and restart-persistent at-most-once execution, has passed isolated-host testing but has not yet been deployed across enrolled hosts.

**Arbitrary remote shell execution is not part of the job protocol.**

### Device trust and auditing

- TPM-backed attestation using measured-boot evidence where available.
- Explicitly weaker software evidence on hosts without suitable TPM support.
- Periodically re-evaluated device trust states.
- Trust-gated privileged job creation and dispatch.
- Append-only records for authentication, step-up, device administration, remote jobs and SSH certificate issuance.
- Detection of outstanding privileged jobs following device trust loss.

Device authentication and device attestation are separate security claims. A valid device certificate does not independently establish a trustworthy runtime state.

[Full capability matrix and implementation status →](https://vitrinode.com/product)

## Architecture

Vitrinode separates infrastructure management into three principal components:

| Component | Responsibility |
|---|---|
| **Control plane** | Administrator authentication, device identity, inventory, policy enforcement, job signing and auditing |
| **Agent** | Outbound communication, heartbeat, inventory collection and job delivery |
| **Executor** | Local privileged operations through a constrained, permission-checked Unix socket |

Agents initiate connections to the control plane. Managed hosts do not require inbound management connections from the server.

The architecture establishes independent trust boundaries:

- **Administrator ↔ Control plane:** WebAuthn/FIDO2 and server-side sessions.
- **Control plane ↔ Agent:** Mutual TLS, device certificates and continuous revocation enforcement.
- **Agent ↔ Executor:** Local Unix socket, independent signature validation and typed operations.
- **Control plane ↔ Proxmox VE:** Dedicated read-only API credentials with explicit TLS certificate trust.

Cryptographic keys are separated by purpose, and security-sensitive actions are recorded for later review.

[Architecture and trust boundaries →](https://vitrinode.com/architecture)

## Short-lived SSH access

**In development**

Vitrinode is developing certificate-based SSH access using short-lived OpenSSH user certificates.

The development implementation includes:

- Certificate issuance following request-bound WebAuthn/FIDO2 authentication.
- Certificate lifetimes of up to 15 minutes.
- Per-user principal restrictions.
- Refusal of root principal issuance.
- End-to-end authentication demonstrated against stock OpenSSH using a development-only software certificate authority.

Production readiness remains dependent on additional host-side enforcement, CA trust distribution, revocation controls and hardware-backed signing integration.

The development software CA is not intended for production use.

## Vitrinode-HSM

[Vitrinode-HSM](https://vitrinode.com/hsm) is a separate, purpose-built hardware-signing project based on the Raspberry Pi RP2350.

Its purpose is to protect Vitrinode's SSH user certificate authority key and restrict certificate issuance to an explicitly defined policy.

The design includes:

- Arm TrustZone-M separation between Secure and Non-secure execution.
- On-device Ed25519 CA key generation.
- Typed, policy-constrained OpenSSH certificate issuance.
- No general-purpose signing or private-key export interface.
- A dedicated USB communication protocol.
- Planned independent trusted time, tamper-evident auditing and production key protection.

**Verified on physical hardware:**

- End-to-end OpenSSH certificate signing and authentication.
- Secure/Non-secure execution boundaries.
- USB request and response handling.
- On-device CA provisioning with physical-presence confirmation.
- Persistent CA storage and recovery across power cycles.
- Defined behaviour under injected provisioning power failures.

Production key protection, trusted time, device audit logging, rate limiting, sealing and control-plane integration remain outstanding.

Vitrinode-HSM is not currently integrated into the management platform and has not undergone independent security certification.

[Hardware architecture, threat model and development status →](https://vitrinode.com/hsm)

## Platform support

Platform coverage differs between implemented code and verified environments.

| Platform | Current status |
|---|---|
| Debian | Validated on real hosts |
| Ubuntu | Implemented; containerised validation |
| SUSE Linux Enterprise | Targeted; distribution-neutral collection implemented |
| Red Hat Enterprise Linux | Targeted; distribution-neutral collection implemented |
| Docker | Inventory validated on a real host |
| Proxmox VE | Read-only correlation validated against a live cluster |

RPM-based package inventory and additional distribution-specific compatibility testing remain outstanding.

Windows and macOS host management are outside the current product scope.

## Development roadmap

Vitrinode is progressing towards its first release through implementation, testing and recorded engineering acceptance.

**Current development priorities**

- Short-lived SSH certificate access and host-side enforcement.
- Vitrinode-HSM acceptance, provisioning and server integration.
- Executor trust-boundary hardening and clock-health enforcement.

**Subsequent development**

- Full per-host inventory presentation and historical change tracking.
- VMware vSphere/ESXi read-only correlation.
- Production HSM key protection, trusted time, auditing and sealing.
- Additional operational interfaces and platform validation.

Roadmap items are development intentions, not release commitments.

[Roadmap and development history →](https://vitrinode.com/about)

## Security and assurance

Vitrinode is designed around least privilege, explicit device identity, constrained execution and auditable administrative activity.

Security engineering and validation are ongoing. The platform has not undergone independent penetration testing or formal compliance certification.

Published technical documentation distinguishes implemented controls, development prototypes and planned security protections.

[Read the security model and limitations →](https://vitrinode.com/security)

## Availability

**Vitrinode is under active development towards v1.0.**

There is no announced general availability date.

Technical enquiries, questions about compatibility and expressions of interest are welcome. Commercial agreements, licensing and formal technical evaluations will be considered from the v1.0 release.

[Make an enquiry →](https://vitrinode.com/evaluate)

## About Vitrinode Ltd

Vitrinode is developed by **Vitrinode Ltd**, an independent software development company based in Barnsley, South Yorkshire, United Kingdom.

Registered in England and Wales.

**Company number:** 17494950

**Website:** [vitrinode.com](https://vitrinode.com)  
**Contact:** [hello@vitrinode.com](mailto:hello@vitrinode.com)

---
