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
The GetPCR RPC provides a standardized gRPC interface for retrieving Platform Configuration Register (PCR) values from vendors. This service is essential for establishing a "Golden" reference of measurements used in remote attestation and verified boot processes.

PCR values represent the state of a device's boot chain, from the initial Root of Trust through the kernel and container layers. By providing a common proto definition, this service allows network operators to query expected PCR measurements across different hardware models and software versions, ensuring that the device's integrity can be validated against a known-good baseline.

### Key Components

### Integrity Measurement: 
Supports both TPM 1.2 and TPM 2.0 PCR banks, covering various stages of the boot process defined in the BootStage enumeration (e.g., BIOS, Boot Loader, Kernel).

### Flexible Querying: 
Users can retrieve specific PCR sets based on a combination of hardware models, software/firmware image versions, and preferred hash algorithms (SHA256, SHA512, etc.).

### Discovery RPCs: 
Includes helper methods to fetch lists of supported hardware models, bootloader versions, and software versions available in the vendor's database.




