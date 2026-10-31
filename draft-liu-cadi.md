---
stand_alone: true
category: std
submissionType: IETF
ipr: trust200902
lang: en

title: Cryptographic Asset Discovery and Inventory
abbrev: CADI
docname: draft-liu-cadi-latest
obsoletes:
updates:
date:

# area:
# workgroup:

kw:
  - CADI
  - PQC
  - CBOM
  - YANG

author:
 -
  ins: C. P. Liu
  name: Chunchi Peter Liu
  organization: Huawei
  email: liuchunchi@huawei.com
  country: China

 -
  ins: Y.Fu
  name: Yufan Fu
  organization: Huawei
  email: fuyufan2@huawei.com
  country: China

normative:
  RFC7950:
  RFC9911:
  I-D.ietf-ivy-network-inventory-yang:
  CycloneDX:
    title: "ECMA-424: CycloneDX Bill of Materials Specification, 2nd Edition"
    author:
      org: Ecma International
    date: 2025
    target: https://ecma-international.org/publications-and-standards/standards/ecma-424/
  CDX-REGISTRY:
    title: CycloneDX Cryptography Registry
    author:
      org: OWASP CycloneDX
    target: https://cyclonedx.org/registry/cryptography/

informative:
  RFC6241:
  RFC7951:
  RFC8040:
  RFC8340:
  RFC8792:
  RFC9347:
  RFC9849:
  NIST-SP-800-57:
    title: "Recommendation for Key Management: Part 1 - General"
    author:
      org: National Institute of Standards and Technology
    seriesinfo:
      NIST SP: 800-57 Part 1 Rev. 5
    date: 2020-05
    target: https://doi.org/10.6028/NIST.SP.800-57pt1r5
  IANA-TLS:
    title: Transport Layer Security (TLS) Parameters
    author:
      org: IANA
    target: https://www.iana.org/assignments/tls-parameters
  G7:
    title: Advancing a Coordinated Roadmap for the Transition to Post-Quantum Cryptography in the Financial Sector
    date: 2026-01
    target: https://home.treasury.gov/system/files/136/G7-CEG-Quantum-Roadmap.pdf
  CISAACDI:
    title: Strategy for Migrating to Automated Post-Quantum Cryptography Discovery and Inventory Tools
    date: 2024-08
    target: https://www.cisa.gov/sites/default/files/2024-09/Strategy-for-Migrating-to-Automated-PQC-Discovery-and-Inventory-Tools.pdf
  NCCOEFAQ:
    title: Frequently Asked Questions about Post-Quantum Cryptography
    date: 2026-06
    target: https://pages.nist.gov/nccoe-migration-post-quantum-cryptography/FAQ/index.html
  OMBM2302:
    title: MEMORANDUM FOR THE HEADS OF EXECUTIVE DEPARTMENTS AND AGENCIES, Migrating to Post-Quantum Cryptography
    date: 2022-11
    target: https://www.whitehouse.gov/wp-content/uploads/2022/11/M-23-02-M-Memo-on-Migrating-to-Post-Quantum-Cryptography.pdf
  NIST-SP-1800-38B:
    title: Migration to Post-Quantum Cryptography Quantum Readiness- Cryptographic Discovery
    date: 2023-12
    target: https://www.nccoe.nist.gov/sites/default/files/2023-12/pqc-migration-nist-sp-1800-38b-preliminary-draft.pdf
  MERCURY:
    title: Mercury - network metadata capture and analysis.
    target: https://github.com/cisco/mercury/
  IBMCBOM:
    title: CBOM
    target: https://github.com/IBM/CBOM
  CDXGEN:
    title: CycloneDX Generator
    target: https://github.com/cdxgen/cdxgen

--- abstract

This document compiles existing Cryptographic Asset Discovery and Inventory (CADI) methods and analyzes potential gaps. It also defines a YANG data model that augments the base network inventory model with the cryptographic assets of each network element. The data model is aligned with the Cryptography Bill of Materials (CBOM) defined in CycloneDX.

--- middle

# Introduction {#intro}

Cryptographic Asset Discovery and Inventory (CADI) refers to a set of tools and guidelines that assist in identifying, collecting, normalizing, correlating, and continuously maintaining all cryptography-related assets, including configurations, dependencies, and usage contexts within a given organization, system, or product scope. It requires both technical tools and management approaches.

Many Post-Quantum transition roadmaps have highlighted the importance of identifying legacy cryptographic assets inside of an organization as a crucial preparatory part. For example:

* The G7 statement on "Advancing a Coordinated Roadmap for the Transition to Post-Quantum Cryptography in the Financial Sector" {{G7}} mentions there should exist six key PQ-migration activities:
    * Awareness and Preparation, **Discovery and Inventory**, Risk Assessment and Planning, Migration Execution, Migration Testing, Validation and Monitoring. The exact six stages are also published in NCCoE analysis on "Six Key Phases of the PQC Migration Journey" {{NCCOEFAQ}}.
* The United States Memorandum for the Heads of Executive Departments and Agencies on "Migrating to Post-Quantum Cryptography" {{OMBM2302}} requires:
    * All federal agencies to "**Submit cryptographic system inventory By May 4, 2023 and annually thereafter**". This important date is also mentioned in CISA directive "Strategy for Migrating to Automated Post-Quantum Cryptography Discovery and Inventory Tools"{{CISAACDI}}.

Although the exact PQ-migration roadmap in each different nations varies, the fact that cryptographic asset discovery and inventory is a key prerequisite for Chief Information Security Officers to asses a migration budget and draft a migration plan does not change.

This document analyzes existing CADI methods and potential gaps. It also defines a YANG data model that extends the network inventory with cryptographic assets.

# Conventions and Terminology {#term}

{::boilerplate bcp14-tagged}

The following terms are defined in {{RFC7950}} and are not redefined here: augment, data model, and data node. The terms "network element" and "component" are used as defined in {{I-D.ietf-ivy-network-inventory-yang}}. The meanings of the symbols in the YANG tree diagrams are defined in {{RFC8340}}.

This document also uses the following terms:

* CADI (Cryptographic Asset Discovery and Inventory) -- A set of tools and guidelines designed to automate the identification, collection, normalization, correlation, and lifecycle management of an organization's cryptographic assets.
* CBOM (Cryptography Bill of Materials) -- A machine-readable inventory of cryptographic assets, their properties, and their dependencies, defined as part of CycloneDX {{CycloneDX}}. In a CBOM, a cryptographic asset is a component of type "cryptographic-asset" described by "cryptoProperties".
* Cryptographic Asset -- A digital or physical element within a device or service-such as an algorithm, module, configuration, credential, secret, or communication protocol-that protects the confidentiality and integrity of secure data transmission or storage.
* Cryptographic System -- An active software or hardware implementation of one or more cryptographic algorithms that provide one or more of the following services: (1) creation and exchange of encryption keys; (2) encrypted connections; or (3) creation and validation of digital signatures. (according to {{OMBM2302}})
* Cryptographically Relevant Quantum Computers (CRQC) -- A fault-tolerant quantum system capable of solving the underlying hardness assumption problems of widely used public-key cryptography in polynomial time, such as RSA and ECC.

# Problem Statement and Challenges {#ps}

Apart from the milestones and requirements discussed above, there are also a few challenges to identify cryptographic assets:

* Legacy ICT systems were not engineered to treat cryptographic primitives as distinct, trackable assets. Instead, cryptographic implementations are deeply integrated, highly distributed, and hidden across heterogeneous layers of the technology stack.
* Taking inventory manually would take enormous amount of effort, yet there may still be statistical omissions that leave security exposures.
* Methods like Cryptographic Bill of Materials (CBOM) only take effect on new devices to self-announce its cryptographic assets, but it cannot help identify legacy devices and systems.
* Legacy devices and systems often behave like blackboxes, many does not allow installing additional programs or tools on them.

As a result, the ideal CADI tool should meet the following requirements:
*  **Automated** scanning and inventory taking
*  **Agnostic** to target object's system designs and architectures
*  **Integrate** the result to existing network management and operations systems for unified presentation

The requirements for automation, architectural agnosticism, and network management integration suggest a protocol-driven approach, potentially placing this within the scope of the IETF. However, this remains open for discussion.

# Discovery and Identification {#discovery}

There are many ways of doing discovering and identification. We catagorize them into two: active identification and passive identification.

   * **Active Identification**: The case where devices and services modifies its development and/or management process, actively self-announcing their cryptographic assets. Methods falls into this category includes:
     * CBOM Declaration
     * Static Code Scanning
     * Binary / Image Scanning
     * CI/CD Integration
   * **Passive Identification**: The case where devices and services will stay as-is and cannot self-announce their cryptographic assets. These devices will require additional external tools to assist the identification. Methods falls into this category includes:
     * Simulated Handshakes
     * Configuration Extraction
     * Traffic Pattern Analysis
     * Process Identification

## Active Identification

### Cryptographic Bill of Materials (CBOM) and Auxiliary Methods

CBOM-based identification is to empower cryptographic asset owners or vendors to explicitly declare cryptographic usage within their products, components, or services. Usually CBOM declaration is achieved through static code scanning and/or CI/CD integration, so we keep them in the same section.

Scanning Methods:
* **Static Code Scanning**:
    * SCA (Software Composition Analysis): Scan manifest files (like `package.json`, `requirements.txt`, or `pom.xml`) from the codebase to create CBOMS, usually for analyzing third-party libraries dependency.
    * SAST (Static Application Security Testing): Builds a Abstract Syntax Tree (AST) with parsed syntax and data flow of the target source code. It then uses AST to track call chain to end crypto library, track variables (data flow) from source to destination, do reference matching, etc.
        * SCA and SAST is used by IBM CBOMKit-Hyperion.
* **Binary / Image Scanning**:
    * Scan for algorithmic constants, signatures; Extract from OS trust stores and runtime configurations... The process details are omitted in this document due to complexity.
        * Binary / Image Scanning is used by IBM CBOMKit-Theia and CycloneDX Cdxgen.

Tools and Modelling:
* **IBM CBOMkit**, Static code level scanning (Hyperion) and Artifact level scanning (Theia) {{IBMCBOM}}: uses `implements` and `uses` relationship to create dependency diagram.
    * `implements`: Describes the list of protocols or algorithms this module implements.
    * `uses`: Describes what modules are used by this service.
    * For example, Application Nginx `uses` Library `libssl.so` that `uses` Protocol TLS v1.3/v1.2 that `uses` `libcrypto.so` that `implements` Algorithm MD5, SHA256, AES-128-GCM.
* **CycloneDX Cdxgen**, Artifact level scanning {{CDXGEN}}: uses `dependsOn` and `provides` relationship to create dependency diagram.
    * `dependsOn`: Describes other modules or services this module depends on.
    * `provides`: Describes all capabilities this module provides.
    * For example, Application Nginx `dependsOn` Library `libssl.so` , which `provides` TLS 1.2.

After scanning, the result will be recorded as a CBOM object.

CBOM data specification: A standardized cryptographic asset object includes cryptographic algorithms, digital certificates, protocols, private keys, public keys, cryptographic keys, ciphertext information, digital signatures, digests (the output values of hash functions), initialization vectors (input parameters for encryption algorithms), seeds, salts, shared secrets, authentication tags, passwords, credentials, and tokens. CycloneDX 1.7 {{CycloneDX}} groups them into four asset types: algorithm, certificate, protocol, and related cryptographic material. {{fig-cbom}} shows an example.

~~~
"components": [
 {
  "name": "google.com",
  "type": "cryptographic-asset",
  "bom-ref": "crypto/certificate/google.com@sha256:1e15e0fbd3ce9...",
  "cryptoProperties": {
    "assetType": "certificate",
    "certificateProperties": {
      "subjectName": "CN = www.google.com",
      "issuerName": "CN = GTS CA 1C3, O = Google Trust Services",
      "notValidBefore": "2016-11-21T08:00:00Z",
      "notValidAfter": "2017-11-22T07:59:59Z",
      "certificateFormat": "X.509",
      "certificateFileExtension": "crt",
      "relatedCryptographicAssets": [
        { "type": "algorithm", "ref": "crypto/algorithm/sha512-rsa" },
        { "type": "publicKey", "ref": "crypto/key/rsa-2048" }
      ]
    }
  }
 }
]
~~~
{: #fig-cbom title="Example CBOM (CycloneDX 1.7)"}

Gaps: Obviously, this does not help with identifying the mass legacy devices, components or services.

## Passive Identification

### Simulated Handshakes

Simulated handshake is a network-probing technique where a discovery tool acts as a client and intentionally initiates cryptographic protocol negotiations (e.g., TLS, SSH, IPsec) with enterprise network endpoints without completing the full data session. During the negotiation process, the discovery tool determines the specific cryptographic protocol versions, negotiation mechanisms, and ciphersuites supported by the endpoint. This is also known as *Active Probing*.

Steps of simulated handshake include:
1. **Port Scan:** The tool scans for typical ports for specific secure communication protocols (443 for HTTPS, 22 for SSH, etc)
2. **Probing:** The tool generates a RFC-compliant connection initialization packet, including all currently standardized ciphersuites, and examine the response.
3. **Response Extraction:** The target object respond with supported ciphersuites, following its own internal priority rules.
4. **Parsing:** The tool intercepts this raw response and decodes it as cryptographic assets.

This works for most secure protocols that includes negotiation:
* IPSEC
* TLS
* SSH

Pros and Cons/Gaps:
* Strengths:
    * This kind of method does not require complex engineering refactoring. Since it is agnostic to the design of the target, it works best for the case where the manager operates vast heterogeneous devices from different vendors, versions and locations, hence telecommunications case.
    * This probing is one-time, thus it will not create unhandleable amount of probing packets.
* The limitation of this method is that
    * Some masking mechanism will stop the probing (firewalls, load balancers, port-concealing protocols like Single Packet Authorization that enforce a default drop for unexpected packets), leaving a hidden security posture inside of the system.
    * Network management cannot know local application components that rests internally in the device and are not exposed over a socket.

Tools:
* Cisco Mercury project {{MERCURY}} provides an open source packet capture and analysis tool. It can read network packets, identify metadata of interest, and write out the metadata (including cryptographic-related) in JSON format.

### Traffic Pattern Analysis

Unlike simulated handshakes that actively query an endpoint, Traffic Pattern Analysis listens passively to live network traffic at strategic aggregation points. It infers cryptographic asset usage from protocol metadata, statistical characteristics, and behaviors without decrypting the underlying data payload.

Methods of Traffic Pattern Analysis includes:
* Listen to unencrypted negotiations like `clientHello`/`serverHello`, Server Name Indication (SNI), response of an unencrypted server digital certificate request, etc.
* Listen to handshake fingerprints and compare them to existing patterns, as protocol implementations and behaviors are often rigid.
* Listen to packet length and arrival time, do statistical pattern recognition.

Pros and Cons/Gaps:
* Strengths:
    * Similar to Simulated Handshakes, this kind of method is not intrusive and is agnostic to the design of the target.
* The limitation of this method is that
    * Negotiation information like `clientHello` {{RFC9849}} and SNI are tending to be encrypted.
    * Protocols tend to have constant size packets {{RFC9347}}.
    * Alternative routing: network requests/responses could exit through a different routing path.

### Process Identification

Process Identification requires Endpoint Detection and Response (EDR) installed to have cryptographic visibility inside of a system.

Methods of Process Identification includes:
* Kernel-Level System Call Auditing by deploying eBPF programs into the kernel space. These filters can capture kernel events to determine exactly which process (down to PID) have initiated an encrypted network communication.
* Shared Library and Binary Hooking, by monitoring process load table to see if cryptographic dynamic libraries are loaded into the memory space.
* Live Memory Scanning, by periodically scanning volatile memory for indication of creation of keys or signatures (as high-entropy memory segments), and then compare them with known patterns.

Pros and Cons/Gaps:
* Strengths:
    * Have deeper visibility for cryptographic assets within devices.
* The limitation of this method is that
    * Requires loading new software (at least EDR/RDR) to legacy devices.
    * Potential blind spots due to lower pattern coverage.

### Configuration Extraction

The Configuration Extraction is a bit similar to the source code scanning. This section focuses more on the runtime configurations. In different use cases, the configuration extraction methods differs significantly. This section is to be extended.

Methods of Configuration Extraction includes:
* In network device management, NETCONF and YANG provide a standardized, model-driven management interface that enables programmable configuration and state validation. The network managing platform can define a YANG model and extract configurations from a device.
* In cloud-native development, general `.yaml` configuration files, `.env` environment variables files, Kubernetes secret manifest, etc, can be extracted by administrative scripts or through management interfaces (e.g., AWS CloudControl, Kubernetes API).

# Inventory {#inventory}

## Overview

The discovery methods in {{discovery}} produce information about cryptographic assets in different forms, e.g., CBOMs delivered by vendors or handshake results collected by scanners. To support migration planning, this information needs to be correlated with the network elements and components on which the assets are deployed, and exposed through a standard management interface.

This section defines the "ietf-network-inventory-crypto" YANG module. It augments the base network inventory model {{I-D.ietf-ivy-network-inventory-yang}} with a read-only cryptographic inventory of each network element. Like the base model, it is a network model: a network controller uses it to report the cryptographic assets of all network elements under its control, e.g., to an Operations Support System (OSS) or to a higher-level controller, via NETCONF {{RFC6241}} or RESTCONF {{RFC8040}}.

The module complements CycloneDX CBOM {{CycloneDX}} rather than replacing it, as summarized in the table below. Asset types, properties, and relationships are aligned with CycloneDX 1.7, so that a vendor CBOM can be imported into a controller and controller data can be exported as a CBOM. This enables network-wide post-quantum cryptography (PQC) readiness assessment.

|                                                              | CycloneDX CBOM                  | This Module                                  |
| :----------------------------------------------------------- | :------------------------------ | :------------------------------------------- |
| Form                                                         | Document (JSON, XML)            | Interface (NETCONF, RESTCONF)                |
| Scope                                                        | One product or software version | All network elements managed by a controller |
| Producer                                                     | Vendor or scanning tool         | Network controller                           |
| Typical use                                                  | Supply chain, audit             | Operations, PQC readiness assessment         |
| Linked to                                                    | Software components             | Network elements                             |
| {: #tab-cbom-yang title="CBOM Documents and the Inventory Data Model"} |                                 |                                              |

## Model Structure

The module adds a "crypto-inventory" container to each network element, alongside the "components" list of the base model. An asset can thus refer to the components that implement it, e.g., a Hardware Security Module (HSM), or a cryptographic library ({{relationships}}). All data nodes of the container are read-only, as the augmented "network-inventory" container is "config false". It contains two lists:

"cbom-ref":
: References to CBOM documents that describe the network element or one of its components. Each entry contains the BOM serial number, version, and specification version, retrieval URIs, a document hash for integrity verification, and the lifecycle phases in which the data was captured.

"crypto-asset":
: A structured list of the cryptographic assets of the network element, as described below.

### Cryptographic Assets

Each asset is identified by an "asset-id" that is unique within the network element. The common attributes are:

* "asset-type": one of the four CycloneDX asset types, i.e., algorithm, certificate, protocol, or related cryptographic material (e.g., keys, passwords, and tokens).
* "uuid", "name", "alias", and "description": reused from the base model.
* "oid": the object identifier (OID) of the asset, if any.
* "bom-link": a CycloneDX BOM-Link to the element describing the asset in a CBOM listed in "cbom-ref", for traceability.
* "usage": the purposes for which the asset is used, i.e., management plane, control plane, data plane, device identity, secure boot, or storage protection. CycloneDX has no equivalent; this network-specific context is needed to prioritize migration.
* "discovery": the discovery methods ({{discovery}}), the confidence in the result, and the time at which the asset was last observed.

Type-specific properties are reported in one of four containers, which are present only for the corresponding asset type:

* "algorithm-properties": primitive, algorithm family, parameter set, elliptic curve, mode, execution environment (e.g., software or hardware), certification level, classical security level, and NIST quantum security level, where the value 0 indicates a quantum-vulnerable algorithm.
* "certificate-properties": serial number, subject, issuer, validity period, format, fingerprint, and state (e.g., active or revoked).
* "related-material-properties": material type (e.g., private key), key state as defined in {{NIST-SP-800-57}}, size, format, creation and expiration time, and protection mechanism (e.g., HSM or Trusted Platform Module (TPM)).
* "protocol-properties": protocol type, version, and cipher suites.

Only the CycloneDX properties needed for inventory and PQC readiness assessment are modeled. Further details remain available in the CBOM referenced by "cbom-ref" and can be added by augmentation.

### Naming {#naming}

CycloneDX 1.7 introduces the Cryptography Registry {{CDX-REGISTRY}}. It defines algorithm families together with naming patterns, and elliptic curves in namespaces of the defining organizations. The Registry evolves independently of the CycloneDX specification. This module uses it as follows:

* "algorithm-family" SHOULD contain a family name from the Registry, e.g., "AES", "ML-KEM", or "SM4".
* The "name" of an algorithm SHOULD follow the naming pattern of its family, e.g., "AES-128-GCM" or "ML-KEM-768".
* "elliptic-curve" SHOULD contain a Registry curve name in the form "\<namespace>/\<curve>", e.g., "nist/P-256". A curve can have aliases in other namespaces, e.g., "secg/secp256r1" and "x962/prime256v1" for "nist/P-256"; consumers need to treat aliases as the same curve.
* A hybrid scheme is reported as an algorithm whose "primitive" is "combiner" and whose "name" is the registered name of the hybrid, e.g., the TLS group name "X25519MLKEM768" {{IANA-TLS}}. No "algorithm-family" is reported for it; it depends on its component algorithms ({{relationships}}).

### Relationships {#relationships}

Two relationship types are defined, matching the two dependency types of CycloneDX:

"provided-by":
: From an asset to the components that implement or provide it, e.g., from a TLS implementation to a cryptographic library or a main processing unit. CycloneDX expresses this as "dependencies\[\].provides", in the opposite direction.

"depends-on":
: From an asset to other assets that it uses or is bound to. CycloneDX expresses this as "dependencies\[\].dependsOn" or "relatedCryptographicAssets". The meaning of the relationship is given by the type of the referenced asset. For example, a certificate depends on an algorithm (its signature algorithm) and on a public key (its subject public key), a protocol depends on a certificate (its credential), and a combiner depends on its component algorithms.

A relationship is recorded once, at the asset that uses or holds the other asset; reverse relationships are not recorded. The resulting graph is expected to be acyclic.

Two references are modeled as properties rather than as relationships:

* "algorithm-ref" in "protocol-properties/cipher-suite" keeps the grouping of algorithms per cipher suite, as in CycloneDX "cipherSuites\[\].algorithms".
* "algorithm-ref" in "related-material-properties/secured-by" identifies the algorithm protecting a key (e.g., a key-wrap algorithm). It is distinct from the algorithm with which the key is used, which is expressed by "depends-on".

{{fig-rel}} shows the relationships of the example in {{example}}. Unlabeled arrows denote "depends-on", and the labeled arrow denotes "provided-by". Property references, such as "algorithm-ref" in a cipher suite, are not shown.

~~~
  +------------+  provided-by   +-------------+
  |   tls-1    +--------------->|    mpu-1    |
  | (protocol) |                | (component) |
  +--+------+--+                +-------------+
     |      |
     |      +----------------+
     v                       v
  +------------+     +----------------+
  |  dev-cert  |     | x25519mlkem768 |
  +-----+------+     +---+--------+---+
        |                |        |
        v                v        v
  +-------------+   +--------+  +------------+
  | ecdsa-p-256 |   | x25519 |  | ml-kem-768 |
  +-------------+   +--------+  +------------+
~~~
{: #fig-rel title="Example of Relationships"}

## Tree Diagram {#tree}

{{fig-tree}} shows the tree diagram of the "ietf-network-inventory-crypto" module.

~~~
module: ietf-network-inventory-crypto

  augment /nwi:network-inventory/nwi:network-elements
            /nwi:network-element:
    +--ro crypto-inventory!
       +--ro cbom-ref* [serial-number]
       |  +--ro serial-number      cbom-serial-number
       |  +--ro version?           uint32
       |  +--ro spec-version?      string
       |  +--ro uri*               inet:uri
       |  +--ro document-hash
       |  |  +--ro algorithm?   string
       |  |  +--ro value?       string
       |  +--ro lifecycle-phase*   string
       |  +--ro component-ref?     leafref
       +--ro crypto-asset* [asset-id]
          +--ro asset-id       string
          +--ro asset-type     identityref
          +--ro uuid?          yang:uuid
          +--ro name?          string
          +--ro alias?         string
          +--ro description?   string
          +--ro oid?           yang:object-identifier
          +--ro bom-link?      bom-link
          +--ro usage*         identityref
          +--ro provided-by*   leafref
          +--ro depends-on*    leafref
          +--ro discovery
          |  +--ro method*            identityref
          |  +--ro confidence?        decimal64
          |  +--ro last-discovered?   yang:date-and-time
          +--ro algorithm-properties
          |  +--ro primitive?                     string
          |  +--ro algorithm-family?              algorithm-family
          |  +--ro parameter-set-identifier?      string
          |  +--ro elliptic-curve?                elliptic-curve
          |  +--ro mode?                          string
          |  +--ro execution-environment?         string
          |  +--ro certification-level*           string
          |  +--ro classical-security-level?      uint16
          |  +--ro nist-quantum-security-level?   uint8
          +--ro certificate-properties
          |  +--ro serial-number?        string
          |  +--ro subject-name?         string
          |  +--ro issuer-name?          string
          |  +--ro not-valid-before?     yang:date-and-time
          |  +--ro not-valid-after?      yang:date-and-time
          |  +--ro certificate-format?   string
          |  +--ro fingerprint
          |  |  +--ro algorithm?   string
          |  |  +--ro value?       string
          |  +--ro state?                enumeration
          +--ro related-material-properties
          |  +--ro material-type?     string
          |  +--ro state?             enumeration
          |  +--ro size?              uint32
          |  +--ro format?            string
          |  +--ro creation-date?     yang:date-and-time
          |  +--ro expiration-date?   yang:date-and-time
          |  +--ro secured-by
          |     +--ro mechanism?       string
          |     +--ro algorithm-ref?   leafref
          +--ro protocol-properties
             +--ro protocol-type?   string
             +--ro version?         string
             +--ro cipher-suite* [name]
                +--ro name             string
                +--ro identifier*      string
                +--ro algorithm-ref*   leafref
~~~
{: #fig-tree title="Tree Diagram of ietf-network-inventory-crypto"}

## YANG Module {#module}

This module imports types from {{RFC9911}} and {{I-D.ietf-ivy-network-inventory-yang}}. It references {{CycloneDX}}, {{CDX-REGISTRY}}, and {{NIST-SP-800-57}}.

~~~ yang
<CODE BEGINS> file "ietf-network-inventory-crypto@2026-10-30.yang"
module ietf-network-inventory-crypto {
  yang-version 1.1;
  namespace
    "urn:ietf:params:xml:ns:yang:ietf-network-inventory-crypto";
  prefix nwic;

  import ietf-yang-types {
    prefix yang;
    reference
      "RFC 9911: Common YANG Data Types";
  }
  import ietf-inet-types {
    prefix inet;
    reference
      "RFC 9911: Common YANG Data Types";
  }
  import ietf-network-inventory {
    prefix nwi;
    reference
      "RFC YYYY: A Base YANG Data Model for Network Inventory";
  }

  organization
    "IETF Network Inventory YANG (ivy) Working Group";
  contact
    "WG Web:   <https://datatracker.ietf.org/wg/ivy/>
     WG List:  <mailto:ivy@ietf.org>

     Editor:   Chunchi Peter Liu
               <mailto:liuchunchi@huawei.com>";
     Editor:   Yufan Fu
               <mailto:fuyufan2@huawei.com>";
  description
    "This module augments the base network inventory model with
     a read-only inventory of the cryptographic assets of each
     network element.  Asset types, properties, and relationships
     are aligned with the Cryptography Bill of Materials (CBOM)
     defined in CycloneDX 1.7 (ECMA-424).

     The key words 'MUST', 'MUST NOT', 'REQUIRED', 'SHALL',
     'SHALL NOT', 'SHOULD', 'SHOULD NOT', 'RECOMMENDED',
     'NOT RECOMMENDED', 'MAY', and 'OPTIONAL' in this document
     are to be interpreted as described in BCP 14 (RFC 2119)
     (RFC 8174) when, and only when, they appear in all
     capitals, as shown here.

     Copyright (c) 2026 IETF Trust and the persons identified
     as authors of the code.  All rights reserved.

     Redistribution and use in source and binary forms, with or
     without modification, is permitted pursuant to, and subject
     to the license terms contained in, the Revised BSD License
     set forth in Section 4.c of the IETF Trust's Legal Provisions
     Relating to IETF Documents
     (https://trustee.ietf.org/license-info).

     This version of this YANG module is part of RFC XXXX; see
     the RFC itself for full legal notices.";

  revision 2026-10-30 {
    description
      "Initial revision.";
    reference
      "RFC XXXX: Cryptographic Asset Discovery and Inventory";
  }

  /*
   * Identities
   */

  identity crypto-asset-type {
    description
      "Base identity for cryptographic asset types.";
    reference
      "ECMA-424: CycloneDX Bill of Materials Specification,
       cryptoProperties.assetType";
  }

  identity algorithm {
    base crypto-asset-type;
    description
      "A cryptographic algorithm, e.g., AES-128-GCM or
       ML-KEM-768.";
  }

  identity certificate {
    base crypto-asset-type;
    description
      "A certificate, e.g., an X.509 device certificate.";
  }

  identity protocol {
    base crypto-asset-type;
    description
      "A cryptographic protocol, e.g., TLS, SSH, or IKEv2.";
  }

  identity related-crypto-material {
    base crypto-asset-type;
    description
      "Material related to algorithms, certificates, or
       protocols, e.g., a key, a password, or a token.";
  }

  identity crypto-usage {
    description
      "Base identity for the purposes for which a cryptographic
       asset is used on a network element.";
  }

  identity management-plane {
    base crypto-usage;
    description
      "Protection of management traffic, e.g., NETCONF over SSH.";
  }

  identity control-plane {
    base crypto-usage;
    description
      "Protection of control plane protocols, e.g., routing
       protocol authentication.";
  }

  identity data-plane {
    base crypto-usage;
    description
      "Protection of user traffic, e.g., IPsec or MACsec.";
  }

  identity device-identity {
    base crypto-usage;
    description
      "Identification and authentication of the device, e.g.,
       using an IDevID certificate.";
  }

  identity secure-boot {
    base crypto-usage;
    description
      "Verification of boot loaders and images during start-up.";
  }

  identity software-integrity {
    base crypto-usage;
    description
      "Verification of software images and patches.";
  }

  identity storage-protection {
    base crypto-usage;
    description
      "Protection of data at rest, e.g., configuration files or
       key stores.";
  }

  identity discovery-method {
    description
      "Base identity for the methods by which a cryptographic
       asset is discovered.";
  }

  identity cbom-declaration {
    base discovery-method;
    description
      "The asset is declared in a CBOM, e.g., one produced by
       static code scanning, binary scanning, or CI/CD
       integration.";
    reference
      "RFC XXXX: Cryptographic Asset Discovery and Inventory,
       Section 4.1";
  }

  identity simulated-handshake {
    base discovery-method;
    description
      "The asset is discovered by initiating protocol
       negotiations with the network element.";
    reference
      "RFC XXXX: Cryptographic Asset Discovery and Inventory,
       Section 4.2.1";
  }

  identity traffic-analysis {
    base discovery-method;
    description
      "The asset is inferred from passively observed traffic.";
    reference
      "RFC XXXX: Cryptographic Asset Discovery and Inventory,
       Section 4.2.2";
  }

  identity process-identification {
    base discovery-method;
    description
      "The asset is identified by monitoring processes on the
       network element.";
    reference
      "RFC XXXX: Cryptographic Asset Discovery and Inventory,
       Section 4.2.3";
  }

  identity configuration-extraction {
    base discovery-method;
    description
      "The asset is identified from the configuration of the
       network element.";
    reference
      "RFC XXXX: Cryptographic Asset Discovery and Inventory,
       Section 4.2.4";
  }

  /*
   * Typedefs
   */

  typedef algorithm-family {
    type string;
    description
      "An algorithm family name registered in the CycloneDX
       Cryptography Registry, e.g., 'AES', 'ML-KEM', or 'SM4'.
       The value SHOULD match the Registry entry exactly.";
    reference
      "CycloneDX Cryptography Registry";
  }

  typedef elliptic-curve {
    type string {
      pattern '[a-z0-9]+/.+';
    }
    description
      "An elliptic curve name registered in the CycloneDX
       Cryptography Registry, in the form '<namespace>/<curve>',
       e.g., 'nist/P-256' or 'oscaa/SM2'.";
    reference
      "CycloneDX Cryptography Registry";
  }

  typedef cbom-serial-number {
    type string {
      pattern 'urn:uuid:[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-'
            + '[0-9a-f]{4}-[0-9a-f]{12}';
    }
    description
      "The serial number of a CycloneDX BOM, i.e., a UUID URN.";
    reference
      "ECMA-424: CycloneDX Bill of Materials Specification,
       serialNumber";
  }

  typedef bom-link {
    type string {
      pattern 'urn:cdx:[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-'
            + '[0-9a-f]{4}-[0-9a-f]{12}/[1-9][0-9]*#.+';
    }
    description
      "A CycloneDX BOM-Link to an element of a BOM, in the form
       'urn:cdx:<serial-number>/<version>#<bom-ref>'.";
    reference
      "ECMA-424: CycloneDX Bill of Materials Specification,
       BOM-Link";
  }

  /*
   * Groupings
   */

  grouping hash {
    description
      "A hash value and the algorithm that produced it.";
    leaf algorithm {
      type string;
      description
        "The hash algorithm, using the CycloneDX hash algorithm
         names, e.g., 'SHA-256' or 'SHA3-384'.";
    }
    leaf value {
      type string {
        pattern '[0-9a-fA-F]+';
      }
      description
        "The hex-encoded hash value.";
    }
  }

  /*
   * Augmentation
   */

  augment "/nwi:network-inventory/nwi:network-elements"
        + "/nwi:network-element" {
    description
      "Adds the cryptographic inventory to network elements.";
    container crypto-inventory {
      presence
        "Indicates that the server reports the cryptographic
         inventory of this network element.";
      description
        "The cryptographic inventory of the network element.";
      list cbom-ref {
        key "serial-number";
        description
          "References to CBOM documents that describe the
           cryptographic assets of the network element or of one
           of its components.";
        leaf serial-number {
          type cbom-serial-number;
          description
            "The serial number of the CBOM.";
        }
        leaf version {
          type uint32 {
            range "1..max";
          }
          description
            "The version of the CBOM.";
        }
        leaf spec-version {
          type string;
          description
            "The CycloneDX specification version to which the CBOM
             conforms, e.g., '1.7'.";
        }
        leaf-list uri {
          type inet:uri;
          description
            "Locations from which the CBOM can be retrieved.";
        }
        container document-hash {
          description
            "The hash of the CBOM document, for integrity
             verification.";
          uses hash;
        }
        leaf-list lifecycle-phase {
          type string;
          description
            "The lifecycle phases in which the CBOM data was
             captured, using the CycloneDX phase names, e.g.,
             'build' or 'operations'.";
          reference
            "ECMA-424: CycloneDX Bill of Materials Specification,
             metadata.lifecycles";
        }
        leaf component-ref {
          type leafref {
            path "../../../nwi:components/nwi:component"
               + "/nwi:component-id";
          }
          description
            "The component described by the CBOM.  If not present,
             the CBOM describes the whole network element.";
        }
      }
      list crypto-asset {
        key "asset-id";
        description
          "The cryptographic assets of the network element.";
        leaf asset-id {
          type string;
          description
            "An identifier of the asset, unique within the network
             element.";
        }
        leaf asset-type {
          type identityref {
            base crypto-asset-type;
          }
          mandatory true;
          description
            "The type of the asset.";
        }
        uses nwi:basic-common-entity-attributes {
          refine "name" {
            description
              "The name of the asset.  The name of an algorithm
               SHOULD follow the naming pattern of its family in
               the CycloneDX Cryptography Registry, e.g.,
               'AES-128-GCM'.";
          }
        }
        leaf oid {
          type yang:object-identifier;
          description
            "The object identifier of the asset.";
        }
        leaf bom-link {
          type bom-link;
          description
            "A reference to the element describing this asset in a
             CBOM listed in 'cbom-ref'.";
        }
        leaf-list usage {
          type identityref {
            base crypto-usage;
          }
          description
            "The purposes for which the asset is used.";
        }
        leaf-list provided-by {
          type leafref {
            path "../../../nwi:components/nwi:component"
               + "/nwi:component-id";
          }
          description
            "The hardware or software components that implement or
             provide the asset.";
        }
        leaf-list depends-on {
          type leafref {
            path "../../crypto-asset/asset-id";
          }
          description
            "Other assets that this asset uses or is bound to.  The
             meaning of the relationship is given by the type of the
             referenced asset.";
        }
        container discovery {
          description
            "Information on how the asset was discovered.";
          leaf-list method {
            type identityref {
              base discovery-method;
            }
            description
              "The methods by which the asset was discovered.";
          }
          leaf confidence {
            type decimal64 {
              fraction-digits 2;
              range "0..1";
            }
            description
              "The overall confidence in the discovery result, where
               1 means 100% confidence.";
          }
          leaf last-discovered {
            type yang:date-and-time;
            description
              "The time at which the asset was last observed.";
          }
        }
        container algorithm-properties {
          when "derived-from-or-self(../asset-type, "
             + "'nwic:algorithm')";
          description
            "Properties of an algorithm.";
          leaf primitive {
            type string;
            description
              "The cryptographic primitive, using the CycloneDX
               values, e.g., 'block-cipher', 'ae', 'signature',
               'key-agree', 'kem', or 'combiner'.";
          }
          leaf algorithm-family {
            type algorithm-family;
            description
              "The algorithm family.  Not present for combiners.";
          }
          leaf parameter-set-identifier {
            type string;
            description
              "The parameter set, e.g., '128' for AES-128 or '768'
               for ML-KEM-768.";
          }
          leaf elliptic-curve {
            type elliptic-curve;
            description
              "The elliptic curve, if any.";
          }
          leaf mode {
            type string;
            description
              "The mode of operation of a block cipher, using the
               CycloneDX values, e.g., 'gcm' or 'cbc'.";
          }
          leaf execution-environment {
            type string;
            description
              "The environment in which the algorithm is executed,
               using the CycloneDX values, e.g.,
               'software-plain-ram' or 'hardware'.";
          }
          leaf-list certification-level {
            type string;
            description
              "The certifications of the implementation, using the
               CycloneDX values, e.g., 'fips140-3-l1'.";
          }
          leaf classical-security-level {
            type uint16;
            units "bits";
            description
              "The classical security level of the algorithm.";
          }
          leaf nist-quantum-security-level {
            type uint8 {
              range "0..5";
            }
            description
              "The NIST post-quantum security strength category.
               The value 0 indicates that none of the categories is
               met, i.e., the algorithm is quantum-vulnerable.";
            reference
              "ECMA-424: CycloneDX Bill of Materials Specification,
               algorithmProperties.nistQuantumSecurityLevel";
          }
        }
        container certificate-properties {
          when "derived-from-or-self(../asset-type, "
             + "'nwic:certificate')";
          description
            "Properties of a certificate.";
          leaf serial-number {
            type string;
            description
              "The serial number assigned by the issuer.";
          }
          leaf subject-name {
            type string;
            description
              "The subject name.";
          }
          leaf issuer-name {
            type string;
            description
              "The issuer name.";
          }
          leaf not-valid-before {
            type yang:date-and-time;
            description
              "The start of the validity period.";
          }
          leaf not-valid-after {
            type yang:date-and-time;
            description
              "The end of the validity period.";
          }
          leaf certificate-format {
            type string;
            description
              "The certificate format, e.g., 'X.509'.";
          }
          container fingerprint {
            description
              "The fingerprint of the certificate.";
            uses hash;
          }
          leaf state {
            type enumeration {
              enum pre-activation {
                description
                  "Issued but not yet authorized for use.";
              }
              enum active {
                description
                  "Authorized for use.";
              }
              enum suspended {
                description
                  "Use temporarily suspended.";
              }
              enum deactivated {
                description
                  "Not to be used to apply cryptographic
                   protection.";
              }
              enum revoked {
                description
                  "Revoked by the issuer before its expiration.";
              }
              enum destroyed {
                description
                  "Destroyed.";
              }
            }
            description
              "The current state of the certificate.";
            reference
              "ECMA-424: CycloneDX Bill of Materials Specification,
               certificateState";
          }
        }
        container related-material-properties {
          when "derived-from-or-self(../asset-type, "
             + "'nwic:related-crypto-material')";
          description
            "Properties of related cryptographic material.";
          leaf material-type {
            type string;
            description
              "The type of the material, using the CycloneDX values,
               e.g., 'private-key', 'public-key', 'secret-key', or
               'password'.";
          }
          leaf state {
            type enumeration {
              enum pre-activation {
                description
                  "Generated but not yet authorized for use.";
              }
              enum active {
                description
                  "Authorized for use.";
              }
              enum suspended {
                description
                  "Use temporarily suspended.";
              }
              enum deactivated {
                description
                  "Not to be used to apply cryptographic
                   protection.";
              }
              enum compromised {
                description
                  "Known or suspected to be compromised.";
              }
              enum destroyed {
                description
                  "Destroyed.";
              }
            }
            description
              "The state of the key.";
            reference
              "NIST SP 800-57 Part 1 Rev. 5, Section 7";
          }
          leaf size {
            type uint32;
            units "bits";
            description
              "The size of the material.";
          }
          leaf format {
            type string;
            description
              "The encoding format, e.g., 'PEM' or 'DER'.";
          }
          leaf creation-date {
            type yang:date-and-time;
            description
              "The time at which the material was created.";
          }
          leaf expiration-date {
            type yang:date-and-time;
            description
              "The time at which the material expires.";
          }
          container secured-by {
            description
              "The mechanism protecting the material.";
            leaf mechanism {
              type string;
              description
                "The protection mechanism, e.g., 'HSM', 'TPM',
                 'Software', or 'None'.";
            }
            leaf algorithm-ref {
              type leafref {
                path "../../../../crypto-asset/asset-id";
              }
              description
                "The algorithm used to protect the material, e.g.,
                 a key-wrap algorithm.";
            }
          }
        }
        container protocol-properties {
          when "derived-from-or-self(../asset-type, "
             + "'nwic:protocol')";
          description
            "Properties of a protocol.";
          leaf protocol-type {
            type string;
            description
              "The protocol, using the CycloneDX values, e.g.,
               'tls', 'ssh', 'ipsec', 'ike', 'dtls', or 'quic'.";
          }
          leaf version {
            type string;
            description
              "The protocol version, e.g., '1.3'.";
          }
          list cipher-suite {
            key "name";
            description
              "The cipher suites supported by the protocol.";
            leaf name {
              type string;
              description
                "The cipher suite name, e.g.,
                 'TLS_AES_256_GCM_SHA384'.";
            }
            leaf-list identifier {
              type string;
              description
                "The code points of the cipher suite, e.g.,
                 '0x13,0x02'.";
            }
            leaf-list algorithm-ref {
              type leafref {
                path "../../../../crypto-asset/asset-id";
              }
              description
                "The algorithms that make up the cipher suite.";
            }
          }
        }
      }
    }
  }
}
<CODE ENDS>
~~~
{: #fig-module title="ietf-network-inventory-crypto YANG Module"}

## Operational Considerations

As with the base model, this module provides a read-only view of the cryptographic inventory known to the controller. How the controller obtains the information, e.g., by importing CBOMs or by the methods in {{discovery}}, is outside the scope of this document.

The inventory needs to be updated when cryptographic assets change, e.g., after a software upgrade, a certificate renewal, or a configuration change. The "last-discovered" leaf allows consumers to detect stale entries.

Network-wide inventories can be large. Clients can use protocol filtering mechanisms, e.g., subtree or XPath filtering in NETCONF, to retrieve only the data they need, such as the algorithms whose "nist-quantum-security-level" is 0.

# Potentially Related Working Groups

* IVY -- {{inventory}} augments the base network inventory model {{I-D.ietf-ivy-network-inventory-yang}} with cryptographic assets.
* SCITT -- extend SBOMs to adapt CBOMs.
* PQUIP -- as a migration guideline.

# Tools Compilation

Due to limited time, compiling existing CADI tools remains open and will be done in next version.

# Security Considerations

This document has no further security considerations.

# IANA Considerations

TBD

--- back

# Example {#example}

{{fig-example}} shows an instance of the module in JSON encoding {{RFC7951}}. Network element "ne-1" runs a TLS 1.3 implementation that is used on the management plane and provided by component "mpu-1". The TLS implementation depends on a device certificate and on the hybrid key exchange X25519MLKEM768. The key exchange includes ML-KEM-768 (NIST quantum security level 3), but the device certificate still relies on ECDSA with P-256, which is quantum-vulnerable (level 0). AES-256-GCM was imported from the vendor CBOM listed in "cbom-ref".

~~~
=============== NOTE: '\' line wrapping per RFC 8792 ================

{
  "ietf-network-inventory:network-inventory": {
    "network-elements": {
      "network-element": [
        {
          "ne-id": "ne-1",
          "components": {
            "component": [
              {
                "component-id": "mpu-1",
                "class": "iana-hardware:module"
              }
            ]
          },
          "ietf-network-inventory-crypto:crypto-inventory": {
            "cbom-ref": [
              {
                "serial-number": "urn:uuid:3e671687-395b-41f5-a30f-\
                   a58921a69b79",
                "version": 1,
                "spec-version": "1.7",
                "uri": ["https://vendor.example.com/ne-1.cdx.json"],
                "lifecycle-phase": ["build"]
              }
            ],
            "crypto-asset": [
              {
                "asset-id": "tls-1",
                "asset-type": "protocol",
                "name": "TLSv1.3",
                "usage": ["management-plane"],
                "provided-by": ["mpu-1"],
                "depends-on": ["dev-cert", "x25519mlkem768"],
                "discovery": {
                  "method": ["simulated-handshake"],
                  "confidence": "1.0"
                },
                "protocol-properties": {
                  "protocol-type": "tls",
                  "version": "1.3",
                  "cipher-suite": [
                    {
                      "name": "TLS_AES_256_GCM_SHA384",
                      "identifier": ["0x13,0x02"],
                      "algorithm-ref": ["aes-256-gcm"]
                    }
                  ]
                }
              },
              {
                "asset-id": "x25519mlkem768",
                "asset-type": "algorithm",
                "name": "X25519MLKEM768",
                "depends-on": ["x25519", "ml-kem-768"],
                "algorithm-properties": {
                  "primitive": "combiner"
                }
              },
              {
                "asset-id": "x25519",
                "asset-type": "algorithm",
                "name": "x25519",
                "algorithm-properties": {
                  "primitive": "key-agree",
                  "algorithm-family": "ECDH",
                  "elliptic-curve": "other/Curve25519",
                  "nist-quantum-security-level": 0
                }
              },
              {
                "asset-id": "ml-kem-768",
                "asset-type": "algorithm",
                "name": "ML-KEM-768",
                "algorithm-properties": {
                  "primitive": "kem",
                  "algorithm-family": "ML-KEM",
                  "parameter-set-identifier": "768",
                  "nist-quantum-security-level": 3
                }
              },
              {
                "asset-id": "aes-256-gcm",
                "asset-type": "algorithm",
                "name": "AES-256-GCM",
                "bom-link": "urn:cdx:3e671687-395b-41f5-a30f-\
                   a58921a69b79/1#aes-256-gcm",
                "discovery": {
                  "method": ["cbom-declaration"]
                },
                "algorithm-properties": {
                  "primitive": "ae",
                  "algorithm-family": "AES",
                  "parameter-set-identifier": "256",
                  "mode": "gcm",
                  "execution-environment": "software-plain-ram",
                  "nist-quantum-security-level": 5
                }
              },
              {
                "asset-id": "dev-cert",
                "asset-type": "certificate",
                "name": "ne-1 device certificate",
                "usage": ["device-identity", "management-plane"],
                "depends-on": ["ecdsa-p-256"],
                "certificate-properties": {
                  "subject-name": "CN=ne-1.example.com",
                  "issuer-name": "CN=Example CA",
                  "not-valid-after": "2028-01-01T00:00:00Z",
                  "certificate-format": "X.509",
                  "state": "active"
                }
              },
              {
                "asset-id": "ecdsa-p-256",
                "asset-type": "algorithm",
                "name": "ECDSA-P-256",
                "algorithm-properties": {
                  "primitive": "signature",
                  "algorithm-family": "ECDSA",
                  "elliptic-curve": "nist/P-256",
                  "nist-quantum-security-level": 0
                }
              }
            ]
          }
        }
      ]
    }
  }
}
~~~
{: #fig-example title="Example of Cryptographic Inventory"}
