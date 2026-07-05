# Token2 PIN+ OpenPGP Applet — SmartPGP Fork, Attribution & Notices

This repository is a **fork of [SmartPGP](https://github.com/ANSSI-FR/SmartPGP)**,
the free and open-source JavaCard implementation of the OpenPGP card
specification published by ANSSI (Agence nationale de la sécurité des systèmes
d'information, France).

SmartPGP is licensed under the **GNU General Public License, version 2 or later
(GPLv2+)**. The OpenPGP applet in Token2 PIN+ Release3-series devices is a
derivative work of SmartPGP and is therefore distributed under the same licence.

## A note on timing

We have used the SmartPGP applet as the basis of our OpenPGP implementation for
a couple of years now. We are formalizing the fork, the attribution to ANSSI,
and the GPLv2 compliance documentation in one place. This should have been done
from the outset; we are putting it right now (July 2026) — better late than
never. The sections below record the upstream basis, the changes we have made,
and the source-availability commitment required by the licence.

---

## Upstream

| | |
|---|---|
| Upstream project | SmartPGP |
| Upstream author | ANSSI-FR |
| Upstream URL | https://github.com/ANSSI-FR/SmartPGP |
| Upstream licence | GPLv2 or later |
| OpenPGP card spec | 3.4 |
| Base commit / release we forked from | <!-- TODO: exact upstream commit hash or release tag --> |

Original SmartPGP copyright © ANSSI. Modifications © TOKEN2 Sàrl.

---

## Attribution & source availability (NOTICE)

This product includes software derived from **SmartPGP** (copyright © ANSSI,
GPLv2+, https://github.com/ANSSI-FR/SmartPGP). Modifications for the Token2 PIN+
Release3 series are copyright © TOKEN2 Sàrl and are likewise licensed under
GPLv2+.

The complete text of the GNU General Public License, version 2, is provided in
the `LICENSE` file at the root of this repository. Because the Token2 OpenPGP
applet is a derivative work of SmartPGP, the corresponding source code of the
applet must be made available to recipients of the binary (CAP file) under the
terms of the GPL.

**Written offer for corresponding source** (GPLv2 §3):

TOKEN2 Sàrl offers to provide, to any recipient of the Token2 PIN+ OpenPGP
applet in binary form, the complete corresponding source code under the terms of
GPLv2. This covers the OpenPGP-card logic of our SmartPGP fork; it does not
include the secure element cryptographic libraries and our NDA-bound interfaces
to them, which are separate proprietary firmware supplied by the chip
manufacturers and are not derived from SmartPGP (see "ed25519 / x25519" below).

- Contact: <!-- TODO: e.g. support@token2.com -->
- Repository (source): <!-- TODO: URL where the applet source is published -->
- Period of availability: <!-- TODO: at least as required by GPLv2 §3 -->

*"Token2" and related marks are trademarks of TOKEN2 Sàrl. This notice concerns
copyright licensing only and grants no trademark rights.*

---

## What we changed

Items are marked **Verified** (observable on a shipped device), **Documented**
(stated in our release notes; code-level detail pending), or **To be completed**
(must be filled from our internal source tree; not asserted here as fact).

### Verified — observable via `gpg --card-status`

| Area | Upstream SmartPGP default | Token2 PIN+ | Notes |
|---|---|---|---|
| Applet AID (manufacturer bytes) | `d276000124010304AFAF…` (`AFAF` placeholder) | `d276000124010304`**`0011`**`…` | We use manufacturer ID `0x0011`. |
| Per-device serial number | Not provisioned by default | Unique per unit | Injected at production. The OpenPGP serial uses the **last 8 digits** of the device's (longer) Token2 serial number. |
| Default PIN retry counters (PW1 / RC / PW3) | Provisions a resetting code | `3 / 0 / 3` (resetting code not provisioned) | RC disabled by default. |
| Default signature-PIN mode | Configurable | `forced` | Ships in forced-PW1 mode. |
| Max PIN length | 127 | 127 | Unchanged (inherited from SmartPGP). |

### Documented — per firmware release

| Firmware | Change relative to the minimal initial build |
|---|---|
| Release3 (initial) | Minimal build due to storage constraints; RSA only. |
| Release3.1 | Added RSA4096, ed25519, x25519, and UIF functionality. |
| Release3.2 | Added KDF support; performance and compatibility improvements. |
| Release3.3 | Co-resides with PIV applet; other integration changes beyond the applet's scope. |

---

## ed25519 / x25519

Curve25519 support — ed25519 for the signature and authentication slots, x25519
(cv25519) for the encryption slot — is **not** present in upstream SmartPGP,
which implements RSA and short-Weierstrass ECC (NIST and brainpool curves) only.
Adding Curve25519 was new work on our side, introduced in Release3.1.

**Portable part (in this GPLv2 fork).** The OpenPGP-card logic for Curve25519
lives in our SmartPGP fork and follows the OpenPGP card specification:

- Algorithm-attribute handling using the standard curve OIDs — ed25519
  `1.3.6.1.4.1.11591.15.1` (signature / authentication) and x25519/cv25519
  `1.3.6.1.4.1.3029.1.5.1` (encryption).
- Routing of `GENERATE ASYMMETRIC KEY PAIR`, key import,
  `PSO:COMPUTE DIGITAL SIGNATURE` / `INTERNAL AUTHENTICATE` (ed25519) and
  `PSO:DECIPHER` (x25519) to the Curve25519 path.
- Public-key export (`7F49`) in the 32-byte little-endian form these curves use.
- Buffer / memory sizing adjusted to fit the new key type on our combined
  FIDO2 / OpenPGP devices.

**Hardware-specific part (not published).** The actual ed25519/x25519 operations
are performed by the secure element's own cryptographic library — NXP's on-chip
library on our JCOP-based devices, and the TMCOS cryptographic services on our
THD89-based devices. The applet drives those services; it does not perform the
curve arithmetic itself.

**Why this part is not upstreamed or published.** The chip cryptographic-library
interfaces from **both** vendors (NXP for JCOP, and the THD89/TMCOS vendor) are
provided to us under **non-disclosure agreements**, and the code that uses them
is **hardware-specific** — it would not compile or run on any other chip. For
both reasons we do not contribute it to upstream SmartPGP (it is not portable
JavaCard code and is of no use to a generic-JavaCard project) and it is not part
of the GPLv2 corresponding source we publish. That corresponding source covers
the portable OpenPGP-card logic above — the part actually derived from SmartPGP.
The chip cryptographic library and our NDA-bound interface to it are separate
proprietary firmware from the secure element manufacturer and are not a
derivative of SmartPGP.

---

## Syncing this fork with upstream

Our documentation is confined to this single file (plus `LICENSE`, unchanged),
so pulling upstream changes will not conflict with it.

One-time setup:
```
git remote add upstream https://github.com/ANSSI-FR/SmartPGP.git
```

Routine sync:
```
git fetch upstream
git checkout master            # or 'main'
git merge upstream/master      # or: git rebase upstream/master
git push origin master
```

**The one rule to avoid conflicts: add, don't overwrite.** This file is a new
path upstream does not have, so it always carries through. If you later publish
the actual applet modifications by editing files under `src/`, expect and
resolve conflicts on those specific files at sync time — that is normal for a
real code fork and does not affect this document.
