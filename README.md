# Workstation Hardening

This repository documents a practical, security-focused approach to hardening **workstations** used for daily administration, development, and infrastructure access across multiple operating systems.

The goal is to establish a **repeatable, auditable baseline** for workstation security that reflects real-world operational needs rather than checkbox compliance.

This project is written in the style of **internal IT / MSP documentation**, emphasizing *why* controls exist, *what risks they mitigate*, and *how to implement them safely*.

---

##  Project Goals

- Establish secure default postures for workstations
- Reduce credential exposure and attack surface
- Document security decisions and tradeoffs
- Provide modular, incremental hardening guidance
- Support multi-platform environments

This is **not** an automated hardening script.

It is a **documented baseline** intended for learning, auditing, and controlled rollout.

---

This project focuses on **workstation-side security**, not servers or network perimeter devices.

### In Scope
- SSH key handling and agent security
- Local credential protection
- Session and user-space hardening
- Desktop and OS-specific security controls
- Operational safeguards for admin workstations

### Out of Scope (for now)
- Server hardening
- Enterprise IAM systems
- Full compliance frameworks (CIS, NIST, etc.)
- Network appliance configuration

---

##  Repository Structure

workstation-hardening/
├── README.md
└── docs/
    ├── linux/
    │   ├── ssh-agent-hardening.md
    │   └── README.md   (planned)
    └── windows/
        └── README.md   (planned)

