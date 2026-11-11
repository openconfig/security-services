# Openconfig Security Services

## SBOM

### CLI

#### Overview

The SBOM CLI tool allows for validation and conversion of SBOM from external sources into the SPDX standard format.
The current formats supported are input in SPDX 2.3 and Cyclone DX 1.6 proto and JSON. These formats will then be validated against an SBOM conformance tool.

SBOM are used to convey the software manifest of a package including a dependencies.  The [NTIA](https://www.ntia.gov/page/software-bill-materials) defines two major formats for SBOMs, SPDX and CycloneDX.  The SBOM CLI will support both formats for conversion and conformance check to OpenConfig SBOM format.

#### Build

* `go build -o sbom_cli cli/main.go`

#### Examples

* Convert CycloneDX 1.6 JSON to SPDX 2.3

```shell
./sbom_cli convert ./cyclonedx.json ./spdx.json --format=cyclonedx-v16-json
```

* Convert and Validate CycloneDX 1.6 JSON to SPDX 2.3

```shell
./sbom_cli convert ./cyclonedx.json ./spdx.json --format=cyclonedx-v16-json --validate
```

* Convert and Validate CycloneDX 1.6 PROTO to SPDX 2.3

```shell
./sbom_cli convert ./cyclonedx.json ./spdx.json --format=cyclonedx-v16-proto --validate
```

### PCRService

### Overview
The FetchSignedRefValues RPC provides a standardized gRPC interface for retrieving reference values, such as reference Platform Configuration Register (PCR) values, from vendors.

Reference values represent the expected state of each software and firmware component on a device. Specifically, reference PCR values reflect the expected state of a device's boot chain—from the initial Root of Trust through the kernel and container layers—extended into TPM PCRs during boot. By providing a common proto definition, this service allows network operators to query expected reference values across different hardware models and software versions, ensuring that the device's integrity can be validated against a known-good baseline.

### Key Components

### Integrity Measurement: 
Supports both TPM 1.2 and TPM 2.0 PCR banks, covering various stages of the boot process defined in the BootStage enumeration (e.g., BIOS, Boot Loader, Kernel).

### Flexible Querying: 
Users can retrieve specific PCR sets based on a combination of hardware models, software/firmware image versions, and preferred hash algorithms (SHA256, SHA512, etc.).

### Discovery RPCs: 
Includes helper methods to fetch lists of supported hardware models, bootloader versions, and software versions available in the vendor's database.

### Signed Reference Values:
PCRResponse has no integrity protection on its own. To make it verifiable, vendors sign the binary-serialized PCRResponse and wrap it in a `SignedReferencePcrProto` envelope (see `TbsReferenceValues` and `SignedReferencePcrProto` in `proto/securityservices.proto`).

Signed reference values can be provided as a binary-serialized `SignedReferenceMeasurements` in two ways.  NOTE: Text format is not accepted.

* Binary-serialized `SignedReferenceMeasurements` dropped next to the image as a file, e.g. `signed.ref.measurements.binpb`. 
* Returned by the `FetchSignedRefValues` RPC.

Protobuf messages can have multiple serialized formats, and two different protobuf messages can have the same exact serialized format on the wire. Consequently, to avoid type manipulation attacks, the signature for the signed protobuf format includes a fixed prefix plus an exact binary-serialized message. Therefore, verifiers must never re-encode the payload.

Only ECDSA P-256/SHA-256, ECDSA P-384/SHA-384 and ML-DSA-87 are accepted. The certificate chain must validate against the vendor's root CA, which is shared out of band.



