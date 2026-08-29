# Automated Acres Benchmark (AAB)

> **Status: Experimental / Active Development**
>
> AAB is not ready for production use. Interfaces, methods, schemas, safety logic, and result formats may change while validation work continues.

## What AAB Is

Automated Acres Benchmark is a benchmarking and hardware-validation project focused on more than raw speed.

AAB is intended to measure and document:

- CPU performance
- GPU compute performance
- AI workload performance
- Vulkan compute performance
- idle power consumption
- loaded power consumption
- performance per watt
- thermal behavior
- sustained-load stability
- hardware and sensor behavior
- environmental conditions
- repeatability between runs

The long-term goal is a benchmark that helps users understand **how fast a system is, how much power it uses to achieve that performance, how safely it operates, and whether the result can be reproduced**.

## Planned Test Areas

Current validation work is organized around capabilities such as:

- idle power testing
- CPU benchmarks
- native AI GPU benchmarks
- Vulkan GPU compute benchmarks
- combined CPU + GPU workloads
- CPU stability testing
- GPU stability testing
- full-system stability testing
- monitor-only telemetry
- result comparison
- export/share workflows
- hardware and sensor inspection
- optional integration with other benchmark suites

## Safety Is Part of the Benchmark

AAB treats thermal limits, failed sensors, invalid telemetry, interrupted tests, and incomplete configuration as engineering conditions rather than cosmetic warnings.

A benchmark result is not useful if obtaining it requires unsafe behavior or if the system state cannot be trusted afterward.

Development under this directory therefore follows additional rules in [`AGENTS.md`](AGENTS.md), including strict requirements for:

- thermal abort behavior
- restoration of changed system settings
- transactional sensor and power-backend configuration
- live validation before activating external telemetry
- result integrity and methodology versioning
- safe handling of machine-specific information

## External Telemetry

AAB is being designed to support external environmental and power data from systems such as:

- Home Assistant
- network-connected power monitors
- generic JSON endpoints
- supported environmental sensors
- explicitly configured external commands

External integrations must prove that they can return a valid live reading before AAB saves or activates them.

Public examples use fictional values such as:

```text
http://192.168.1.10:8123
<HA_TOKEN>
sensor.server_room_temperature
sensor.server_power
```

Never publish real credentials or private infrastructure details.

## Results

AAB is intended to produce both:

- **human-readable output** for people learning about their system
- **machine-readable output** for comparison, analysis, and future tooling

Result sets should identify the AAB version, methodology version, schema version, test status, relevant environment information, and integrity data needed to understand what was actually measured.

## Current Repository State

The AAB directory is being established before the current validation script line is published.

Existing development builds require additional sanitization, review, and validation before they are suitable for a public repository. In particular, no development file should be copied into this repository until private infrastructure values have been removed and the file has been reviewed against the repository privacy rules.

## Planned Layout

As AAB matures, this directory may grow to include areas such as:

```text
aab/
├── AGENTS.md
├── README.md
├── docs/
├── examples/
├── tests/
└── source or release artifacts
```

The layout will be introduced only as those components become useful; empty structure will not be created merely for appearance.

## Development Rule

All repository-wide instructions in the root [`AGENTS.md`](../AGENTS.md) apply here.

The AAB-specific [`AGENTS.md`](AGENTS.md) adds stricter benchmark, telemetry, safety, methodology, and result-integrity requirements.
