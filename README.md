# Vitrinode

**Inventory and controlled operations for Linux estates.**

Vitrinode builds secure, identity-first infrastructure management software for engineers and small teams managing Debian and Ubuntu hosts. It brings device identity, host visibility and signed remote jobs into one management plane, with Docker inventory and read-only Proxmox VE correlation.

**[Website](https://vitrinode.com) · [Product](https://vitrinode.com/product) · [Architecture](https://vitrinode.com/architecture) · [Security model](https://vitrinode.com/security)**

## What Vitrinode does

Vitrinode is built for the point where a Linux estate grows beyond a handful of machines, but doesn't warrant an unnecessarily complex management platform. It complements existing SSH and configuration-management workflows rather than replacing them.

**Implemented capabilities** include:

- **Device identity and connectivity:** client-generated private keys, one-time enrolment, mutually authenticated TLS, per-request revocation checks and host heartbeats.
- **Inventory collection:** system, storage, networking, systemd, APT and Docker facts from managed Debian and Ubuntu hosts.
- **Controlled remote operations:** signed, expiring, replay-protected jobs using a fixed set of typed operations, with a separate privileged local executor.
- **Administrator authentication:** passwordless WebAuthn/FIDO2 login and server-enforced step-up authentication for sensitive actions.
- **Device attestation:** TPM-backed quotes where available, distinguished from weaker software evidence and used in device trust decisions.
- **Proxmox VE integration:** read-only VM and container inventory, power-state correlation and administrator-confirmed links to enrolled agents.

The full per-host inventory interface, change history, VMware correlation and short-lived SSH certificate access remain **planned**. Proxmox lifecycle operations and general-purpose remote shell execution are not part of the implemented feature set.

[See the capability matrix and supported environments →](https://vitrinode.com/product)

## Architecture and security

Vitrinode separates the management plane, the network-facing agent and the privileged local executor. Agents initiate all connections to the control plane; managed hosts do not need inbound management ports opened for the server.

Security boundaries are deliberately distinct:

- **Administrator ↔ control plane:** WebAuthn/FIDO2 and server-side sessions.
- **Control plane ↔ agent:** mutual TLS with device certificates and revocation enforcement.
- **Agent ↔ executor:** permission-checked local Unix socket with a constrained, typed operation set.
- **Control plane ↔ Proxmox:** dedicated read-only API credentials over TLS with explicit certificate trust.

The system uses established cryptographic primitives, separates keys by purpose and audits authentication, device administration and remote-job lifecycles. Device authentication is **not** treated as proof of device attestation.

Vitrinode has **not** undergone independent penetration testing or formal compliance certification. Design goals and future protections are distinguished from implemented controls in the published documentation.

[Explore the architecture →](https://vitrinode.com/architecture) · [Read the security model →](https://vitrinode.com/security)

## Vitrinode-HSM

[Vitrinode-HSM](https://vitrinode.com/hsm) is a separate hardware-signing project under active development, built around the RP2350. Its purpose is to hold the planned SSH user certificate authority key on dedicated hardware and permit only policy-bounded OpenSSH user certificate issuance—not arbitrary signing or private-key export.

End-to-end certificate signing and OpenSSH authentication have been demonstrated on physical hardware. Persistent CA storage, production key protection, trusted time and control-plane integration are **not yet complete**. The HSM is not part of the deployed management system.

[Hardware design, threat model and implementation status →](https://vitrinode.com/hsm)

## Status and evaluation

Vitrinode is **in active development**. There is no announced general availability date.

Technical evaluations can be arranged for a defined use case, beginning with a lab or isolated environment. Scope, compatibility, permitted operations and success criteria are agreed individually.

[Request a technical evaluation →](https://vitrinode.com/evaluate) · [Roadmap and development history →](https://vitrinode.com/about)

## About

**Vitrinode Ltd** is a software development company based in Barnsley, South Yorkshire, United Kingdom.

[**vitrinode.com**](https://vitrinode.com) · [**hello@vitrinode.com**](mailto:hello@vitrinode.com)
