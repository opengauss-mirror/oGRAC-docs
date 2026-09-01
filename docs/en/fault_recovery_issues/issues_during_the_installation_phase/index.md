# Issues During the Installation Phase

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:22.006Z pushedAt=2026-08-17T00:59:35.401Z -->

This directory provides a summary of common failures encountered during the **installation phase** of oGRAC, along with corresponding cause analysis and solutions. It is primarily intended for **developers performing initial deployment or those lacking experience with shared storage**, helping them quickly identify the type of issue and narrow the troubleshooting scope when an installation fails.

## Common Issues

- [CMS Installation Failure Due to Missing `.bashrc` File in Residual User Home Directory](./cms_installation_failed_missing_bashrcin.md)
- [Installation Stuck When Retrieving NUMA Information Due to Chinese OS Locale](./os_environment_language_issue.md)
- [Two‑Node oGRAC Installation Failure Due to Time Synchronization Issues Between Nodes](./installation_failure_due_to_node_time_is_out_of_sync.md)
- [Installation Failure Due to Abnormal CM Voting Disk (`gcc-disk`)](./installation_failure_due_to_cm_voting_disk_abnormal.md)
- [Installation Failure Due to DSS LUN Registration Conflicts](./installation_failure_due_to_dss_lun_registration_conflict.md)
