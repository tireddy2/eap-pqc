---
title: "Post-Quantum Enhancements to EAP-TLS and EAP-TTLS"
abbrev: "PQC Enhancements to EAP-TLS/TTLS"
category: std

docname: draft-reddy-emu-pqc-eap-tls-latest
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
  RFC7030:
  RFC8295:

informative:

---

--- abstract


This document proposes enhancements to the Extensible Authentication Protocol with Transport Layer Security (EAP-TLS) and EAP Tunneled TLS (EAP-TTLS) to incorporate post-quantum cryptographic mechanisms. It also addresses challenges related to large certificate sizes and long certificate chains, as identified in {{?RFC9191}}, and provides recommendations for integrating PQC algorithms into EAP-TLS and EAP-TTLS deployments.

--- middle

# Introduction

The emergence of a Cryptographically Relevant Quantum Computer (CRQC) would break the mathematical assumptions that underpins widely deployed public-key algorithms, rendering them insecure and obsolete. As a result, there is an urgent need to update protocols and infrastructure with post-quantum cryptographic (PQC) algorithms designed to resist attacks from both quantum and classical adversaries. The cryptographic primitives requiring replacement are discussed in {{?I-D.ietf-pquip-pqc-engineers}}, and the NIST PQC Standardization process has initially selected algorithms such as ML-KEM, SLH-DSA, and ML-DSA for usage in security protocols.

To mitigate the risks posed by a CRQC, such as the potential compromise of encrypted data and the forging of digital signatures, existing security protocols must be upgraded to support PQC. These risks include "Harvest Now, Decrypt Later" (HNDL) attack, where adversaries capture encrypted traffic today with the intent to decrypt it once CRQCs become available. Protocols such as EAP-TLS and EAP-TTLS are widely used for network access authentication in Enterprise and Wireless environments. To continue providing long-term confidentiality and authentication guarantees, EAP-TLS and EAP-TTLS must evolve to incorporate post-quantum algorithms.

However, transitioning these protocols to support PQC introduces practical challenges. {{?RFC9191}} highlights issues related to large certificates and certificate chains in EAP-TLS, which can lead to session failures due to round-trip limitations. PQC certificates and certificate chains tend to be significantly larger than their traditional counterparts, further exacerbating these issues by increasing TLS handshake sizes and the likelihood of session failures. To address these challenges, this draft proposes mitigation strategies that enable the  use of PQC within EAP-TLS and EAP-TTLS, ensuring secure and efficient authentication even in constrained network environments.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document adopts terminology defined in {{?I-D.ietf-pquip-pqt-hybrid-terminology}}. For the purposes of this document, it is useful to categorize cryptographic algorithms into three distinct classes:

* Traditional Algorithm: An asymmetric cryptographic algorithm based on integer factorization, finite field discrete logarithms, or elliptic curve discrete logarithms. In the context of TLS, an example of a traditional key exchange algorithm is Elliptic Curve Diffie-Hellman (ECDH), which is almost exclusively used in its ephemeral mode, referred to as Elliptic Curve Diffie-Hellman Ephemeral (ECDHE).

* Post-Quantum Algorithm: An asymmetric cryptographic algorithm designed to be secure against attacks from both quantum and classical computers. An example of a post-quantum key exchange algorithm is the Module-Lattice Key Encapsulation Mechanism (ML-KEM).

* Hybrid Algorithm: We distinguish between key exchanges and signature algorithms:

  - Hybrid Key Exchange: A key exchange mechanism that combines two component algorithms - one traditional algorithm and one post-quantum algorithm. The resulting shared secret remains secure as long as at least one of the component key exchange algorithms remains unbroken.

  - PQ/T Hybrid Digital Signature: A multi-algorithm digital signature scheme composed of two or more component signature algorithms, where at least one is a post-quantum algorithm and at least one is a traditional algorithm.

Digital signature algorithms play a critical role in X.509 certificates, Certificate Transparency Signed Certificate Timestamps, Online Certificate Status Protocol (OCSP) statements, and any other mechanism that contributes signatures during a TLS handshake or in context of a secure communication establishment.

# Data Confidentiality in EAP-TLS {#confident}

One of the primary threats to EAP-TLS and EAP-TTLS is the HNDL attack. In this scenario, adversaries can passively capture EAP-TLS handshakes such as those transmitted over the air in Wi-Fi networks and store them for future decryption once CRQCs become available.

While EAP-TLS 1.3 {{RFC9190}} provides forward secrecy through ephemeral key exchange and improves privacy by encrypting client identity and reducing exposure of session metadata, these protections rely on the security of the underlying key exchange algorithm. In the presence of a CRQC, traditional key exchange mechanisms (e.g., ECDHE) would no longer provide long-term confidentiality. In such cases, an adversary could mount a HDNL attack by passively recording EAP-TLS handshakes and decrypting the captured traffic once quantum-capable cryptanalysis becomes feasible. This could retroactively expose information that TLS 1.3 is otherwise designed to protect, including:

   * The identity of the authenticated client.

   * Client credentials used in certificate-based authentication (e.g., usernames, device or organization identifiers).

To preserve the intended privacy guarantees of TLS 1.3 and to protect against HDNL attacks, EAP-TLS and EAP-TTLS deployments that require long-term confidentiality will need to adopt post-quantum key exchange mechanisms, as outlined in Section 4 of {{!I-D.ietf-uta-pqc-app}}.

These mechanisms ensure that even if handshake data is recorded today, it cannot be decrypted in the future, maintaining the confidentiality and privacy of the TLS session.

Furthermore, to support hybrid or PQC-only key exchange in bandwidth or latency-constrained EAP deployments, EAP clients and servers should apply the optimizations described in Section 4.1 of {{I-D.ietf-uta-pqc-app}} to minimize performance overhead.

# Post-Quantum Authentication in EAP-TLS {#eaptls-authentication}

Although a CRQC would primarily impact the confidentiality of recorded TLS sessions, it could also pose risks to authentication mechanisms that rely on traditional public-key algorithms with long-lived credentials. In particular, if quantum-capable cryptanalysis were to become practical within the validity period of a certificate, an adversary could recover the private key corresponding to a traditionally signed certificate and subsequently impersonate the certificate holder in real time. The feasibility and impact of such attacks depend on several factors, including certificate lifetimes and key management practices.

EAP-TLS and EAP-TTLS deployments rely on X.509 certificates issued by CAs, and the transition to PQ certificate authentication is constrained by the long lifecycle associated with distributing, deploying, and validating new trust anchors. If CRQCs arrive sooner than anticipated, deployed authentication systems may lack the agility to transition credentials and trust anchors in a timely manner.

As a result, deployments that rely on long-lived certificates or that require resistance to future quantum-capable adversaries face an increased risk of authentication compromise. In such scenarios, an on-path attacker that is able to recover a server’s private key within the certificate validity period could impersonate access points (APs) in real time, potentially deceiving users into revealing credentials or connecting to rogue networks.

To mitigate these risks, EAP-TLS and EAP-TTLS deployments will need to adopt, over time, either PQ or PQ/T hybrid certificate-based authentication, as described in {{Section 5 of I-D.ietf-uta-pqc-app}}.

The use of PQ or PQ/T hybrid certificates increases the size of individual certificates, certificate chains, and signatures, resulting in significantly larger handshake messages. These larger payloads can lead to packet fragmentation, retransmissions, and handshake delays, issues that are particularly disruptive in constrained or lossy network environments.

To address these impacts, EAP-TLS and EAP-TTLS deployments can apply certificate chain optimization techniques outlined in Section 6.1 of {{I-D.ietf-uta-pqc-app}} to reduce transmission overhead and improve handshake reliability.

# EST Integration {#ext-extn}

The EAP-client is expected to validate the certificate presented by the EAP-server using a trust anchor that is provisioned out-of-band prior to authentication (e.g., using EST). The Intermediate certificates are provided by the EAP server during the EAP-TLS handshake. The EAP client relies solely on the pre-provisioned trust anchor to build and validate the certificate chain. This model assumes a managed deployment environment with explicitly configured trust relationships between the EAP-client and EAP-server.

To further reduce handshake overhead, particularly in deployments using large certificate chains due to post-quantum (PQ) or composite certificates, this draft proposes an optimization that leverages the Enrollment over Secure Transport (EST) protocol {{RFC7030}}, extended by {{RFC8295}}. Specifically, it allows intermediate certificates to be retrieved in advance by using EST, thereby avoiding the need to transmit them during each EAP-TLS exchange.

This section defines extensions to EST to support retrieval of the certificate chain used by a EAP server and EAP clients. The first extension enables clients to obtain access to the complete set of published intermediate certificates of the EAP server.

A new path component is defined under the EST well-known URI:

~~~
GET /.well-known/est/eapservercertchain
~~~

The '/eapservercertchain' is intended for informational retrieval only and does not require client authentication. It allows clients to retrieve the intermediate certificate chain that the EAP server presents during TLS handshakes. This request is performed using the HTTPS protocol. The EST server MUST support requests without requiring client authentication. The certificate chain provided by the EST server MUST be the same certificate chain EAP server uses in a EAP-TLS or EAP-TTLS session.

The second extension enables EAP servers to obtain access to the complete set of published intermediate certificates of the EAP clients. Rather than relying on static configuration, the EAP server can dynamically fetch the client's intermediate certificate chain from a trusted EST server within the same administrative domain.

A new path component is defined under the EST well-known URI:

~~~
GET /.well-known/est/eapclientcertchain
~~~

The '/eapclientcertchain' is intended for informational retrieval only and does not require client authentication. It allows EAP server to retrieve the intermediate certificate chain that the EAP clients present during TLS handshakes. This request is performed using the HTTPS protocol. The EST server MUST support requests without requiring client authentication. The certificate chain provided by the EST server MUST be the same certificate chain EAP clients use in the EAP-TLS or EAP-TTLS session.

EAP servers and clients are RECOMMENDED to cache retrieved certificate chains to reduce latency and network overhead. However, they SHOULD implement mechanisms to detect changes or expiration. These include periodic re-fetching, honoring HTTP cache control headers (e.g., Cache-Control, ETag), and verifying the validity period of intermediate certificates.

As an alternative, a device MAY attempt to retrieve the certificate chain from the EST server (e.g., /eapservercertchain or /eapclientcertchain) only when certificate validation fails during an EAP-TLS or EAP-TTLS handshake. While this on-demand retrieval can serve as a fallback to recover from outdated intermediate certificate, it has the drawback of delaying authentication.

After retrieving intermediate certificates via EST, a EAP client that believes it has a complete set of intermediate certificates to authenticate the EAP server sends the tls_flags extension as defined in {{?I-D.kampanakis-tls-scas-latest}} with the 0xTBD1 flag set to 1 in its ClientHello message. Similarly, a EAP server that believes it has a complete set of intermediate certificates to authenticate the EAP client sends the same tls_flags extension with 0xTBD1 set to 1 in its CertificateRequest message. In both cases, only the end-entity certificates will be provided by the EAP client and server during the TLS handshake, relying on the recipient to possess or retrieve the necessary intermediate certificates for certificate chain validation.

# Security Considerations

The security considerations outlined in {{I-D.ietf-uta-pqc-app}} and {{?I-D.ietf-pquip-pqc-engineers}} must be carefully evaluated and taken into account for both EAP-TLS and EAP-TTLS deployments.

# IANA Considerations

This document does not request the creation of a new IANA registry nor the registration of the two URI path components defined in {{ext-extn}}.

# Acknowledgements
{:numbered="false"}

TBA.



