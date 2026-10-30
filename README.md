
# CA/Browser Forum S/MIME Baseline Requirements: Infrared certificate profiles

Machine-readable subscriber certificate profiles for the CA/Browser Forum *Baseline Requirements for the Issuance and Management of Publicly-Trusted S/MIME Certificates* (S/MIME BR), written against the Infrared certificate profile schema developed by Corey Bonnell:

    https://infrared.turbolightsolutions.com/infrared-profile-2026-10-23.schema.json

## Target version

S/MIME BR **v1.0.16** published September 28, 2026.
Where a requirement has a future effective date, the profile notes say which state is modeled.

## Files

| File | Covers | Policy OID |
|---|---|---|
| `cabf-smime-sbr-common.json5` | Shared library (`$defs` only, not loadable on its own) | n/a |
| `cabf-smime-sbr-mailbox-validated-strict.json5` | Mailbox-validated, Strict | 2.23.140.1.5.1.3 |
| `cabf-smime-sbr-organization-validated-strict.json5` | Organization-validated, Strict | 2.23.140.1.5.2.3 |
| `cabf-smime-sbr-sponsor-validated-strict.json5` | Sponsor-validated, Strict | 2.23.140.1.5.3.3 |
| `cabf-smime-sbr-individual-validated-strict.json5` | Individual-validated, Strict | 2.23.140.1.5.4.3 |
| `cabf-smime-sbr-mailbox-validated-multipurpose.json5` | Mailbox-validated, Multipurpose | 2.23.140.1.5.1.2 |
| `cabf-smime-sbr-organization-validated-multipurpose.json5` | Organization-validated, Multipurpose | 2.23.140.1.5.2.2 |
| `cabf-smime-sbr-sponsor-validated-multipurpose.json5` | Sponsor-validated, Multipurpose | 2.23.140.1.5.3.2 |
| `cabf-smime-sbr-individual-validated-multipurpose.json5` | Individual-validated, Multipurpose | 2.23.140.1.5.4.2 |

Legacy generation profiles are not provided. Issuance under Legacy profiles was
deprecated effective July 15, 2025 (Ballot SMC08).

Every profile pulls the library in through `$includes`, so the eight profile
files and the library must stay in the same directory. The files are plain JSON,
which is valid JSON5, so they load in either parser.

Every block carries a `clause` giving the S/MIME BR section it implements and a
`description` restating the normative text, so findings trace back to the BR and
the generated CPS documentation reads as policy rather than as tool output.
Profile-level `org` is `cabforum` and `spec` is `smime-br-1.0.16`.

## What the library provides

**Values** (used with `%label%` substitution)

* DER encodings of every `SubjectPublicKeyInfo` AlgorithmIdentifier permitted
  by SBR 6.1.5 and 7.1.3.1: RSA, P-256, P-384, P-521, Ed25519, Ed448,
  ML-DSA-44/65/87, ML-KEM-512/768/1024.
* DER encodings of every signature AlgorithmIdentifier permitted by SBR 7.1.3.2:
  RSASSA-PKCS1-v1_5 and RSASSA-PSS with SHA-256/384/512, ECDSA with
  SHA-256/384/512, Ed25519, Ed448, ML-DSA-44/65/87. Each encoding was verified
  by decoding it with OpenSSL.
* The twelve reserved policy identifiers from SBR 7.1.6.1 (only the eight
  Strict and Multipurpose ones are referenced), `id-kp-emailProtection`,
  `id-kp-clientAuth`, `id-ad-ocsp` and `id-ad-caIssuers`.

**Blocks** (used with `$use`)

* `version_v3`, `serialNumber_subscriber`, `signatureAlgorithm_sbr`,
  `signatureValue_any`, `subjectPublicKey_sbr`, `issuer_subordinateCa`.
* Extension entries: `ext_crlDistributionPoints_strict` (shared by both
  generations, since the URI scheme rule is the same), `ext_authorityInformationAccess_strict`,
  `ext_basicConstraints_subscriber`, `ext_authorityKeyIdentifier_subscriber`,
  `ext_subjectKeyIdentifier_subscriber`, `ext_subjectAltName_mailbox`,
  `ext_subjectAltName_mailbox_nonCritical`, `ext_extKeyUsage_strict`,
  `ext_extKeyUsage_multipurpose`.
* Key usage blocks, one per key algorithm family plus a union block per
  generation: `ext_keyUsage_strict_any`, `ext_keyUsage_strict_rsa`,
  `ext_keyUsage_strict_ec`, `ext_keyUsage_strict_signatureOnly` (EdDSA and
  ML-DSA), `ext_keyUsage_strict_mlkem`, `ext_keyUsage_multipurpose_any`,
  `ext_keyUsage_multipurpose_rsa`.

## What the profiles check

* **Version** v3; **serial number** greater than zero and below 2^159 (SBR 7.1).
* **Validity** of at most 825 days for both generations (SBR 6.3.2). RFC 5280
  validity is inclusive of `notAfter`, so the check is
  `notAfter - notBefore <= 71,279,999 seconds`.
* **Public key** algorithm restricted to the permitted encodings; RSA modulus
  at least 2048 bits; RSA exponent odd and at least 3 (SBR 6.1.5, 6.1.6).
* **Issuer** containing countryName, organizationName and commonName
  (SBR 7.1.4.3.1), with optional state, locality and organizationIdentifier.
* **Subject** limited to the attributes each certificate type and generation
  permits (SBR 7.1.4.2.3 to 7.1.4.2.6), with: DirectoryString attributes
  restricted to PrintableString or UTF8String; countryName as two upper-case
  letters; emailAddress and the Mailbox-validated commonName as a mailbox
  address; organizationIdentifier matching the Appendix A Registration
  Reference format; RFC 5280 upper bounds on attribute length.
* **certificatePolicies**: exactly the reserved policy identifier for the
  profile; id-qt-cps, if present, an HTTP or HTTPS URL (SBR 7.1.2.3 a).
* **cRLDistributionPoints**: present, at least one HTTP URI, every URI HTTP;
  a second optional HTTP-only distribution point is modeled for dual-CRL
  deployments (SBR 7.1.2.3 b).
* **authorityInformationAccess**: optional, non-critical, caIssuers and OCSP
  locations HTTP only (SBR 7.1.2.3 c).
* **basicConstraints**: optional, cA not true (SBR 7.1.2.3 d).
* **keyUsage**: present, bits limited to those the generation allows across all
  key algorithms (SBR 7.1.2.3 e).
* **extKeyUsage**: Strict allows only id-kp-emailProtection. Multipurpose
  requires id-kp-emailProtection and allows id-kp-clientAuth (SBR 7.1.2.3 f and
  Ballot SMC018, see Known issues).
* **authorityKeyIdentifier**: present, non-critical, keyIdentifier present
  (SBR 7.1.2.3 g).
* **subjectAltName**: present, at least one rfc822Name mailbox address;
  constrained to non-critical for OV and SV, where the subject is never empty
  (SBR 7.1.2.3 h, 7.1.4.2.1).
* **subjectKeyIdentifier**: optional, non-critical (SBR 7.1.2.3 n).

### Strict versus Multipurpose

The two generations differ only in: the policy identifier; RSA key usage
(Multipurpose allows dataEncipherment alongside keyEncipherment); extended key
usage; streetAddress and postalCode in OV, SV and IV subjects (allowed in
Multipurpose, prohibited in Strict); and the set of additional extensions the
BR tolerates (see Limitations).

## Limitations

### Schema limitations

1. **"SHALL NOT be present" cannot be stated.** The schema has no negative
   assertion. Prohibited attributes and extensions are simply omitted, so they
   surface as *undocumented* findings rather than violations. Read undocumented
   findings against the descriptions, which name the prohibited items.
2. **Unmodeled extensions.** The schema has no blocks for smimeCapabilities,
   qcStatements, the Legal Entity Identifier extensions (1.3.6.1.4.1.52266.1
   and .2), the Adobe Timestamp and ArchiveRevInfo extensions, or any other
   extension the BR permits in Multipurpose. All of these are reported as
   undocumented when present, even where the BR allows them. The reverse also
   holds: the schema cannot flag a critical qcStatements extension or a
   subjectDirectoryAttributes extension as the violations they are.
3. **otherName in subjectAltName.** An `id-on-SmtpUTF8Mailbox` otherName is
   skipped by pattern matching and is not value-checked. A certificate whose
   only mailbox is an SmtpUTF8Mailbox otherName will fail the `size: min 1`
   rule on rfc822Name entries. Any other supported GeneralName type in the SAN
   (dNSName, iPAddress, URI, directoryName) is reported as a mismatch.
4. **Key usage is not conditional on key algorithm.** The default
   `_any` block allows the union of all algorithms' bits, so an EC key with
   keyEncipherment set would pass. For single-algorithm issuance, substitute
   the matching per-algorithm block. The SBR usage modes (signing only, key
   management only, dual use) are also not expressible; a keyUsage of
   nonRepudiation alone would pass this schema but fail the BR.
5. **Cross-field rules** are not expressible: the RSA modulus being evenly
   divisible by 8; every Mailbox Address in the subject being repeated in the
   SAN (7.1.4.2.1); Sponsor-validated certificates needing givenName and
   surname, or pseudonym; pseudonym and givenName/surname not appearing
   together; pathLenConstraint being absent (it would appear as undocumented).
6. **Policy qualifiers.** Only id-qt-cps is modeled. An id-qt-unotice
   qualifier is reported as undocumented, and its explicitText and noticeRef
   rules cannot be checked. CA-defined policy identifiers permitted alongside
   the reserved OID are also reported as undocumented until the CA operator
   adds them.
7. **authorityCertIssuer and authorityCertSerialNumber** in the AKI are
   unchecked and reported as undocumented; the BR prohibits them.
8. **Serial number entropy** and non-sequential allocation cannot be judged
   from a single certificate.
9. **Issuer identity.** The issuer block is generic. It cannot confirm that the
   issuer is byte-for-byte identical to the issuing CA's subject, or that the
   issuing CA is itself an S/MIME CA under SBR 7.1.2.2.

### Modeling choices

* **SHOULD-level criticality is left unconstrained.** keyUsage (SHOULD be
  critical), certificatePolicies and cRLDistributionPoints (SHOULD NOT be
  critical) are not pinned, so a BR-compliant certificate that departs from the
  recommendation is not flagged. AKI and AIA are pinned to non-critical because
  those are SHALL NOT requirements. SAN criticality is pinned only where the
  subject can never be empty.
* **Multipurpose EKU models the SMC018 end state.** From July 1, 2027 only
  id-kp-emailProtection and id-kp-clientAuth are permitted. Until then the BR
  still allows other values (for example document signing EKUs); those will
  show as undocumented, not as violations, so the profile is usable now and
  needs no change at the transition date.
* **Signature algorithms are the full BR set.** The algorithm actually used is
  fixed by the issuing CA key. A CA operator should narrow
  `signatureAlgorithm_sbr` and `subjectPublicKey_sbr` to what it issues, and
  should pin the issuer with `tlvHexEq`.
* **The common second CRL distribution point** is optional and exists only so
  dual-CRL deployments are not reported as undocumented. The BR requires at
  least one distribution point; it does not require two.
* **Length bounds** come from RFC 5280 Appendix A upper bounds, not from the
  S/MIME BR, which sets none of its own.

