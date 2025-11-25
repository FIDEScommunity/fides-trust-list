# FIDES Sandbox Trust Lists (LOTL & TL, TLv6)

> **Status:** Sandbox / demo • **Spec:** ETSI TS 119 612 (TLv6) • **Last updated:** 2025-11-25

This repository hosts the **FIDES Sandbox List of Trusted Lists (LOTL)** and the **FIDES Sandbox Trusted List (TL)** used in pilots around **organizational wallets** and **verifiable credentials**.

- The **TL** is **PKI-anchored**: `ServiceDigitalIdentity` contains **X.509** identifiers (leaf certificate and/or Subject Key Identifier).
- The **LOTL** provides a **pointer** to the TL and includes the **X.509 identity** of the TL signer so clients can validate the TL signature chain.
- Both documents follow **Trusted List v6 (TLv6)** from **ETSI TS 119 612** and are designed for interoperability testing in a sandbox context.

---

## 🔗 Files / Endpoints

### LOTL (List of Trusted Lists)
- **Human-readable (GitHub):** `https://github.com/FIDEScommunity/fides-trust-list/FIDES-LOTL.xml`
- **Machine-readable (raw):** `https://raw.githubusercontent.com/FIDEScommunity/fides-trust-list/main/FIDES-LOTL.xml`

### TL (Trusted List)
- **Human-readable (GitHub):** `https://github.com/FIDEScommunity/fides-trust-list/FIDES-TL.xml`
- **Machine-readable (raw):** `https://raw.githubusercontent.com/FIDEScommunity/fides-trust-list/main/FIDES-TL.xml`

> Relying parties SHOULD use the **raw** URLs for programmatic download and caching.

---

## 📘 About (modeled after EWC)

Inspired by the **EWC Trust List** structure, this repo contains:
- A **LOTL** (`FIDES-LOTL.xml`) with a pointer to the FIDES TL.
- A **TL** (`FIDES-TL.xml`) listing **EAA** services (issuance of electronic attestations of attributes).
- Clear **scheme rules**, **onboarding instructions**, and **validation guidance**.
- XAdES guidance for **signing** both LOTL and TL with the **FIDES sandbox QSeal**.

---

## 🧩 Service Types (current)

- **EAA (Issuance of electronic attestations of attributes)**  
  URI: `http://uri.etsi.org/TrstSvc/Svctype/EAA`

---

## 🧪 What’s different in this sandbox?

- The **TL SDI** uses **X.509** anchors: at least one of
  - `<X509Certificate>`: base64 of DER **leaf** certificate (1 line, no headers), and/or
  - `<X509SKI>`: base64 of the **Subject Key Identifier** (20-byte value) from the leaf cert.
- The **LOTL**’s pointer to the TL includes the **TL signer certificate** in its `ServiceDigitalIdentities` to let clients establish trust in the TL signature.
- Optional **DID** references can be provided as informational links (e.g., issuer metadata) but are **not** used for trust anchoring in this PKI-first profile.

---

## 📝 What you’ll find in `FIDES-TL.xml` (TL)

- **SchemeTypeCommunityRules / PolicyOrLegalNotice (TSLPolicy):** links to these README sections.
- **SchemeTerritory:** `NL` (preferred for validator compatibility).
- **DistributionPoints:** GitHub and **raw** URL.
- **TSLSequenceNumber / ListIssueDateTime / NextUpdate:** versioning & refresh policy.
- **ServiceDigitalIdentity (SDI):** **X.509** anchors (`X509Certificate` and/or `X509SKI`).
- **ServiceInformationExtensions:** optional pointers to issuer metadata (e.g., OIDC/OID4VCI endpoint or DID page).
- **Signature:** to be **XAdES (enveloped)** over the `TrustServiceStatusList` element with the **FIDES sandbox QSeal**.

---

## 🧭 Scheme Rules

> How relying parties should interpret and use the **FIDES Sandbox LOTL & TL (TLv6)**.  
> This sandbox is **not** an official national TL/LOTL. Consumers must explicitly **opt-in** to trust it.

### 1) Purpose & Scope
- **Scope:** pilots around organizational wallets & verifiable credentials.
- **Audience:** developers and relying parties experimenting with TLv6 consumption.
- **Trust model:** **PKI-first**. TL SDI contains **X.509** identifiers (leaf cert and/or SKI). DID references are optional and informative only.

### 2) Identification & SDI Policy (TL)
- `ServiceDigitalIdentity` MUST contain at least:
  - `<X509Certificate>` (base64 DER of the **leaf** used to sign credentials) **or**
  - `<X509SKI>` (base64 of the **Subject Key Identifier** from that leaf).
- MAY additionally contain:
  - intermediate chain certificates as extra `<DigitalId>` entries,
  - `<X509SubjectName>`,
  - informational links (e.g., DID, issuer metadata) in `ServiceInformationExtensions`.

### 3) Matching Rules (verifier)
A verifier MUST anchor the signer of a credential to a **granted** TL service by matching the credential’s **leaf certificate** (from `x5c[0]`) to the TL SDI via **one** of:

1. **Exact certificate match:** credential’s leaf DER equals `<X509Certificate>` (byte-equal after base64 decode).  
2. **SKI match:** credential’s leaf **SKI** (RFC 5280 §4.2.1.2) equals `<X509SKI>` (base64).  
3. *(Optional)* **SPKI hash equivalence:** if published as an additional rule in `ServiceInformationExtensions`.

Additional policy checks (recommended):
- Certificate time validity, optional revocation (OCSP/CRL) if configured in the sandbox.
- Service **status** semantics: `granted`, `undersupervision` (warn), `withdrawn` (reject new artefacts).

### 4) LOTL Pointers
- The LOTL (`FIDES-LOTL.xml`) contains a pointer to the TL with the **TL signer identity** (`X509Certificate`), the **TSLLocation** (raw URL), MIME type, and additional descriptive fields.
- Clients SHOULD use the LOTL to discover & validate the TL, then use the TL to validate services.

### 5) Versioning & Updates
- **TSLSequenceNumber:** +1 on each change (both LOTL & TL).
- **ListIssueDateTime:** issuance time (UTC, ISO 8601 with `Z`).
- **NextUpdate:** typically **+30 days**; emergency updates may occur sooner.
- **Caching:** consumers SHOULD respect HTTP caching; MUST re-fetch if `ETag`/`Last-Modified` changes before `NextUpdate`.

### 6) Distribution
- Preferred machine URLs (raw):
  - LOTL: `https://raw.githubusercontent.com/FIDEScommunity/fides-trust-list/main/FIDES-LOTL.xml`
  - TL:   `https://raw.githubusercontent.com/FIDEScommunity/fides-trust-list/main/FIDES-TL.xml`

### 7) Signature
- Both LOTL and TL SHOULD be signed as **XAdES (enveloped)** over the `TrustServiceStatusList` element using the **FIDES sandbox QSeal**.
- Canonicalization: **exclusive c14n** (without comments).
- Include **KeyInfo** with **leaf + intermediate(s)**.

---

## 🚀 Onboarding / requesting changes

### A) Add or modify a TL service (TSP)
Open a **Pull Request** including:
1. **TSP (issuer) name & website**  
2. **ServiceTypeIdentifier** (`EAA`) & **ServiceName**  
3. **ServiceDigitalIdentity (SDI)**:  
   - `<X509Certificate>` (base64 DER of the **leaf** used to sign credentials), and/or  
   - `<X509SKI>` (base64 of the leaf’s SKI)  
   - *(optional)* intermediates, subject name  
4. **ServiceInformationExtensions** (optional): issuer metadata, profile links, etc.  
5. **ServiceStatus** and **StatusStartingTime** (UTC), contact email for change notifications.

> Every merge MUST bump `TSLSequenceNumber` and refresh `ListIssueDateTime` / `NextUpdate`.

### B) Update LOTL pointer
For the LOTL, provide:
- `X509Certificate` of the **TL signer** (leaf),  
- `TSLLocation` (raw TL URL),  
- and any optional additional information (scheme name/territory, rules, etc.).

---

## ✅ Validation (suggested)

- **XML schema & TLv6 structure:** `TSLVersionIdentifier=6`, `TSLTag`, required scheme fields present.
- **Signature (XAdES):** verify enveloped signature over root, **exclusive c14n**.
- **TL matching:** confirm that `x5c[0]` (leaf) from a test credential matches TL SDI via certificate or SKI.

> Useful tools: **EU DSS** (sign/validate), `xmlsec`, `openssl`, `xades4j`.

---

## 📄 File list

- `FIDES-LOTL.xml` — FIDES Sandbox List of Trusted Lists (pointers to TLs; currently the FIDES TL).
- `FIDES-TL.xml` — FIDES Sandbox Trusted List (services and their X.509 identities).

---

## 🔐 Governance

- **Releases:** PR + review → merge → bump `TSLSequenceNumber`  
- **Publishing cadence:** typical **NextUpdate** = 30 days  
- **Scope:** Sandbox/demo for organizational-wallet pilots  
- **Contact:** info@fides.community

---

## 📚 References

- ETSI TS **119 612** (TLv6) — Trusted Lists  
- EC **DSS** — signing & validation (XAdES)  
- EWC Trust List — structure & onboarding inspiration

---

## 📜 License

Unless stated otherwise, content in this repo is provided under the **Apache-2.0** license.
