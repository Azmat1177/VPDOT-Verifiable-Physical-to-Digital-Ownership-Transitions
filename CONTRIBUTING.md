# Contributing to VPDOT

Thank you for your interest in VPDOT. This project is currently in its
**research and pre-development phase** (Ethereum Foundation PhD Fellowship 2026).

---

## Current Status

No code has been implemented yet. Development begins upon fellowship approval,
following the four-milestone research plan described in the README.

---

## How to Get Involved

### During Research Phase (Now)

- **Open an Issue** to discuss the architecture, suggest improvements to the
  state machine design, or flag regulatory considerations we may have missed
- **Watch the repository** to be notified when development begins
- **Reach out directly** if you represent a pharmaceutical SME in Italy and
  are interested in participating in the pilot evaluation (Milestone 3)

### During Development Phase (Months 4–12)

Once development begins, contributions are welcome in the following areas:

- **Smart contract review** — security analysis of Solidity state machine and escrow logic
- **Firmware** — ESP32 SRAM-PUF implementation and sensor integration
- **Gas optimisation** — reducing per-shipment L2 costs
- **Documentation** — deployment guides for non-technical SME operators
- **ERC/EIP drafting** — Verifiable Physical Identity (VPI) standard

---

## Code Standards (Planned)

### Solidity
- Solidity ≥ 0.8.20
- OpenZeppelin contracts for AccessControl, ERC-721, and ECDSA
- NatSpec documentation on all public functions
- 100% test coverage target via Hardhat

### Firmware (C++)
- ESP-IDF framework
- No persistent key storage — SRAM-PUF only
- All attestation packets must include nonce for replay prevention

---

## Contact

**Azmat Ullah**  
📧 azmat.ullah@unicam.it  
🔗 [GitHub](https://github.com/azmat1177)
