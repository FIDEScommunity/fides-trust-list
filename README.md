# FIDES Sandbox Trusted List (TLv6)

> **Status:** Sandbox / demo • **Spec:** ETSI TS 119 612 (TLv6) • **Last updated:** 2025-10-23

This repository hosts the **FIDES Sandbox Trusted List** used in pilots around **organizational wallets** and **verifiable credentials**.  
It follows **Trusted List v6 (TLv6)** structure from **ETSI TS 119 612** and intentionally demonstrates a **DID‑first SDI** variant for learning and interoperability testing.

---

## 🔗 Trust List

- **Human-readable (GitHub):** `https://github.com/FIDEScommunity/fides-trust-list/FIDES-TL.xml`  
- **Machine-readable (raw):** `https://raw.githubusercontent.com/FIDEScommunity/fides-trust-list/main/FIDES-TL.xml`

> Consumers should prefer the **raw** URL when programmatically downloading the TL XML.

---

## 📘 About (modeled after EWC)

Inspired by the structure of the **EWC Trust List**, this repo includes:
- A single **`FIDES-TL.xml`** file maintained via PRs.
- Simple **onboarding instructions** for issuers/services to be added.
- A clear statement of **service types** and **policy/rules** references.

This FIDES Sandbox focuses on **EAA (issuance of electronic attestations of attributes)** services related to **organizational wallets**. The sandbox deliberately includes **DID-only SDI** to show how a DID can serve as a stable identifier alongside—or in place of—X.509 in experimental setups.

---

## 🧩 Service Types (current)

- **EAA (Issuance of electronic attestations of attributes)**  
  URI: `http://uri.etsi.org/TrstSvc/Svctype/EAA`

> In production ecosystems, EAA services are commonly **PKI-anchored** (SDI contains **X.509**). In this sandbox we also publish an **experimental DID-only SDI** to illustrate a DID-first trust anchor.

---

## 🧪 Sandbox Deviations (read me!)

This sandbox TL intentionally demonstrates the following **deviation** from common practice:

- **SDI as DID-only (experimental):**  
  Services include `<ServiceDigitalIdentity><DigitalId><Other><URI>did:web:…</URI></Other></DigitalId></ServiceDigitalIdentity>` instead of X.509.  
  This helps demonstrate **stable identifiers** across key rotations and improves **developer experience** for DID‑aware tools.

**Implications:** Some TLv6 clients and validators may **expect X.509 in SDI** for EAA services. Treat this TL as **educational/demo material**.

---

## 📝 Scheme information you will find in `FIDES-TL.xml`

- **SchemeTypeCommunityRules:** link to rules explaining scope, DID‑binding, key‑match rule (e.g., SPKI‑SHA256), release cadence.
- **PolicyOrLegalNotice/TSLPolicy:** legal/policy page.
- **SchemeTerritory:** typically `NL` (country code) for best validator compatibility.
- **DistributionPoints:** includes both GitHub and raw URLs.
- **TSLSequenceNumber / ListIssueDateTime / NextUpdate:** versioning and refresh policy.
- **Signature:** to be signed as **XAdES (enveloped)** over the `TrustServiceStatusList` element (exclusive c14n) with the **FIDES sandbox eSeal**.

> The current file may be unsigned while under active editing. Before publication, ensure the TL is **XAdES-signed**.

---

## 🚀 Onboarding / requesting changes

Open a **Pull Request** with the following minimum information:

1. **TSP (issuer) name & website**  
2. **ServiceTypeIdentifier & ServiceName**  
3. **ServiceDigitalIdentity (SDI)**  
   - **DID-only SDI (sandbox):** your `did:web` (and `#key` if needed)   
4. **ServiceInformationExtensions (optional but recommended):**  
   - DID-binding URIs (e.g., `https://fides.community/tl/asi/did-binding#did`, `#vm`, `#match=spkiSha256`)  
   - Any profile/schema references for your credential types  
5. **Status/StartingTime** and contact email for change notifications.

> Every merge **must** bump `TSLSequenceNumber` and refresh `ListIssueDateTime` / `NextUpdate`.

---

## ✅ Validation (suggested)

- **XML schema & TLv6 structure** — validate against TLv6 (`TSLVersionIdentifier=6`)  
- **Signature (XAdES)** — when signed, verify enveloped signature over root, exclusive c14n  
- **SDI equivalence checks:**  
  - For **DID SDI**: resolve `did:web`, compare public key (SPKI/JWK thumbprint) with declared binding rule

> Tools: **EU DSS** (supports TLv6), `xmlsec`, `openssl`, `xades4j`

---

## 📄 Files

- `FIDES-TL.xml` — the Trusted List (TLv6).

---

## 🔐 Governance

- **Releases:** PR + review → merge → bump `TSLSequenceNumber`  
- **Publishing cadence:** `NextUpdate` typically set to **+30 days**  
- **Scope:** Sandbox/demo for organizational-wallet pilots  
- **Contact:** info@fides.community

---

## 📚 References

- ETSI TS **119 612** (TLv6) — Trusted Lists  
- EC **DSS** — TLv6 support and guidance  
- EWC Trust List — structure & onboarding inspiration

---

## 📜 License

Unless stated otherwise, content in this repo is provided under the **Apache-2.0** license.

---

## 🧭 Scheme Rules

> This section describes how relying parties should interpret and use the **FIDES Sandbox Trusted List (TLv6)**.
> The sandbox is **not** an official national TL/LOTL derivative. Consumers must explicitly opt-in to trust it.

### 1) Purpose & Scope
- **Scope:** pilots around organizational wallets & verifiable credentials.
- **Audience:** developers and relying parties experimenting with TLv6 consumption.
- **Trust model:** **DID‑first**. Services use **DID‑only SDI** instead of X.509 to demonstrate a DID anchor.
- **Out of scope:** production assurance or legal reliance; this list is educational/demo material.

### 2) Identification & SDI Policy
- **ServiceDigitalIdentity (SDI)** uses **`did:web` URIs** only.
- The DID **MUST** resolve over HTTPS to a valid DID document.
- The public key used by the issuer to sign artefacts **MUST** be present in the DID document’s `verificationMethod` and referenced via `assertionMethod`.
- DID documents **SHOULD** expose keys as JWK (preferred) and **MAY** additionally include `x5c` if a certificate chain is published for interoperability.

### 3) DID Binding & Key‑Match Rules
Relying parties **MUST** verify that the key used to sign a credential/presentation **matches** the key referenced by the DID in the SDI.

Accepted match strategies (any one is sufficient):
1. **DID URL key reference**: the JOSE/COSE `kid` contains the DID URL of the key (e.g., `did:web:…#key-1`). The verifier resolves the DID and matches the key by id.
2. **JWK thumbprint**: compute RFC 7638 thumbprint of the JWK from the artefact and compare to the JWK from the DID document.
3. **SPKI hash** *(optional)*: compute SHA‑256 over SubjectPublicKeyInfo (SPKI) of the artefact’s public key and compare to the SPKI derived from the DID document’s key.
4. **x5c leaf equivalence** *(optional)*: if the artefact provides `x5c`, compare the leaf certificate’s SPKI to the DID key (or the DID key’s `x5c` leaf when present).

**Algorithms:** the sandbox focuses on **ES256 (P‑256)** and **EdDSA (Ed25519)**. Other algorithms may not be tested.

### 4) Service Status Semantics
- `granted` — service is active and trusted.
- `undersupervision` — service is active but under investigation; relying parties **SHOULD warn**.
- `withdrawn` — service is not trusted; relying parties **MUST reject** signatures from that service for new artefacts.
- Status changes are recorded via **ServiceHistory** when applicable.

### 5) Versioning & Updates
- **TSLSequenceNumber**: increment by **+1** on each change.
- **ListIssueDateTime**: set to issuance time (UTC, ISO‑8601 with `Z`).
- **NextUpdate**: typically **+30 days**; emergency updates may occur sooner.
- **Caching:** consumers **SHOULD** respect HTTP caching, but **MUST** re‑fetch if `ETag`/`Last‑Modified` changes before `NextUpdate`.

### 6) Distribution
- Preferred machine URL: `https://raw.githubusercontent.com/FIDEScommunity/fides-trust-list/main/FIDES-TL.xml`  
- GitHub HTML view: `https://github.com/FIDEScommunity/fides-trust-list/FIDES-TL.xml`

### 7) Signature
- The TL **SHOULD** be signed as **XAdES (enveloped)** over the `TrustServiceStatusList` element using the **FIDES sandbox eSeal**.
- Canonicalization: **exclusive c14n**.
- While unsigned drafts may exist during editing, relying parties **SHOULD** prefer signed releases.

### 8) Onboarding Requirements
When requesting a new TSP/service entry via PR, provide:
1. **TSP name** and contact URL.
2. **ServiceTypeIdentifier** and **ServiceName**.
3. **SDI DID**: `did:web:…` (and optional key id `#…`).
4. **Optional**: `ServiceInformationExtensions` with DID‑binding URIs, credential profile links, or additional semantics.
5. **ServiceStatus** and **StatusStartingTime** (UTC).

### 9) Change & Incident Handling
- **Key rotation**: update the DID document to point to the new key **and** submit a PR to reflect changes if service metadata changes.
- **Compromise**: set status to `withdrawn` (or `undersupervision` while investigating) and document the incident in the PR/commit message.
- **Backward validation**: relying parties MAY keep historical keys for validating artefacts issued before a withdrawal, per their own policy.

### 10) Deviations & Compatibility Notes
- This sandbox intentionally uses **DID‑only SDI** for EAA‑like services to showcase a DID‑first approach.
- Some TL clients expect X.509 in SDI; such clients may not consume this list without adaptation.
