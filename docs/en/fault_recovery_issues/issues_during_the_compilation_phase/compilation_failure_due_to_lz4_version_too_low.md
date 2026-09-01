# Compilation Failure Due to lz4 Version Below 1.8.3

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:47:01.289Z pushedAt=2026-08-17T00:46:29.645Z -->

## 1. Symptom

When compiling oGRAC by running `sh local_install.sh compile -b debug` or `sh build_ograc.sh debug --with-dss`, the compilation fails. An error message similar to the following appears:

```text
error: 'LZ$F_INIT_PREFERENCES' undeclared (first use in this function); did you mean 'OG_PRIV_REFERENCES'?
```

## 2. Issue Locating

1. Check the compilation log to confirm whether the error is related to lz4.
2. Check the lz4 version currently installed on the system.

   ```bash
   lz4 --version
   # or
   rpm -qa | grep lz4
   ```

3. Check whether the lz4 header file exists.

   ```bash
   ls /usr/include/lz4*.h
   ```

## 3. Root Cause

The compilation of oGRAC depends on the lz4 compression library, and the frame API of lz4, such as the `LZ4F_INIT_PREFERENCES` macro and related functions, is used in modules including backup and compression. These frame APIs were introduced in lz4 version 1.8.3 and later.

The current compilation error `LZ4F_INIT_PREFERENCES undeclared` indicates that the compiler cannot find this macro definition in the lz4 header files of the current system, meaning that the installed lz4 version is lower than 1.8.3. Due to version incompatibility, oGRAC is unable to correctly resolve the relevant interfaces during compilation, resulting in a compilation failure.

## 4. Solution

1. Install or upgrade lz4 and its development packages.

   ```bash
   yum install -y lz4 lz4-devel
   ```

2. After installation, verify whether the lz4 version meets the requirement.

   ```bash
   lz4 --version
   ```

   The expected output contains the version number, which must be greater than or equal to 1.8.3. Example:

   ```text
   *** LZ4 command line interface 64-bits v1.9.2, by Yann Collet ***
   ```

3. If the lz4 version meets the requirement, recompile oGRAC.

4. If the lz4 version in the yum repository is still lower than 1.8.3, compile and install a later version of lz4 from source.

   ```bash
   # Download the lz4 source code with root privileges (using version 1.9.4 as an example).
   wget https://github.com/lz4/lz4/releases/download/v1.9.4/lz4-1.9.4.tar.gz
   tar -zxf lz4-1.9.4.tar.gz
   cd lz4-1.9.4
   make
   make install
   # After installation, verify that the lz4 version meets the requirement.
   lz4 --version
   ```
