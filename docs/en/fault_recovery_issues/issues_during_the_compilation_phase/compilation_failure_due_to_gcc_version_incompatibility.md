# Compilation Failure of patchelf Due to GCC Version Incompatibility and Third-Party Library Path Configuration Errors

<!-- md-trans-meta sourceCommit=unknown translatedAt=2026-08-05T07:46:52.413Z pushedAt=2026-08-17T00:46:29.638Z -->

## 1. Symptom

When compiling patchelf 0.18.0 in an openEuler 20.03 aarch64 environment following the official documentation, two types of errors were encountered in sequence.

1. An error occurred during the compilation phase when using the system default GCC 7.3.0.

   ```shell
   patch-elf2.cc:2072:21: error: template argument deduction failed
   std::vector tmp(dst.begin(), dst.end());
   ```

   ![Patchelf Compilation Issue](PatchelfComilation.png)

2. After switching to GCC 10.3.1 from the oGRAC third-party package, an error occurred during the configure phase.

   ```shell
   configure: error: C compiler cannot create executables
   See `config.log' for more details
   ```

   Upon inspecting `config.log`, the core error can be identified:

   ```shell
   /tmp/patchelf/openGauss-third_party_binarylibs_openEuler_2203_arm/buildtools/gcc10.3/gcc/bin/../libexec/gcc/aarch64-unknown-linux-gnu/10.3.1/cc1: error while loading shared libraries: libisl.so.15: cannot open shared object file: No such file or directory
   ```

3. When attempting to manually mount the dependency library path, an export exception was triggered due to a shell syntax error.

   ```shell
   bash: export: `:/tmp/patchelf/openGauss-third_party_binarylibs_openEuler_2203_arm/buildtools/gcc10.3/isl/lib': not a valid identifier
   ```

## 2. Issue Locating

1. First, the error reported during the `make` phase was analyzed, confirming it was a compatibility issue with a new C++17 feature, and the root cause was identified as an outdated GCC version.
2. After switching to GCC 10.3.1, the `config.log` was examined, where it was found that `cc1` failed to load `libisl.so.15`, confirming a missing compiler dependency.
3. The command `find $THIRD -name 'libisl*'` was executed to search for library files within the third-party package, confirming that `libisl.so.15` was actually located in the `buildtools/gcc10.3/isl/lib` directory, rather than the previously misidentified `buildtools/isl/lib` root directory.
4. After correcting `LD_LIBRARY_PATH`, the `ldd` command was used to verify the loading of compiler dependencies, confirming that `libisl.so.15` could be properly recognized.

## 3. Root Cause

This issue is a compilation failure caused by the accumulation of multiple configuration deviations. The core causes fall into two categories:

1. **Initial compilation root cause**: patchelf 0.18.0 introduced the C++17 class template argument deduction (CTAD) feature, whereas gcc 7.3.0, which is shipped by default with openEuler 20.03, only provides experimental support for C++17. The standard library does not implement the overload that enables `std::vector` to automatically deduce types from an iterator pair, which directly led to the source code compilation failure.
2. **Derived root causes after switching compilers**:
   - Path perception deviation: The runtime libraries `isl`, `mpfr`, and `gmp` in the oGRAC third-party dependency packages are nested under the `buildtools/gcc10.3/` subdirectory rather than the `buildtools` root directory. The initial incorrect path configuration prevented `cc1` from loading `libisl.so.15`.
   - Shell syntax error: When using a backslash `\` for line continuation to concatenate environment variables, a space was mistakenly added at the beginning of the next line, causing the Shell to misinterpret the colon-prefixed path as a new variable name, triggering the `not a valid identifier` error and preventing the dependent library path from actually taking effect.

## 4. Solution

### 1. Regular Repair

Recompile after correcting the configuration as follows:

```bash
# Define the third-party package root directory.
THIRD=/tmp/patchelf/openGauss-third_party_binarylibs_openEuler_2203_arm
# Fix the dependency library path (note that there is no space after the backslash, and the path points to the gcc10.3 subdirectory).
export LD_LIBRARY_PATH=$THIRD/buildtools/gcc10.3/gcc/lib64\
:$THIRD/buildtools/gcc10.3/isl/lib\
:$THIRD/buildtools/gcc10.3/mpfr/lib\
:$THIRD/buildtools/gcc10.3/gmp/lib\
:$LD_LIBRARY_PATH
# Thoroughly clean the old compilation cache.
cd /tmp/patchelf
make clean
rm -f config.cache config.status Makefile
# Re-execute the compilation process.
./bootstrap.sh
./configure --prefix=/usr/local
make -j$(nproc)
make install
# Verify the version.
patchelf --version  # patchelf 0.18.0 indicates success.
```

> **NOTE**: Old compilation caches must be cleaned; otherwise, configure will reuse the erroneous historical configuration, causing the compilation failure to recur.

### 2. Alternative Solution (Cross-Node Copy)

If a complete source-code compilation is not required, the binary file can be directly copied from a successfully deployed oGRAC node without adjusting the compilation environment.

```bash
# Copy the patchelf binary from a successful node.
scp root@<IP address of the successfully deployed node>:/usr/local/bin/patchelf /usr/local/bin/
chmod +x /usr/local/bin/patchelf
patchelf --version
```
