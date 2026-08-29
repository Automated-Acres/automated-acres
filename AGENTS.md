# Automated Acres — Repository Instructions

## Purpose

Automated Acres is an open-source project focused on practical technology for modern homesteading.

The project exists to build, test, document, and teach reproducible systems involving areas such as:

- local AI
- self-hosted infrastructure
- Proxmox and Linux
- Home Assistant
- energy monitoring and management
- solar, batteries, and generators
- networking
- sensors and telemetry
- irrigation and water systems
- livestock and property automation
- monitoring and data analysis
- open-source scripts and tools

The goal is not merely to produce working code.

The goal is to create systems that are understandable, testable, maintainable, safe, and documented well enough that another person can learn how they work and reproduce them.

---

# Core Development Workflow

## Protect `main`

Never commit directly to `main`.

All normal repository changes must follow this workflow:

1. Start from the current `main` branch.
2. Work on a separate task or feature branch.
3. Make the requested changes.
4. Run all appropriate validation and tests.
5. Review the resulting diff for unintended changes.
6. Commit the work using a clear Conventional Commit message.
7. Push the branch.
8. Create a pull request targeting `main`.
9. Clearly summarize what changed and what was tested.
10. Stop after creating the pull request.

Never merge a pull request automatically.

A human must review and explicitly approve the merge.

When practical, prefer squash merging so each completed pull request becomes one clear commit on `main`.

---

# Scope Discipline

Only change files necessary for the requested task.

Do not perform unrelated cleanup, refactoring, formatting, dependency changes, or restructuring unless they are required for the task or explicitly requested.

If an unrelated problem is discovered:

- document it,
- mention it in the task or pull request,
- and leave it unchanged unless fixing it is necessary for the requested work.

Do not expand the scope silently.

---

# Privacy and Security — Critical Rule

This repository is public.

NEVER commit, publish, expose, reproduce, or embed private user or homelab information.

This rule is absolute.

Never include real:

- passwords
- API keys
- access tokens
- refresh tokens
- session cookies
- authentication headers
- SSH private keys
- encryption keys
- recovery keys
- Wi-Fi credentials
- email credentials
- service credentials
- database passwords
- Home Assistant tokens
- GitHub tokens
- cloud credentials
- certificates containing private material
- public IP addresses belonging to the user's infrastructure
- private/internal IP addresses belonging to the user's infrastructure
- MAC addresses
- hostnames that identify the user's private infrastructure
- domain names that expose private infrastructure
- device serial numbers
- personally identifying infrastructure details
- other secrets or environment-specific private information

Do not copy real values into:

- code
- comments
- documentation
- examples
- test fixtures
- screenshots
- log samples
- configuration files
- issue descriptions
- commit messages
- pull request descriptions

Use clearly fictional placeholders instead.

Examples:

```text
192.168.1.10
192.168.1.20
<PROXMOX_HOST>
<HOME_ASSISTANT_HOST>
<HA_TOKEN>
<API_KEY>
<USERNAME>
<PASSWORD>
<DOMAIN>
<DEVICE_ID>
```

Before committing, inspect the diff specifically for accidental credentials, private network information, or identifying infrastructure details.

If private information is encountered while performing a task, treat it as sensitive input and do not reproduce it in repository content.

---

# Safety

Automated Acres projects may interact with:

- electrical systems
- batteries
- solar equipment
- generators
- networking equipment
- servers
- computer hardware
- machinery
- motors
- pumps
- irrigation
- water systems
- livestock equipment
- environmental controls

Code must never assume that a failed command, bad sensor value, missing device, malformed configuration, or partial operation is harmless.

Prefer safe failure modes.

Never introduce destructive behavior casually.

Operations that can:

- delete data,
- overwrite important configuration,
- repartition storage,
- destroy filesystems,
- disable networking,
- alter boot configuration,
- remove working services,
- modify electrical or machinery control behavior,
- or otherwise cause significant disruption

must be clearly identified and guarded.

Where practical:

- validate prerequisites first,
- validate inputs,
- back up existing configuration,
- preserve a rollback path,
- stop on unexpected failures,
- and explain the consequences of the operation.

---

# Script Engineering Standards

Scripts should be safe, readable, portable, and educational.

## Idempotency

Scripts should be safe to rerun whenever practical.

Running a setup script twice should not normally:

- duplicate configuration,
- create duplicate services,
- corrupt existing files,
- or cause unnecessary changes.

Check the existing system state before modifying it.

## Validation

Before making significant changes:

- verify required commands exist,
- verify required files or directories exist,
- verify expected services or devices are present,
- validate user-supplied values,
- and fail with a useful explanation if prerequisites are missing.

Do not silently continue after an important validation failure.

## Error Handling

For Bash scripts, use robust error handling when appropriate, including:

```bash
set -Eeuo pipefail
```

Do not use strict modes mechanically where they would create incorrect behavior.

Handle expected failures explicitly.

Errors should explain:

- what failed,
- why it matters,
- and what the user can do next.

Avoid unexplained stack traces or silent failures when a clearer message can be provided.

## Configuration Changes

Before modifying an important existing configuration file:

1. verify the file exists,
2. inspect the current state when necessary,
3. create an appropriate backup,
4. make the smallest required change,
5. validate the resulting configuration,
6. restart or reload the affected service only when necessary,
7. verify that the service works afterward.

Do not overwrite entire existing configuration files when a smaller targeted edit is sufficient.

## Destructive Operations

Do not delete, destroy, erase, repartition, reset, or overwrite user data without explicit authorization.

If a destructive operation is genuinely required:

- explain it clearly,
- identify what will be affected,
- provide verification steps,
- and require explicit confirmation where technically practical.

## Comments

Comment non-obvious logic.

Comments should explain **why**, not merely restate **what** the next command does.

Avoid excessive comments around self-explanatory commands.

## Help

Substantial command-line scripts should provide:

```text
--help
```

or an equivalent usage mechanism when practical.

Help output should explain:

- purpose
- syntax
- important options
- defaults
- examples
- potentially destructive behavior

---

# Debian and Ubuntu Package Management

Automated Acres commonly uses `nala`, but public scripts must not assume that `nala` is installed.

When writing documentation:

- `nala` may be shown as the preferred interactive package manager.
- Also provide a standard Debian/Ubuntu alternative where useful.

When writing automated scripts:

1. Detect whether `nala` is available.
2. Use it when appropriate.
3. Fall back safely to standard Debian/Ubuntu package tooling when it is not available.

For non-interactive scripting, prefer stable package-management behavior such as `apt-get` when appropriate rather than assuming the interactive `apt` interface.

A script must not fail merely because the user does not have `nala`.

---

# Documentation Is Part of the Product

Code is not considered complete when only the code works.

A user should be able to understand:

- what the project does,
- why it exists,
- what the important components mean,
- how to install it,
- how to configure it,
- how to verify it,
- how to troubleshoot it,
- and how to remove or reverse it when appropriate.

Do not assume that the reader already understands Linux, Proxmox, networking, Git, Home Assistant, or programming concepts.

Explain unfamiliar terms where doing so helps the reader learn.

Avoid unexplained commands.

When practical, documentation for a finished project should contain:

## What It Does

Explain the purpose of the project in plain language.

## How It Works

Explain the major components and their relationship.

## Requirements

List:

- hardware
- software
- operating-system assumptions
- permissions
- network requirements
- dependencies

## Installation

Provide reproducible installation instructions.

## Configuration

Explain important settings rather than merely listing them.

## Usage

Provide practical examples.

## Verification

Explain how the user can prove that the system is working correctly.

## Troubleshooting

Include common failure conditions and useful diagnostic commands.

## Rollback / Uninstall

Where applicable, explain how to safely reverse the changes.

---

# Teaching First

Automated Acres is intended to help people learn.

Do not optimize documentation solely for the shortest possible instructions.

When an important command introduces a concept that a typical reader may not understand, briefly explain what the command or concept means.

Examples include:

- containers
- virtual machines
- bridges
- VLANs
- systemd services
- timers
- mount points
- filesystems
- package repositories
- APIs
- environment variables
- inference
- embeddings
- telemetry
- authentication
- permissions

The objective is not just:

> Run this command.

The objective is:

> Understand what this command does, why it is being run, and how to tell whether it worked.

---

# Script Distribution and Transfer Instructions

Do not assume that the user's workstation runs Linux.

Many users will be managing Linux servers from Windows.

For substantial Linux scripts, prefer distributing a script file rather than instructing users to paste a very large multi-line script directly into a server console.

Documentation should normally provide:

## Windows PowerShell

Show commands for:

- locating or downloading the script,
- transferring it with `scp`,
- and executing it with `ssh`.

Example pattern:

```powershell
scp .\script.sh user@192.168.1.10:/tmp/
ssh user@192.168.1.10 "chmod +x /tmp/script.sh && /tmp/script.sh"
```

## Linux

Provide equivalent commands when useful.

## macOS

Provide equivalent commands when useful.

If Linux and macOS commands are identical, they may still be shown as separate sections when that improves clarity for beginners.

Never use the user's real infrastructure information in examples.

---

# Testing and Validation

Run tests appropriate to the type of file being changed.

At minimum, always inspect the final diff.

Use:

```text
git diff --check
```

before completing a task.

Additional checks should be used when relevant.

## Bash

When applicable:

```text
bash -n
shellcheck
```

## Python

Run appropriate:

- syntax validation
- unit tests
- integration tests

Use project-configured tools such as Ruff when they already exist.

## Markdown

Check Markdown structure and rendering.

Use an existing Markdown linter if the project already provides one.

## YAML

Validate syntax and use existing project linting tools where available.

## Other Languages

Use the repository's existing tests, linters, formatters, and validation tools.

---

# Development Dependencies

Do not add a new:

- linter
- formatter
- testing framework
- package
- runtime dependency
- development dependency

solely for convenience without approval.

Use tools already present in the project when practical.

If a new dependency would materially improve the project, explain:

- what it does,
- why it is useful,
- what maintenance burden it introduces,
- and whether there is a dependency-free alternative.

Wait for approval before adding it.

---

# Formatting and Existing Style

Follow the existing style of the project being modified.

Do not reformat unrelated files.

Do not perform repository-wide formatting as part of an unrelated change.

Keep diffs focused.

---

# Commit Messages

Use Conventional Commit-style commit messages.

Preferred prefixes include:

- `feat:` — new functionality
- `fix:` — bug fix
- `docs:` — documentation-only change
- `test:` — tests
- `refactor:` — internal code restructuring without changing intended behavior
- `perf:` — performance improvement
- `chore:` — maintenance work
- `build:` — build-system or dependency-management changes
- `ci:` — continuous-integration changes

Examples:

```text
feat: add GPU hardware detection
fix: handle unavailable temperature sensor
docs: add Proxmox installation guide
test: add power backend validation
refactor: simplify configuration loading
```

Commit messages should describe what changed, not vague activity such as:

```text
updates
changes
stuff
fix things
```

---

# Pull Requests

After completing and validating a task, create a pull request automatically unless the user explicitly requests otherwise.

A pull request description should include:

## Summary

What changed.

## Why

What problem or requirement the change addresses.

## Scope

Which files or systems were intentionally changed.

## Validation

Exactly what tests, checks, or manual validation were performed.

## Important Notes

Include:

- known limitations,
- compatibility concerns,
- migrations,
- safety concerns,
- follow-up work

when applicable.

Never claim a test passed unless it was actually run.

Never claim hardware behavior was verified if only software-level testing was performed.

---

# Accuracy

Do not invent:

- command output
- hardware results
- benchmark results
- sensor readings
- test results
- compatibility claims
- service status
- API responses

If something could not be tested, state that explicitly.

Distinguish between:

- verified behavior,
- expected behavior,
- assumptions,
- recommendations.

---

# Hardware-Specific Code

Do not assume all systems match the hardware originally used to develop a project.

Detect hardware or capabilities whenever practical.

When hardware-specific behavior is unavoidable:

- document the requirement,
- validate it before use,
- fail safely when unsupported,
- and explain how another user can determine whether their system is compatible.

---

# Portability

Prefer solutions that work across reasonable variations in:

- Linux distributions
- hardware
- network layouts
- usernames
- installation paths
- device names

Do not hard-code environment-specific values unless there is no reasonable alternative.

Expose configurable values clearly.

Use placeholders in documentation.

---

# Repository-Specific Instructions

This root `AGENTS.md` defines the default behavior for the Automated Acres repository.

Individual applications or directories may contain their own `AGENTS.md`.

A directory-specific `AGENTS.md` should contain only the additional or modified rules required for that application.

Examples may include:

```text
/aab/AGENTS.md
/home-assistant/AGENTS.md
/proxmox/AGENTS.md
/local-ai/AGENTS.md
```

Do not place application-specific implementation rules in this root file unless they genuinely apply to the entire repository.

When working inside a project that has more specific instructions, follow those instructions in addition to these repository-wide rules, with the more specific applicable instruction taking precedence where necessary.

---

# Final Task Checklist

Before completing a repository task, verify:

- [ ] The requested task was actually completed.
- [ ] No unrelated changes were introduced.
- [ ] No private or sensitive information was added.
- [ ] No real homelab-specific information was added.
- [ ] Appropriate tests or validation were run.
- [ ] `git diff --check` passes.
- [ ] Documentation was updated when behavior changed.
- [ ] Important configuration changes include safe handling or rollback guidance.
- [ ] New dependencies were not added without approval.
- [ ] Commit messages are clear.
- [ ] A pull request was created.
- [ ] The pull request accurately states what was and was not tested.
- [ ] The pull request was NOT merged automatically.
