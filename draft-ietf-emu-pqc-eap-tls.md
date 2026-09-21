---
title: "Post-Quantum Enhancements to TLS-Based EAP Methods"
abbrev: "PQC Enhancements to TLS-Based EAP Methods"
category: std

docname: draft-ietf-emu-pqc-eap-tls-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "EAP Method Update"
keyword:
 - PQC
 - PQ/T Hybrid
 - TLS
 - EAP

venue:
  group: "EAP Method Update"
  type: "Working Group"
  mail: "emu@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/emu"


stand_alone: yes
pi: [toc, sortrefs, symrefs, strict, comments, docmapping]

author:
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "k.tirumaleswar_reddy@nokia.com"



normative:
  RFC9190:
  RFC9846:
  RFC9847:
  RFC9881:
  RFC9954:
  RFC10024:
  RFC7030:
  RFC8295:
  I-D.ietf-tls-mldsa:

informative:
  RFC5281:
  RFC7170:
  RFC8879:
  RFC8995:
  RFC9794:
  RFC9958:
  I-D.ietf-uta-pqc-app:
  I-D.ietf-lamps-pq-composite-sigs:
  I-D.reddy-tls-composite-mldsa:
  FIPS203:
    title: "Module-Lattice-Based Key-Encapsulation Mechanism Standard"
    author:
      org: "National Institute of Standards and Technology (NIST)"
    date: 2024
    seriesinfo:
      "FIPS": "203"
  FIPS204:
    title: "Module-Lattice-Based Digital Signature Standard"
    author:
      org: "National Institute of Standards and Technology (NIST)"
    date: 2024
    seriesinfo:
      "FIPS": "204"
  FIPS205:
    title: "Stateless Hash-Based Digital Signature Standard"
    author:
      org: "National Institute of Standards and Technology (NIST)"
    date: 2024
    seriesinfo:
      "FIPS": "205"

---

--- abstract

This document specifies the use of post-quantum cryptography in TLS-based EAP methods,
including the Extensible
Authentication Protocol with Transport Layer Security (EAP-TLS), EAP Tunneled TLS
(EAP-TTLS), Protected EAP (PEAP), and EAP Tunnel Method (TEAP). It also addresses
challenges related to large certificate sizes and long certificate chains, as identified
in {{?RFC9191}}, and specifies a mechanism to reduce TLS handshake size.

--- middle

# Introduction

The emergence of a Cryptographically Relevant Quantum Computer (CRQC) would break the
mathematical assumptions that underpin widely deployed public-key algorithms, rendering
them insecure and obsolete. As a result, there is an urgent need to update protocols and
infrastructure with post-quantum cryptographic (PQC) algorithms designed to resist
attacks from both quantum and classical adversaries. The cryptographic primitives
requiring replacement are discussed in {{RFC9958}}, and the NIST
PQC Standardization process has initially selected algorithms such as ML-KEM
{{FIPS203}}, ML-DSA {{FIPS204}}, and SLH-DSA {{FIPS205}} for usage in security
protocols.

To mitigate the risks posed by a CRQC, such as the potential compromise of encrypted
data and the forging of digital signatures, existing security protocols must be upgraded
to support PQC. These risks include "Harvest Now, Decrypt Later" (HNDL) attacks, where
adversaries capture encrypted traffic today with the intent to decrypt it once CRQCs
become available. TLS-based EAP methods are widely used for network access
authentication in enterprise and wireless environments. This document applies to all EAP
methods that use TLS as their underlying transport, including EAP-TLS {{RFC9190}},
EAP-TTLS {{RFC5281}}, PEAP, and TEAP {{RFC7170}}. To continue providing long-term
confidentiality and authentication guarantees, these methods must evolve to incorporate
post-quantum algorithms.

However, transitioning these protocols to support PQC introduces practical challenges.
{{?RFC9191}} highlights issues related to large certificates and certificate chains in
EAP-TLS, which can lead to session failures due to round-trip limitations. PQC
certificates and certificate chains tend to be significantly larger than their
traditional counterparts, further exacerbating these issues by increasing TLS handshake
sizes and the likelihood of session failures. To address these challenges, this document
specifies post-quantum key agreement and authentication requirements for TLS-based EAP
methods, together with a mechanism that reduces TLS handshake size for use in constrained
network environments.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document adopts terminology defined in {{RFC9794}}.
For the purposes of this document, it is useful to categorize cryptographic algorithms
into three distinct classes:

* Traditional Algorithm: An asymmetric cryptographic algorithm based on integer
  factorization, finite field discrete logarithms, or elliptic curve discrete
  logarithms. In the context of TLS, an example of a traditional key exchange algorithm
  is Elliptic Curve Diffie-Hellman (ECDH), which is almost exclusively used in its
  ephemeral mode, referred to as Elliptic Curve Diffie-Hellman Ephemeral (ECDHE).

* Post-Quantum Algorithm: An asymmetric cryptographic algorithm designed to be secure
  against attacks from both quantum and classical computers. An example of a
  post-quantum key exchange algorithm is the Module-Lattice Key Encapsulation Mechanism
  (ML-KEM).

* Hybrid Algorithm: We distinguish between key exchanges and signature algorithms:

  - Hybrid Key Exchange: A key exchange mechanism that combines two component algorithms
    - one traditional algorithm and one post-quantum algorithm. The resulting shared
    secret remains secure as long as at least one of the component key exchange
    algorithms remains unbroken.

  - PQ/T Hybrid Digital Signature: A multi-algorithm digital signature scheme composed
    of two or more component signature algorithms, where at least one is a post-quantum
    algorithm and at least one is a traditional algorithm.

Digital signature algorithms play a critical role in X.509 certificates, Certificate
Transparency Signed Certificate Timestamps, Online Certificate Status Protocol (OCSP)
statements, and any other mechanism that contributes signatures during a TLS handshake
or in the context of a secure communication establishment.

# Data Confidentiality in TLS-Based EAP Methods {#confident}

One of the primary threats to TLS-based EAP methods is the HNDL attack. In this
scenario, adversaries can passively capture EAP-TLS handshakes such as those transmitted
over the air in Wi-Fi networks and store them for future decryption once CRQCs become
available.

While EAP-TLS 1.3 {{RFC9190}} provides forward secrecy through ephemeral key exchange
and improves privacy by encrypting client identity and reducing exposure of session
metadata, these protections rely on the security of the underlying key exchange
algorithm. In the presence of a CRQC, traditional key exchange mechanisms (e.g., ECDHE)
would no longer provide long-term confidentiality. In such cases, an adversary could
mount an HNDL attack by passively recording EAP-TLS handshakes and decrypting the
captured traffic once quantum-capable cryptanalysis becomes feasible. This could
retroactively expose information that TLS 1.3 is otherwise designed to protect,
including:

   * The identity of the authenticated client.

   * Client credentials used in certificate-based authentication (e.g., usernames,
     device or organization identifiers).

In the case of EAP-TTLS, PEAP, and TEAP, HNDL attacks present an additional threat.
These methods typically carry legacy inner authentication protocols within the outer TLS
tunnel, such as MS-CHAPv2. If a CRQC is used to break the outer TLS tunnel, the exposed
inner authentication exchange could enable offline password attacks, potentially
allowing an adversary to recover user credentials.

To protect against HNDL attacks, TLS-based EAP deployments that require long-term
confidentiality MUST use TLS 1.3 {{RFC9846}} and MUST negotiate a post-quantum or PQ/T
hybrid key agreement group. Traditional key agreement groups alone do not provide
long-term confidentiality against an adversary equipped with a CRQC.

EAP peers and EAP servers MUST support at least one post-quantum or PQ/T hybrid key
agreement group registered in the "TLS Supported Groups" registry. PQ/T hybrid groups
combining ML-KEM with ECDHE are constructed as described in {{RFC9954}} and registered
in {{RFC10024}}.

This document does not mandate a specific group. The Recommended column of the "TLS
Supported Groups" registry, defined in {{RFC9847}}, records current IETF guidance on the
use of registered groups. At the time of writing, X25519MLKEM768 (code point 4588) is
the only PQ/T hybrid group marked Recommended.

PQ/T hybrid key agreement is generally preferred, because the shared secret remains
secure as long as either component algorithm remains unbroken {{RFC9794}}. Standalone
ML-KEM key agreement may be required for deployments subject to regulatory or compliance
mandates that require exclusive use of post-quantum cryptography. The choice is a
deployment decision and is out of scope for this document.

# Post-Quantum Authentication in TLS-Based EAP Methods {#eaptls-authentication}

Although a CRQC would primarily impact the confidentiality of recorded TLS sessions, it
could also pose risks to authentication mechanisms that rely on traditional public-key
algorithms with long-lived credentials. In particular, if quantum-capable cryptanalysis
were to become practical within the validity period of a certificate, an adversary could
recover the private key corresponding to a traditionally signed certificate and
subsequently impersonate the certificate holder in real time. The feasibility and impact
of such attacks depend on several factors, including certificate lifetimes and key
management practices.

TLS-based EAP deployments rely on X.509 certificates issued by CAs, and the transition
to PQ certificate authentication is constrained by the long lifecycle associated with
distributing, deploying, and validating new trust anchors. If CRQCs arrive sooner than
anticipated, deployed authentication systems may lack the agility to transition
credentials and trust anchors in a timely manner.

As a result, deployments that rely on long-lived certificates or that require resistance
to future quantum-capable adversaries face an increased risk of authentication
compromise. In such scenarios, an on-path attacker that is able to recover a server's
private key within the certificate validity period could impersonate access points (APs)
in real time, potentially deceiving users into revealing credentials or connecting to
rogue networks.

To mitigate these risks, TLS-based EAP deployments that require resistance to future
CRQCs MUST support at least one post-quantum or PQ/T hybrid signature scheme registered
in the "TLS SignatureScheme" registry for authentication. ML-DSA {{FIPS204}} is specified for use in X.509
certificates in {{RFC9881}} and for authentication in TLS 1.3 in {{I-D.ietf-tls-mldsa}}.

This document does not mandate a specific signature scheme or parameter set. The
Recommended column of the "TLS SignatureScheme" registry, defined in {{RFC9847}}, records
current IETF guidance on the use of registered schemes.

PQ/T hybrid authentication using composite signatures may be preferred by deployments
seeking defense in depth during the transition, so that authentication remains secure as
long as either component algorithm remains unbroken. Composite ML-DSA is specified for
X.509 in {{I-D.ietf-lamps-pq-composite-sigs}} and for TLS 1.3 in
{{I-D.reddy-tls-composite-mldsa}}. The choice between pure post-quantum and PQ/T hybrid
authentication is a deployment decision and is out of scope for this document.

The relatively large SLH-DSA {{FIPS205}} signatures may make SLH-DSA less suitable for
TLS-based EAP deployments, particularly where handshake size and fragmentation are
significant constraints. Signature sizes are given in {{RFC9958}}. The end-entity
certificate and the CertificateVerify each carry a signature in every handshake, and
neither can be avoided by the mechanism in {{ext-extn}}, which removes only intermediate
certificates.

A post-quantum or PQ/T hybrid end-entity certificate does not by itself provide
post-quantum authentication. Every signature in the path up to the trust anchor has to
use a post-quantum or PQ/T hybrid scheme, and since the trust anchor is provisioned out
of band, it has to be in place before the certificates that chain to it are issued.

The use of PQ or PQ/T hybrid certificates increases the size of individual certificates,
certificate chains, and signatures, resulting in significantly larger handshake messages.
These larger payloads can lead to packet fragmentation, retransmissions, and handshake
delays, issues that are particularly disruptive in constrained or lossy network
environments. {{ext-extn}} describes mitigations.

# EST Integration {#ext-extn}

The EAP client is expected to validate the certificate presented by the EAP server using
a trust anchor that is provisioned out-of-band prior to authentication (e.g., using
EST). The intermediate certificates are provided by the EAP server during the TLS
handshake. The EAP client relies solely on the pre-provisioned trust anchor to build and
validate the certificate chain. This model assumes a managed deployment environment with
explicitly configured trust relationships between the EAP client and EAP server.

Certificate compression {{RFC8879}} provides limited benefit for certificates containing
large high-entropy post-quantum public keys and signatures, and session resumption
requires a prior full handshake. Out-of-band provisioning of the intermediate chain can
avoid transmitting it during the first TLS authentication of a newly provisioned device.

To further reduce handshake overhead, particularly in deployments using large certificate
chains due to post-quantum (PQ) or composite certificates, this document specifies an
optimization that leverages the Enrollment over Secure Transport (EST) protocol
{{RFC7030}}, extended by {{RFC8295}}. Specifically, it allows intermediate certificates
to be retrieved in advance by using EST, thereby avoiding the need to transmit them
during each TLS handshake.

For EAP methods that use TLS as an outer tunnel (e.g., PEAP and TEAP), the EST
optimization described in this section applies to the certificates used in the outer TLS
tunnel. The EST pre-fetching of client intermediate certificates is relevant only when mutual TLS authentication is used. This is always the case for EAP-TLS, and optionally the case for EAP-TTLS and TEAP when client certificate authentication is used in the outer tunnel.

This section defines extensions to EST to support retrieval of the certificate chain used
by an EAP server and EAP clients. The first extension enables EAP clients to retrieve the
intermediate certificates required to build a certification path to the EAP server's
end-entity certificate.

A new path component is defined under the EST well-known URI:

~~~
GET /.well-known/est/eapservercertchain
~~~

The '/eapservercertchain' is intended for informational retrieval only and does not
require client authentication. It allows clients to retrieve the intermediate certificate
chain that the EAP server presents during TLS handshakes. This request is performed
using the HTTPS protocol. The EST server MUST support requests without requiring client
authentication. The EST server MUST provide the intermediate
certificates of the CAs that issue EAP server certificates.

The second extension enables EAP servers to retrieve the intermediate certificates
required to build a certification path to the EAP clients' end-entity certificates. Rather than relying on static
configuration, the EAP server can dynamically fetch the client's intermediate certificate
chain from a trusted EST server within the same administrative domain.

A new path component is defined under the EST well-known URI:

~~~
GET /.well-known/est/eapclientcertchain
~~~

The '/eapclientcertchain' is intended for informational retrieval only and does not
require client authentication. It allows the EAP server to retrieve the intermediate
certificate chain that the EAP clients present during TLS handshakes. This request is
performed using the HTTPS protocol. The EST server MUST support requests without
requiring client authentication. The EST server MUST provide the intermediate
certificates of the CAs that issue EAP client certificates.

Retrieved intermediate certificates are used for certification path construction together
with any certificates received in the TLS handshake. Where an EAP server has certificates
issued by more than one CA, the retrieved intermediate certificates cover all of them, and
the EAP client selects those needed to construct a path for the certificate presented in
the handshake. If no valid path can be constructed, authentication fails as for any other
certificate validation failure.

EAP clients and servers MUST authenticate the EST server using a trust anchor obtained
via a suitable bootstrapping mechanism before retrieving intermediate certificate chains
via HTTPS. Various bootstrapping mechanisms exist for establishing this trust, such as
BRSKI {{RFC8995}}, EST {{RFC7030}}, or out-of-band provisioning. The choice of
bootstrapping mechanism is a deployment decision and is out of scope for this document.
Certificate chains retrieved from an unauthenticated or untrusted EST server MUST NOT be
used for TLS chain validation.

EAP servers and clients are RECOMMENDED to cache retrieved certificate chains to reduce
latency and network overhead. However, they SHOULD implement mechanisms to detect changes
or expiration. These include periodic re-fetching, honoring HTTP cache control headers
(e.g., Cache-Control, ETag), and verifying the validity period of intermediate
certificates.

EAP clients MAY omit intermediate certificates from the TLS handshake only if they have
been explicitly configured by the administrator to do so. Such configuration is recommended only in deployments where both the EAP client and EAP server support this specification and have completed EST pre-fetching as part of provisioning. If no such
configuration is present, the EAP client MUST include the full certificate chain in the
TLS handshake. Similarly, an EAP server MAY omit intermediate certificates from the TLS
handshake only if it has been explicitly configured by the administrator to do so.
Administrators are advised to ensure that clients in the deployment have retrieved the
server's intermediate certificates via EST as part of their provisioning process before
enabling this configuration.

EST is one transport for retrieving intermediate certificates out of band. Device
management systems can deliver the same chains, and TEAP {{RFC7170}} could be extended to
provision them within the tunnel it establishes.

Note: A TLS extension could be used to explicitly signal support for intermediate
certificate omission between peers, avoiding the need for administrator configuration.
Such a mechanism is considered a possible future solution but is out of scope for this
document.

# Security Considerations

The security considerations outlined in {{I-D.ietf-uta-pqc-app}} and {{RFC9958}} must be
carefully evaluated and taken into account for all TLS-based EAP deployments.

# IANA Considerations

This document defines two new path components under the EST well-known URI
'/.well-known/est/', following the extension mechanism established by {{RFC8295}}:
'/eapservercertchain' and '/eapclientcertchain'. As these are sub-paths under the
already-registered '/.well-known/est/' prefix defined in {{RFC7030}}, no new IANA
registry entries are required.

# Acknowledgements
{:numbered="false"}

Thanks to John Mattsson, Hannes Tschofenig, Alan DeKok and Michael Richardson for the discussion and comments.
