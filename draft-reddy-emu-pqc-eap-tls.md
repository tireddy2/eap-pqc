---
title: "Post-Quantum Cryptography Recommendations for EAP-TLS and EAP-TTLS"
abbrev: "PQC Recommendations for EAP-TLS/TTLS"
category: std

docname: draft-reddy-emu-pqc-eap-tls
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "EMU"
keyword:
 - PQC
 - PQ/T Hybrid
 - TLS
 - EAP

venue:
  group: "emu"
  type: "Working Group"
  mail: "emu@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/emu/"


stand_alone: yes
pi: [toc, sortrefs, symrefs, strict, comments, docmapping]

author:
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    city: Bangalore
    region: Karnataka
    country: India
    email: "kondtir@gmail.com"



normative:
  RFC9190:
  RFC7030:
  RFC8295:

informative:

---

--- abstract


This document proposes enhancements to the Extensible Authentication Protocol with Transport Layer Security (EAP-TLS) and EAP Tunneled TLS (EAP-TTLS) to incorporate post-quantum cryptographic mechanisms. It also addresses challenges related to large certificate sizes and long certificate chains, as identified in RFC9191, and provides recommendations for integrating PQC algorithms into EAP-TLS and EAP-TTLS deployments.

--- middle

# Introduction

The emergence of a Cryptographically Relevant Quantum Computer (CRQC) would break the mathematical assumptions that underpin widely deployed public-key algorithms, rendering them insecure and obsolete. As a result, there is an urgent need to update protocols and infrastructure with post-quantum cryptographic (PQC) algorithms designed to resist attacks from both quantum and classical adversaries. The cryptographic primitives requiring replacement are discussed in {{?I-D.ietf-pquip-pqc-engineers}}, and the NIST PQC Standardization process has initially selected algorithms such as ML-KEM, SLH-DSA, and ML-DSA for usage in security protocols.

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

While EAP-TLS 1.3 {{RFC9190}} was designed to provide strong forward secrecy and protect user privacy by encrypting client identity and reducing exposure of session metadata, HNDL attacks effectively nullify these protections. If the handshake is not quantum-resistant, a future CRQC could retroactively decrypt session traffic, revealing:

   * The identity of the authenticated client.

   * Client credentials used in certificate-based authentication (e.g., usernames, device or organization identifiers).

   * Session-specific metadata that may aid surveillance or profiling, such as cipher suites and TLS extensions supported. These elements can help to fingerprint devices, or correlate EAP-TLS sessions over time.

To preserve the intended privacy guarantees of TLS 1.3 and protect against HNDL, EAP-TLS and EAP-TTLS deployments MUST adopt post-quantum key exchange mechanisms, as outlined in Section 4 of {{?I-D.reddy-uta-pqc-app}}. These mechanisms ensure that even if handshake data is recorded today, it cannot be decrypted in the future, maintaining the confidentiality and privacy of the TLS session.

Furthermore, to support hybrid or PQC-only key exchange in bandwidth or latency-constrained EAP deployments, EAP clients and servers should apply the optimizations described in Section 4.1 of {{?I-D.reddy-uta-pqc-app}} to minimize performance overhead.

# Post-Quantum Authentication in EAP-TLS {#eaptls-authentication}

Although CRQCs could eventually decrypt recorded TLS sessions to recover derived keys and access confidential data, they cannot retroactively compromise client or server authentication if the attacker did not possess the corresponding private key at the time of the handshake. However, EAP-TLS and EAP-TTLS deployments rely on X.509 certificates issued by certificate authorities (CAs), and the transition to post-quantum (PQ) authentication is constrained by the long lifecycle involved to distribute, deploy, and validate new trust anchors. If CRQCs arrive sooner than anticipated, authentication systems may lack the agility to adapt in time.

This makes PQC authentication a critical requirement for EAP-TLS and EAP-TTLS deployments deployments. An on-path attacker equipped with a CRQC could compute a server’s private key before the certificate expires, enabling real-time impersonation of access points (APs). This could deceive users into revealing credentials or connecting to rogue networks, leading to privacy violations and potential client credential theft.

To mitigate these risks, EAP-TLS and EAP-TTLS deployments MUST adopt either pure PQ or PQ/T certificate-based authentication, as described in {{Section 5 of I-D.reddy-uta-pqc-app}}.

A composite certificate contains both a traditional public key algorithm (e.g., ECDSA) and a post-quantum algorithm (e.g., ML-DSA) within a single X.509 certificate. This design enables both algorithms to be used in parallel, the traditional component ensures compatibility with existing infrastructure, while the post-quantum component introduces resistance against future quantum attacks. This approach facilitates early adoption of PQC without requiring immediate disruption to established PKI deployments. 

The use of post-quantum or hybrid certificates increases the size of individual certificates, certificate chains, and signatures, resulting in significantly larger handshake messages. These larger payloads can lead to packet fragmentation, retransmissions, and handshake delays, issues that are particularly disruptive in constrained or lossy network environments.

To address these impacts, EAP-TLS deployments can apply certificate chain optimization techniques outlined in Section 6.1 of {{?I-D.reddy-uta-pqc-app}} to reduce transmission overhead and improve handshake reliability.

# EST Integration {#ext-extn}

To reduce handshake overhead further and suppress the transmission of intermediate certificates, especially important when certificate chains become large due to PQC or composite certificates, this draft leverages the Enrollment over Secure Transport (EST) protocol {{RFC7030}} as extended by EST extensions {{RFC8295}}.

This section defines extensions to EST to support retrieval of the certificate chain used by a EAP server and EAP clients. The first extension enables clients to obtain access to the complete set of published intermediate certificates of the EAP server. 

A new path component is defined under the EST well-known URI:

~~~
GET /.well-known/est/eapservercertchain
~~~

The '/eapservercertchain' is intended for informational retrieval only and does not require client authentication. It allows clients to retrieve the intermediate certificate chain that the EAP server presents during TLS handshakes. This request is performed using the HTTPS protocol. The EST server MUST support requests without requiring client authentication. The certificate chain provided by the EST server MUST be the same certificate chain EAP server uses in a EAP-TLS or EAP-TTLS session.

The second extension enables EAP servers to obtain access to the complete set of published intermediate certificates of the EAP clients.

A new path component is defined under the EST well-known URI:

~~~
GET /.well-known/est/eapclientcertchain
~~~

The '/eapclientcertchain' is intended for informational retrieval only and does not require client authentication. It allows EAP server to retrieve the intermediate certificate chain that the EAP clients present during TLS handshakes. This request is performed using the HTTPS protocol. The EST server MUST support requests without requiring client authentication. The certificate chain provided by the EST server MUST be the same certificate chain EAP clients use in the EAP-TLS or EAP-TTLS session.

After retrieving intermediate certificates via EST, a EAP client that believes it has a complete set of intermediate certificates to authenticate the EAP server sends the tls_flags extension as defined in {{!I-D.kampanakis-tls-scas}} with the 0xTBD1 flag set to 1 in its ClientHello message. Similarly, a EAP server that believes it has a complete set of intermediate certificates to authenticate the EAP client sends the same tls_flags extension with 0xTBD1 set to 1 in its CertificateRequest message.
   
# Security Considerations

The security considerations outlined in {{?I-D.reddy-uta-pqc-app}} and {{?I-D.ietf-pquip-pqc-engineers}} must be carefully evaluated and taken into account for both EAP-TLS and EAP-TTLS deployments.

# Acknowledgements
{:numbered="false"}

TBA.



