# Common Fault Recovery Issues

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:48:07.227Z pushedAt=2026-08-17T00:46:29.669Z -->

This module provides a summary of common stability issues encountered with oGRAC during the **compilation, installation, operation, and uninstallation** phases, as well as corresponding **troubleshooting approaches and solutions** to help quickly identify the problem type and narrow down the scope of investigation when a fault occurs.

## Issue Classification by Phase

Select the corresponding directory based on the phase in which the fault occurs:

- [Compilation Issues](./issues_during_the_compilation_phase/index.md): Covers compilation failures of oGRAC caused by factors such as the GCC version, third-party libraries, and missing dependencies.
- [Installation Issues](./issues_during_the_installation_phase/index.md): Covers installation failures of oGRAC caused by factors such as environment configuration, shared storage (`LUN`, `CM` voting disk, `DSS`), and node time synchronization, and provides corresponding troubleshooting approaches.
- [Running Issues](./issues_during_the_running_phase/index.md): Covers runtime failures of oGRAC caused by factors such as insufficient system resources (for example, memory).
- [Uninstallation Issues](./issues_during_the_uninstallation_phase/index.md): Covers uninstallation failures of oGRAC caused by residual processes, residual shared storage, and other factors.

## Troubleshooting

When locating any issue, it is recommended to follow the troubleshooting sequence below:

1. Prioritize checking the installation and runtime logs to identify the failure phase.
2. Determine whether shared LUNs or CM/DSS components are involved.
3. Retain the binaries for auxiliary analysis when necessary.
4. Verify that the shared disk registration status is consistent with the symbolic links.
