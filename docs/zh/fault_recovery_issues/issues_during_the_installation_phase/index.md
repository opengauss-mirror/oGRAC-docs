# 安装阶段问题

本目录汇总 oGRAC 在 **安装阶段** 常见的失败问题及其定位与解决方案，重点面向 **首次部署或缺乏共享存储经验的开发者**，帮助在安装失败时快速判断问题类型、缩小排查范围。

## 常见问题列表

- [`CMS` 安装失败——残留用户家目录缺失 `.bashrc` 文件](./cms_installation_failed_missing_bashrcin.md)
- [操作系统语言为中文导致安装时无法获取 `NUMA` 信息卡住](./os_environment_language_issue.md)
- [因两节点机器时间不同步导致安装双节点 oGRAC 失败的问题](./installation_failure_due_to_node_time_is_out_of_sync.md)
- [`CM` 投票盘（`gcc-disk`）异常导致安装失败](./installation_failure_due_to_cm_voting_disk_abnormal.md)
- [`DSS` `LUN` 注册冲突导致安装失败](./installation_failure_due_to_dss_lun_registration_conflict.md)
