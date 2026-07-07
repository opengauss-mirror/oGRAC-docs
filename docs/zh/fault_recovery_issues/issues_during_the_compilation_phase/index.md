# 编译阶段问题

本目录汇总 oGRAC 在 **编译阶段** 常见的失败问题及其定位与解决方案。

## 常见问题列表

- [因 `gcc` 版本兼容及三方库路径配置错误导致 `patchelf` 编译失败问题](./compilation_failure_due_to_gcc_version_incompatibility.md)
- [因环境缺少 Perl 组件导致 OpenSSL 编译失败的问题](./compilation_failure_due_to_missing_perl.md)
- [因三方库版本与 `OS` 版本不匹配导致编译失败的问题](./compilation_failure_due_to_third_party_inconsistent.md)
- [因系统 `lz4` 版本低于 1.8.3 导致编译失败的问题](./compilation_failure_due_to_lz4_version_too_low.md)
