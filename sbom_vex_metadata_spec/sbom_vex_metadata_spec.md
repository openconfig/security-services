# SBOM/VEX Metadata Specification

# Introduction

This document specifies how software and hardware publishers MUST encode key metadata within Software Bill of Materials (SBOM) and Vulnerability Exploitability eXchange (VEX) documents. The primary purpose of this specification is to ensure that essential information—specifically **Document Version, Vendor Name, OS Family, OS Version, Hardware Model, and Document Signature**—is consistently provided in a machine-readable format across supported VEX standards (CSAF, CycloneDX) and SBOM standards (SPDX, CycloneDX).

Adherence to these guidelines is necessary to enable automated ingestion and processing of SBOM and VEX data into vulnerability management systems. Document producers should encode this information precisely as described below.

# Supported Formats

The following standard formats are supported for SBOM and VEX documents:

**VEX document:**
1. **CycloneDX 1.6+ (Recommended)**
2. **CSAF (Common Security Advisory Framework) 2.0**

**SBOM document:**
1. **SPDX 2.3**
2. **CycloneDX 1.6+**

# Document Encoding

Every individual SBOM/VEX document **MUST** be uniquely encoded for each specific software or firmware release. It is mandatory that SBOM and VEX data remain separate; they **MUST NOT** be integrated into a single document.

## Mandatory Information for Automated Ingestion

The following fields MUST be populated as specified within the chosen SBOM or VEX format:

1. **Document Version:** A version identifier for the SBOM or VEX document itself.
2. **Product Identification:** Clear identification for the software image the SBOM or VEX statement applies to. This **MUST** include:
   - **Vendor Name:** The canonical name of the product vendor. Vendor names must remain uniform across all document submissions to ensure consistent identification and automated processing across vulnerability management systems.
   - **OS Family:** The base operating system or firmware family (software).
   - **OS Version:** The specific version of the OS or firmware.
   - **PURL:** [Package URL](https://www.packageurl.org/docs/purl/specification-folder) string for the software image to be used for software version-level vulnerability scanning against vulnerability databases. The PURL format depends on the specific OS and is determined by vulnerability advisory feeds.
   - **Hardware Model:** The specific hardware model(s) the software runs on. This should only be included if required (i.e., the SBOM is only relevant for specific hardware models).
3. **Signature:** A digital signature used to verify the authenticity and integrity of the SBOM/VEX document, ensuring it was provided by the stated publisher and has not been tampered with. Signatures must be certificate-based so consumers can validate the certificate chain against a trusted root CA.

# Format-Specific Guidelines

## SBOM

### 1. SPDX 2.3

- **Document Version:** Use the `documentNamespace` field, which is a unique URI (Uniform Resource Identifier) identifying the specific instance of the SBOM.
  - When the SBOM is updated or modified to a new version, this must change to a new, unique URI.
  - To version the document, append a revision number to the end of the URI.
  - Example:
    - First Version: `"documentNamespace": "https://example.com/sbom/os/1.2.3/rev-1"`
    - Second Version (Updated SBOM): `"documentNamespace": "https://example.com/sbom/os/1.2.3/rev-2"`
- **Relationships Required:**
  - For the primary package containing the SBOM metadata to be identified, there must be a (**SINGLE**) relationship specified with the following properties:
    ```json
    {
      "spdxElementId": "<The SPDXID of the main document>",
      "relationshipType": "DESCRIBES",
      "relatedSpdxElement": "<The SPDXID of the primary package>"
    }
    ```
  - For the package containing the hardware model information to be identified, there must be (**SINGLE OR MULTIPLE**) relationships specified with the following properties:
    ```json
    {
      "spdxElementId": "<The SPDXID of the hardware model package>",
      "relationshipType": "RUNTIME_DEPENDENCY_OF",
      "relatedSpdxElement": "<The SPDXID of the main package>"
    }
    ```
- **Product Identification (in primary package):**
  - **Vendor name:**
    - Field: `supplier`
    - JSON path: `packages[].supplier`
    - Format: Must be prefixed with `Organization:` followed by the name.
      - Example: `"supplier": "Organization: AcmeCorp"`
  - **OS family:**
    - Field: `name`
    - JSON path: `packages[].name`
    - Format: String representing the product.
      - Example: `"name": "routeros"`
  - **OS version:**
    - Field: `versionInfo`
    - JSON path: `packages[].versionInfo`
    - Format: String representing the version.
      - Example: `"versionInfo": "1.2.3"`
  - **PURL:**
    - Field: `externalRefs`
    - JSON path: `packages[].externalRefs[]`
    - Format: An array containing an object with the following properties:
      - `"referenceCategory": "PACKAGE-MANAGER"`
      - `"referenceType": "purl"`
      - `"referenceLocator": "<The actual PURL string in the format of pkg:generic/package@<version>?vendor=<vendor_name>>"`
  - **Package purpose:**
    - Field: `primaryPackagePurpose`
    - JSON path: `packages[].primaryPackagePurpose`
    - Format: String representing the package purpose.
      - Example: `"primaryPackagePurpose": "OPERATING-SYSTEM"`
- **Product Identification (in hardware model package):**
  - **Hardware model:**
    - Field: `name`
    - JSON path: `packages[].name`
    - Format: String representing the product.
      - Example: `"name": "<A hardware model that runs the software version that the SBOM is associated with>"`
  - **Package purpose:**
    - Field: `primaryPackagePurpose`
    - JSON path: `packages[].primaryPackagePurpose`
    - Format: String representing the package purpose.
      - Example: `"primaryPackagePurpose": "DEVICE"`
- **Signature:** SPDX lacks native support for digital signatures. Given that certificate-based signatures are required and SPDX files are JSON-based, the document will be signed using JSON Web Signature (JWS).

**Example:**

```json
{
  "spdxVersion": "SPDX-2.3",
  "dataLicense": "CC0-1.0",
  "SPDXID": "SPDXRef-DOCUMENT",
  "name": "routeros-1.2.3",
  "documentNamespace": "https://example.com/sbom/routeros/1.2.3/rev-1",
  "packages": [
    {
      "name": "routeros",
      "SPDXID": "SPDXRef-Package-Main",
      "versionInfo": "1.2.3",
      "supplier": "Organization: AcmeCorp",
      "externalRefs": [
        {
          "referenceCategory": "PACKAGE-MANAGER",
          "referenceType": "purl",
          "referenceLocator": "pkg:generic/routeros@1.2.3?vendor=AcmeCorp"
        }
      ],
      "primaryPackagePurpose": "OPERATING-SYSTEM"
    },
    {
      "name": "ACME-1000",
      "SPDXID": "SPDXRef-Package-Hardware-1000",
      "supplier": "Organization: AcmeCorp",
      "primaryPackagePurpose": "DEVICE"
    }
  ],
  "relationships": [
    {
      "spdxElementId": "SPDXRef-DOCUMENT",
      "relatedSpdxElement": "SPDXRef-Package-Main",
      "relationshipType": "DESCRIBES",
      "comment": "The primary package metadata."
    },
    {
      "spdxElementId": "SPDXRef-Package-Hardware-1000",
      "relatedSpdxElement": "SPDXRef-Package-Main",
      "relationshipType": "RUNTIME_DEPENDENCY_OF",
      "comment": "This software runs on the specified target hardware model."
    }
  ]
}
```

### 2. CycloneDX 1.6+

- **Document Version:** Use the top-level `version` integer field (e.g., `1`).
- **Product Identification:** Use `metadata.component` to provide the required vendor name, OS family, OS version, and PURL identifier.
  - **Vendor name:**
    - Field: `supplier.name`
    - JSON path: `metadata.component.supplier.name`
    - Format: String representing the vendor name.
      - Example: `"supplier": {"name": "AcmeCorp"}`
  - **OS family:**
    - Field: `name`
    - JSON path: `metadata.component.name`
    - Format: String representing the product.
      - Example: `"name": "routeros"`
  - **OS version:**
    - Field: `version`
    - JSON path: `metadata.component.version`
    - Format: String representing the version.
      - Example: `"version": "1.2.3"`
  - **PURL:**
    - Field: `purl`
    - JSON path: `metadata.component.purl`
    - Format: String representing the PURL.
      - Example: `"purl": "pkg:generic/routeros@1.2.3?vendor=AcmeCorp"`
- **Product Identification:** Use `components` and `dependencies` to provide the required hardware model identifiers.
  - For any hardware models associated with the SBOM to be identified, the hardware model needs to be specified under `components` with:
    - `"type": "device"`
    - `"name": "<the associated hardware model string>"`
  - The `bom-ref` of the component with the hardware model information needs to be specified under `dependencies` as a dependency of the OS with:
    - `"ref": "<the bom_ref of the metadata.component>"`
    - `"dependsOn": [ "<the bom_ref of the component/s with the hardware model string>" ]`
- **Signature:** The document must be signed using the options supported in the [CycloneDX specification](https://cyclonedx.org/docs/1.7/json/#signature).

**Example:**

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "version": 1,
  "metadata": {
    "timestamp": "2026-05-23T12:00:00Z",
    "component": {
      "bom-ref": "routeros-os",
      "type": "operating-system",
      "supplier": {
        "name": "AcmeCorp"
      },
      "name": "routeros",
      "version": "1.2.3",
      "purl": "pkg:generic/routeros@1.2.3?vendor=AcmeCorp"
    }
  },
  "components": [
    {
      "bom-ref": "hw-model-1000",
      "type": "device",
      "name": "ACME-1000"
    },
    {
      "bom-ref": "hw-model-1001",
      "type": "device",
      "name": "ACME-1001"
    }
  ],
  "dependencies": [
    {
      "ref": "routeros-os",
      "dependsOn": [
        "hw-model-1000",
        "hw-model-1001"
      ]
    }
  ]
}
```

## VEX

### 1. CycloneDX 1.6+ (**Recommended Format**)

**CycloneDX VEX** documentation leverages identical metadata parameters as those defined for the **CycloneDX SBOM**.

#### Key Advantages of CycloneDX 1.6+

1. **Native Signature Support:** CycloneDX 1.6+ is the only version among evaluated formats that provides native, built-in support for digital signatures, ensuring document integrity.
2. **Widespread Ecosystem Adoption:** Due to its maturity and established ecosystem, standard tooling and ecosystem support are widely available.
3. **Unified VEX and SBOM:** The format enables a seamless connection between SBOM components and exploitability data within a single, integrated specification.

**Example:**

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "version": 1,
  "metadata": {
    "timestamp": "2026-05-23T12:00:00Z",
    "component": {
      "bom-ref": "routeros-os",
      "type": "operating-system",
      "supplier": {
        "name": "AcmeCorp"
      },
      "name": "routeros",
      "version": "1.2.3",
      "purl": "pkg:generic/routeros@1.2.3?vendor=AcmeCorp"
    }
  },
  "components": [
    {
      "bom-ref": "hw-model-1000",
      "type": "device",
      "name": "ACME-1000"
    },
    {
      "bom-ref": "hw-model-1001",
      "type": "device",
      "name": "ACME-1001"
    }
  ],
  "dependencies": [
    {
      "ref": "routeros-os",
      "dependsOn": [
        "hw-model-1000",
        "hw-model-1001"
      ]
    }
  ],
  "vulnerabilities": [
    {
      "bom-ref": "MyVuln-CVE-2026-9999",
      "id": "CVE-2026-9999",
      "description": "Example vulnerability to demonstrate VEX integration.",
      "analysis": {
        "state": "not_affected",
        "justification": "code_not_present",
        "detail": "The vulnerable module is excluded from the routeros build."
      },
      "affects": [
        {
          "ref": "routeros-os"
        }
      ]
    }
  ]
}
```

### 2. CSAF 2.0

- **Document Version:** Use `document.tracking.version` (e.g., `"1.1.0"`).
- **Product Identification:** Utilize the `product_tree` to structure the product hierarchy:
  - **Vendor Name:** Extracted from the branch with `category`: `"vendor"`.
  - **OS Family:** Extracted from the branch with `category`: `"product_family"`.
  - **OS Version:** Extracted from the branch with `category`: `"product_version"`.
  - **Hardware Model:** Extracted from the branch with `category`: `"product_name"`.

**Signature:** According to the [CSAF Specification](https://docs.oasis-open.org/csaf/csaf/v2.0/os/csaf-v2.0-os.html#7119-requirement-19-signatures), native signature support is not provided within the framework. While OpenPGP is suggested for signatures, it lacks support for certificate-based signing.

Given that certificate-based signatures are required and CSAF VEX files use JSON formatting, the document will be signed with JSON Web Signature (JWS).

**CSAF Example with Document Version:**

```json
{
  "document": {
    "category": "vex",
    "csaf_version": "2.0",
    "title": "VEX Advisory for AcmeCorp RouterOS 2.1.3",
    "publisher": {
      "category": "vendor",
      "name": "AcmeCorp"
    },
    "tracking": {
      "id": "ACME-RX5000-2026-001",
      "status": "final",
      "version": "1.1.0",
      "revision_history": [
        {
          "date": "2026-05-22T10:00:00Z",
          "number": "1.0.0",
          "summary": "Initial release."
        }
      ],
      "initial_release_date": "2026-05-22T10:00:00Z",
      "current_release_date": "2026-05-23T12:00:00Z"
    }
  },
  "product_tree": {
    "branches": [
      {
        "category": "vendor",
        "name": "AcmeCorp",
        "branches": [
          {
            "category": "product_family",
            "name": "RouterOS",
            "branches": [
              {
                "category": "product_version",
                "name": "2.1.3",
                "branches": [
                  {
                    "category": "product_name",
                    "name": "RX5000",
                    "product_id": "pkg:generic/RouterOS@2.1.3?vendor=AcmeCorp&hardware=RX5000"
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

# Sign Documents Using [JSON Web Signature (JWS)](https://datatracker.ietf.org/doc/html/rfc7515)

## JWS Serialization Protocol

The [JWS JSON Serialization format](https://tools.ietf.org/html/rfc7515#section-3.2) must be employed for all signed VEX/SBOM documentation. Furthermore, signed JWS files are required to use the **.json** file extension.

## Signing Procedures

- The JWS `payload` consists of the complete original JSON metadata, formatted as a UTF-8 string and Base64URL encoded (without padding).
- All digital signatures are required to be certificate-based.
- To facilitate trust establishment, document publishers must make their Root CA certificate available to consumers through a designated secure channel.

## Cryptographic Recommendations

- It is recommended to utilize robust algorithms such as ES256 (ECDSA P-256 with SHA-256) or RS256 (RSA with SHA-256).

**Example:**

**1. Input Files:**

- **SPDX SBOM file (`sbom.json`):**
  ```json
  { 
    "spdxVersion": "SPDX-2.3",
    "SPDXID": "SPDXRef-DOCUMENT",
    "name": "routeros-1.2.3"
  }
  ```

- **Certificate file (`cert.pem`):**
  ```pem
  -----BEGIN CERTIFICATE-----
  MIICVjCCAb6gAwIBAgIGAYXQ/a0KMA0GCSqGSIb3DQEBCwUAMFsx...
  -----END CERTIFICATE-----
  ```

**2. JWS File (Signed SBOM File):**

```json
{ 
  "payload": "BASE64URL_ENCODED_SBOM",
  "protected": "BASE64URL_ENCODED_HEADER",
  "signature": "BASE64URL_ENCODED_SIGNATURE"
}
```

**Encoding Explanation:**

- **Payload (`payload`):** Read the entire `sbom.json` as a UTF-8 string, then encode it using Base64URL encoding (without padding).
- **Protected Header (`protected`):** Create a JSON object containing the algorithm and the certificate chain:
  ```json
  {"alg": "RS256", "x5c": ["MIICVjCCAb6gAwIBAgIGAY...z7jQ=="]}
  ```
  Encode this JSON object using Base64URL encoding (without padding).
- **Signature (`signature`):** Create the signing input by concatenating the encoded protected header, a period (`.`), and the encoded payload:
  `encoded_protected_header + "." + encoded_payload`
  Sign this string using the private key corresponding to the certificate, then encode the resulting byte array using Base64URL (without padding).
