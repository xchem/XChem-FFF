# XChem Fast Forward Fragments (FFF)

![XChem Fast Forward Fragments logo](docs/images/FFF_logo_text.png)

Welcome to the XChem Fast Forward Fragments (FFF) documentation repository. This repository is the starting point for external users who want to understand and follow the FFF pipeline used by XChem at Diamond Light Source (DLS).

FFF supports fragment-based compound design as part of the Design, Make, Test and Analyse (DMTA) cycle. The documentation guides users from access and environment setup through scaffold design, elaboration, pose generation, and related workflows.

> **Documentation status:** The FFF documentation is under active development. Instructions and workflows may change.

FFF version 1 has been released. For a stable version of the documentation, refer to the repository tags for the corresponding release.

## Start here

Read the documentation in this order:

1. [Onboarding](docs/source/onboarding.md) — prepare DLS account access, connect to the IRIS compute cluster, and request Squonk access.
2. [Setup](docs/source/setup.md) — install and configure the FFF tools and prepare the execution environment.
3. Follow the relevant workflow guide:
   - [Scaffold design](docs/source/design.md)
   - [Scaffold elaboration](docs/source/elaboration.md)
   - [Pose generation](docs/source/pose_generation.md)
4. [Contribution](docs/source/contribution.md) — learn how to propose and integrate tools or workflow improvements.

The [documentation index](docs/source/index.md) links to the available guides. The documentation is also configured to build with Sphinx and Read the Docs.

## Access and prerequisites

The workflows described here are designed primarily for the DLS IRIS compute cluster and Squonk. Running them may require a valid DLS FedID, appropriate proposal or group access, IRIS access, and a Squonk account. Some access must be arranged through the DLS User Office, IT Helpdesk, or an FFF coordinator; see [Onboarding](docs/source/onboarding.md) for details.

Windows is not natively supported for the command-line workflow. The onboarding guide describes using Windows Subsystem for Linux (WSL).

## What is in this repository?

- `docs/source/` — the source pages for the user documentation.
- `templates/` — Jupyter notebook templates used in the documented workflows.
- `scripts/` — helper scripts for running and integrating pipeline tools.
- `docs/` — Sphinx configuration and documentation build requirements.

This repository provides workflow documentation and supporting materials; access to the compute and software services described in the guides is managed separately.

## Help and contributions

If you are an external user and need access or encounter an issue with the documented workflow, contact your FFF coordinator. If you would like to contribute software or improve the pipeline, start with the [contribution guide](docs/source/contribution.md).
