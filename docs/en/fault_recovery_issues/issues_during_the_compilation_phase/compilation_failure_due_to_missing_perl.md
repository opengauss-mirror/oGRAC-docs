# Compilation Failure of OpenSSL Due to Missing Perl Components

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:00.019Z pushedAt=2026-08-17T00:46:29.643Z -->

## 1. Symptom

When executing `sh local_install.sh compile -b debug` or `sh build_ograc.sh debug --with-dss` to compile oGRAC, an error message similar to the following is reported:

```text
Can't locate IPC/Cmd.pm in @INC (you may need to install the IPC::Cmd module) (@INC contains: ...)
 at /data0/xxx/oGRAC/open_source/openssl/openssl-3.0.9/util/perl/OpenSSL/config.pm line 19.
BEGIN failed--compilation aborted at /data0/xxx/oGRAC/open_source/openssl/openssl-3.0.9/util/perl/OpenSSL/config.pm line 19.
Compilation failed in require at /data0/xxx/oGRAC/open_source/openssl/openssl-3.0.9/Configure line 23.
...
build_ograc failed.
```

## Issue Locating

1. Check the compilation log to confirm whether the error occurs during the OpenSSL 3.0.9 compilation phase.
2. Check whether the error log contains Perl module missing prompts such as `Can't locate IPC/Cmd.pm`.
3. Verify whether the current system has the complete Perl development package installed. This can be checked using the following command.

   ```bash
   rpm -qa | grep perl
   ```

   Alternatively, check whether the IPC::Cmd module exists using the following command.

   ```bash
   perl -MIPC::Cmd -e 'print "IPC::Cmd ok\n"'
   ```

Errors for other similar modules can also be checked by referring to the preceding command.

## 3. Root Cause

The Configure script of OpenSSL 3.0.9 depends on the IPC::Cmd module of Perl. The absence of Perl core development components in the current operating system environment prevented the OpenSSL configuration phase from executing properly, which in turn caused the entire oGRAC compilation process to fail.

## 4. Solution

1. Install the complete Perl development package.

   ```bash
   yum install -y perl-core
   ```

2. After installation, re-execute the oGRAC compilation.
