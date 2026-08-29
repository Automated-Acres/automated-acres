# Automated Acres Benchmark — Application Instructions

These instructions apply to all work under `/aab/`.

The repository-wide rules in the root [`AGENTS.md`](../AGENTS.md) remain in force. This file adds AAB-specific requirements only.

---

# Project Status

AAB is experimental and under active validation.

Do not describe AAB as production-ready, generally safe for every system, or fully validated unless that status has been explicitly established.

Do not hide experimental limitations merely to make the project appear more polished.

---

# Current Development Model

The current AAB validation line is intentionally centered on a single-file Bash implementation for Debian, Ubuntu, and Proxmox Linux systems.

Do not split the validation implementation into a multi-file application architecture without explicit approval.

Refactoring is allowed when requested, but the public distribution goal is to keep installation and use understandable for people who may be managing a Linux server from another computer.

---

# Safety Takes Priority Over Score

A higher benchmark score never justifies weakening a safety mechanism.

Safety behavior has priority over:

- benchmark completion,
- benchmark score,
- runtime,
- convenience,
- prettier output,
- or compatibility with one unusual system.

Never silently disable, bypass, lower, or ignore:

- thermal abort logic,
- cooldown behavior,
- workload termination,
- state restoration,
- prerequisite validation,
- sensor validation,
- power validation,
- or contamination checks.

Any requested change that would weaken an existing safety mechanism must be called out explicitly before implementation.

---

# Preserve System State

If AAB changes temporary system settings for a test, it must restore them afterward whenever technically possible.

Examples include:

- CPU governors,
- energy-performance preferences,
- frequency limits,
- boost settings,
- temporary workload configuration,
- temporary files,
- or other benchmark-specific runtime state.

Restoration must be attempted on:

- normal completion,
- benchmark abort,
- expected error,
- SIGINT,
- SIGTERM,
- and other handled termination paths.

Do not leave a machine in a benchmark-specific performance state merely because a test failed.

---

# Thermal Protection

Thermal protection must be deterministic, visible, and fail-safe.

When temperature telemetry used for protection becomes invalid during a test, do not assume the temperature is safe.

Thermal abort behavior should:

1. stop the active workload promptly,
2. record that the run was aborted,
3. record the reason and available details,
4. restore modified system state,
5. enter cooldown behavior when applicable,
6. preserve enough result information to explain what happened.

Do not convert an aborted thermal run into a normal completed result.

Do not remove cooldown logic simply to make testing faster.

---

# Hardware Detection

Never assume a specific CPU, GPU, motherboard, sensor chip, or number of devices.

Detect capabilities when practical.

AAB should handle reasonable cases such as:

- CPU-only systems,
- one GPU,
- multiple GPUs,
- NVIDIA GPUs,
- AMD GPUs,
- Intel GPUs,
- missing optional sensors,
- and systems where a supported benchmark backend is unavailable.

Unsupported hardware should fail clearly or disable only the unsupported test path rather than causing unrelated functionality to fail.

Do not claim support for hardware that has not been tested or for which no reliable backend exists.

---

# Linux Sensor Detection

When using `lm-sensors` and `sensors-detect`:

- prefer the safe/default automatic detection path where appropriate,
- validate a recommended kernel module before persisting it,
- do not persist module names that `modinfo` cannot resolve,
- attempt to load validated modules when appropriate,
- re-detect available hardware after sensor configuration changes,
- and record that sensor detection was performed when the application maintains such state.

Do not blindly copy every string produced by sensor-detection output into a modules-load configuration file.

Interactive probing that can carry additional hardware risk must not be silently triggered.

---

# Benchmark Integrity

AAB results are meaningful only when the test method and environment are understood.

Preserve and validate, where applicable:

- single-instance execution locking,
- sufficient free storage,
- pre-test settling,
- workload contamination detection,
- stable sampling intervals,
- benchmark status,
- methodology version,
- schema version,
- AAB version/build,
- environment information,
- hardware information,
- and test timestamps.

If a change materially alters how a result is produced or interpreted, do not silently reuse the old methodology identifier.

If a change breaks the structure or meaning of machine-readable output, evaluate whether the schema version must change.

Do not make unlike methodology versions appear directly comparable without clearly identifying the difference.

---

# Methodology Versioning

Benchmark method identifiers are part of the result contract.

Changes that may require a methodology-version change include:

- workload replacement,
- workload duration changes that alter score meaning,
- scoring-formula changes,
- sampling-method changes,
- different warm-up or settle behavior,
- different performance aggregation,
- different thermal handling that changes valid run conditions,
- or changes to how power/performance-per-watt is calculated.

Documentation-only changes do not require a methodology-version change.

Bug fixes should be evaluated individually: if the fix changes the meaning of previously produced scores, version the method.

---

# External Telemetry Is Transactional

Every external power, energy, temperature, humidity, environmental, or sensor backend must be configured transactionally.

This requirement applies to all supported external backends, including current or future integrations such as:

- Home Assistant,
- Shelly or similar network power monitors,
- SwitchBot or other environmental devices,
- generic JSON endpoints,
- explicitly configured external commands,
- and future API-based backends.

The configuration sequence must be:

1. Collect required values without activating the new backend.
2. Validate that required fields are not blank.
3. Test the endpoint, command, or device live.
4. Validate authentication when authentication is required.
5. Validate that the requested measurement exists.
6. Validate that the returned reading is numeric.
7. Validate or identify the measurement unit.
8. Convert units only through explicit supported conversions.
9. Present a clear success or failure result.
10. Save and activate the new backend only after all required validation succeeds.

If validation fails, do not partially save or activate the new configuration.

Leave the previously valid backend active.

If there was no previously valid backend, leave the backend set to `none` or the appropriate disabled state.

A failed setup attempt must not break an existing working telemetry configuration.

---

# No Partial Configuration Writes

Do not save individual fields one at a time while an external backend is still being configured.

For example, do not save a new Home Assistant URL and token before proving that the complete requested configuration works.

Hold candidate configuration values until validation succeeds, then commit the complete validated configuration together.

This rule exists so an interrupted or failed setup cannot replace a known-good configuration with a half-configured one.

---

# Home Assistant Integration

Home Assistant configuration must support more than one useful entity when the application design calls for it.

AAB should be able to represent multiple configured entities across supported measurement types, including:

- temperature,
- humidity,
- other supported environmental measurements,
- power,
- and energy.

Do not artificially restrict Home Assistant to only one temperature sensor or one power sensor if the surrounding data model supports multiple sources.

## Required Validation

For every Home Assistant setup, validate as applicable:

- base URL is present,
- token is present,
- endpoint returns an HTTP success response,
- authentication succeeds,
- entity exists,
- entity state is available,
- entity state is numeric when numeric telemetry is required,
- unit metadata exists or can be interpreted safely,
- and the unit is compatible with the selected measurement type.

States such as unavailable, unknown, empty, malformed, or non-numeric must not pass numeric telemetry validation.

## Temperature

Temperature entities must have a supported temperature unit and a numeric reading.

AAB may convert between supported temperature units, but it must retain clear knowledge of the source and displayed unit.

Do not silently treat an unknown unit as Celsius or Fahrenheit.

## Humidity

Humidity entities must provide a numeric reading and a unit compatible with humidity measurement.

## Power and Energy

Power and energy are not interchangeable.

Validate the entity's measurement class and unit so that an energy accumulator is not accidentally treated as instantaneous power, or vice versa.

Only perform explicit supported conversions.

---

# Home Assistant Friendly Names and AAB Labels

When a Home Assistant entity is added, use the entity's Home Assistant friendly name as the default AAB label when one is available.

The user must be able to:

- press Enter to accept the friendly name,
- or enter a custom AAB label.

Apply this behavior consistently to supported Home Assistant entity types, including:

- temperature,
- humidity,
- environmental sensors,
- power,
- energy,
- and other supported sensor classes.

If Home Assistant provides no friendly name, use a sensible fallback based on the entity ID and still allow the user to customize it.

Labels are presentation metadata; changing a label must not change which entity is queried.

---

# Generic JSON Backends

A generic JSON backend must not be accepted merely because the URL returns HTTP 200.

Validate:

- endpoint reachability,
- HTTP success,
- selected JSON path or `jq` selector,
- extracted value presence,
- numeric type or numeric parseability where required,
- and expected unit information when the measurement type requires it.

A selector that returns null, an empty value, an object, an error, or a non-numeric string must fail numeric validation.

---

# External Command Backends

External command telemetry is powerful and must be treated cautiously.

Before activation:

- verify that the command is not blank,
- execute it through the intended controlled path,
- require successful exit status,
- validate returned data,
- validate numeric measurements and units,
- and reject unexpected extra output when it makes parsing ambiguous.

Do not silently use `eval` for user-supplied external telemetry commands.

Do not print secrets embedded in configured commands.

---

# Credential Handling

Runtime configuration may legitimately require local credentials such as an API token.

Credentials must never be committed to the public repository.

When AAB stores local secrets:

- use a local configuration location appropriate for the platform,
- restrict file permissions,
- avoid printing the secret in normal UI or logs,
- redact secrets from diagnostic output,
- redact secrets from exported/shareable results,
- and never place them in example configuration files except as placeholders.

---

# Shareable Results and Privacy

AAB may collect detailed hardware and environment information locally, but anything described as "safe", "shareable", or intended for public export must be actually sanitized.

A shareable export must not expose real:

- passwords,
- API tokens,
- authentication headers,
- private keys,
- internal IP addresses,
- public IP addresses associated with the user's infrastructure,
- MAC addresses,
- private hostnames,
- private domain names,
- device serial numbers,
- or other identifying infrastructure details.

Do not call an export "safe" merely because some fields were omitted.

Review the complete exported dataset for identifying information.

A local full diagnostic may contain machine-specific information when necessary for troubleshooting, but it must be clearly distinguished from a sanitized public/shareable export.

---

# Result Files

When a test completes or aborts, preserve enough structured information to explain the run.

Where applicable, result sets should include or support:

- a manifest,
- machine-readable summary data,
- human-readable summary data,
- sampled telemetry,
- environment information,
- hardware information,
- status,
- abort reason/details,
- methodology version,
- schema version,
- and integrity checksums.

An aborted run should remain identifiable as aborted in every relevant representation.

Do not write `COMPLETED` merely because result files were successfully created.

---

# Integrity Checksums

When AAB produces checksum manifests such as `SHA256SUMS`, generate them only after the relevant result files have reached their final state for that run.

Do not imply that checksums provide authenticity; they provide integrity detection for the files they cover.

Document which files are included and excluded when this matters.

---

# Human-Readable and Machine-Readable Output

AAB serves both learners and automated analysis.

Changes to output should consider both audiences.

Human-readable output should explain:

- what is being measured,
- what the important numbers mean,
- why a run stopped,
- and where results were saved.

Machine-readable output should remain structured, versioned, and unambiguous.

Do not make a prettier terminal screen at the expense of losing structured result data.

---

# Units

Do not mix units implicitly.

For every measured quantity, maintain an unambiguous unit.

Support conversions only when mathematically and semantically valid.

Examples:

- temperature: supported temperature units,
- instantaneous power: supported power units,
- accumulated energy: supported energy units,
- humidity: percentage-compatible units,
- duration: explicit time units.

If a source unit is unsupported or unknown, fail validation rather than guessing.

Display units may be user-configurable without changing the meaning of stored canonical data.

---

# Power and Performance-per-Watt

Performance-per-watt calculations must use power data that is valid for the measurement period.

Do not calculate performance per watt from:

- a failed power backend,
- stale readings presented as live,
- an energy total mistaken for power,
- a missing sample silently replaced with zero,
- or an incompatible measurement unit.

Clearly define whether reported power values are mean, median, minimum, maximum, instantaneous, or otherwise aggregated.

---

# Test Contamination

Background system activity can invalidate benchmark interpretation.

Preserve contamination detection where implemented and document what it means.

Do not silently remove contamination warnings because they are inconvenient.

If a run is allowed to continue despite contamination, the result must make that condition visible where appropriate.

---

# Root and Privileged Operations

Current AAB validation builds may require root privileges because they inspect hardware, load validated sensor modules, manage benchmark state, or change temporary system settings.

Do not remove a root requirement simply to make invocation shorter unless every privileged operation has been redesigned safely.

Conversely, do not run commands as root merely out of habit when elevated privileges are not necessary.

Privileged actions must be justified by the operation being performed.

---

# Runtime Paths and Compatibility

Current validation work uses standard AAB-specific locations such as:

```text
/etc/automated-acres-benchmark
/var/lib/automated-acres-benchmark
/run/automated-acres-benchmark.lock
```

Treat changes to persistent configuration or result locations as compatibility-sensitive.

If these paths change after public releases exist, provide migration or backward-compatibility handling when appropriate.

---

# Testing Changes to AAB

Software-level validation and physical-hardware validation are different.

Use software tests, fixtures, or mocks where useful for:

- configuration parsing,
- unit conversion,
- JSON parsing,
- API-response handling,
- transaction/rollback behavior,
- result serialization,
- and failure paths.

Hardware behavior must be tested on suitable physical hardware before claiming it is verified.

Never fabricate benchmark output to make a test appear successful.

Never disable safety logic merely to make an automated test easier to pass.

---

# External Backend Test Cases

When adding or changing an external telemetry backend, cover failure behavior as well as success behavior.

Relevant cases include:

- blank required field,
- unreachable endpoint,
- timeout,
- HTTP error,
- authentication failure,
- entity or measurement not found,
- unavailable/unknown state,
- malformed JSON,
- selector failure,
- non-numeric reading,
- unsupported unit,
- command failure,
- and successful live reading.

Verify that every failure leaves the previously valid configuration unchanged.

---

# User Interface Behavior

Setup screens should make state transitions obvious.

When testing a new backend, distinguish between:

- candidate configuration,
- validation in progress,
- validation failed,
- validation succeeded,
- and configuration activated.

Do not print "saved", "active", or equivalent wording before the validated configuration is actually committed.

Provide actionable failure messages rather than only raw API or parser output.

---

# Public Examples

All examples committed under `/aab/` must use fictional infrastructure details.

Use examples such as:

```text
192.168.1.10
http://192.168.1.10:8123
<HA_TOKEN>
sensor.server_room_temperature
sensor.server_room_humidity
sensor.server_power
sensor.server_energy
```

Never copy values from a real development machine into documentation, sample configuration, test fixtures, comments, or screenshots.

---

# Definition of Done for an AAB Change

In addition to the root repository checklist, verify as applicable:

- [ ] Safety behavior was not weakened.
- [ ] System state is restored after success, failure, and handled interruption.
- [ ] Hardware assumptions were avoided or explicitly validated.
- [ ] External backend configuration is transactional.
- [ ] Failed backend setup preserves the previous valid configuration.
- [ ] Live readings are validated for numeric value and unit.
- [ ] Home Assistant entity existence and authentication are tested before activation.
- [ ] Home Assistant friendly names are used as default labels when available.
- [ ] Multiple supported Home Assistant entities remain supported.
- [ ] Power and energy are not confused.
- [ ] Methodology/schema versions were reviewed when result meaning changed.
- [ ] Aborted runs remain clearly identified as aborted.
- [ ] Shareable exports contain no private infrastructure information.
- [ ] Machine-readable and human-readable outputs remain consistent.
- [ ] Claims about physical hardware are based on actual hardware validation.
