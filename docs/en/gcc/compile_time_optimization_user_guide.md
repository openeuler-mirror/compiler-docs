# Background

To improve the compilation efficiency of openEuler software packages and enhance CI pipeline and developer productivity, version 25.03 employs GCC with profile-guided optimization (PGO) and link time optimization (LTO), alongside the modern mold linker, to reduce compile time for C/C++ libraries within packages.

# Solution

## Optimization Technology Principles

Mold is an open-source linker that achieves higher linking efficiency than other linkers such as ld, gold, and lld. It speeds up linking through techniques such as more efficient identical COMDAT folding (ICF) algorithm, better parallel programming, and efficient multithreaded libraries. For more details, see <https://github.com/rui314/mold/blob/main/docs/design.md>.

PGO uses the profiling information during program running to guide compilation optimization, thereby improving program performance. The profiling information includes the function call frequency and branch behavior.

LTO enables more inter-module optimizations, eliminating redundant code and unnecessary function calls between modules and improving the execution efficiency of applications compiled with GCC.

## Enablement Method

During GCC compilation, `--disable-bootstrap` is removed, and `BUILD_CONFIG=bootstrap-lto profiledbootstrap` is added to enable PGO and LTO.

In macros of `openEuler-rpm-config`, `-fuse-ld=mold` is appended to LDFLAGS for packages that are included in a trustlist. This enables the packages to use the mold linker during compilation.

## Enablement Scope

PGO and LTO compilation are enabled by default for GCC in version 25.03.

Due to certain functional limitations of the mold linker, such as lack of support for kernel compilation and incomplete support for linker scripts, mold is enabled by using a [trustlist](https://gitee.com/src-openeuler/openEuler-rpm-config/blob/openEuler-25.03/0002-Enable-mold-links-through-whitelist.patch#L49) in version 25.03.

In addition, a more flexible linker configuration mode is provided. Each software package can override the `_ld_use` variable in the spec file to switch the linker. For example:

- `%define _ld_use %{nil}` disables mold for a package that is in the trustlist.

- `%define _ld_use -fuse-ld=xxx` switches to a different linker. Note that when multiple `-fuse-ld` options are defined, the compiler uses the last specified option.

# Precautions

1. The mold linker is enabled for software packages in the trustlist only when mold is available in the build environment.
2. When mold is enabled, the build must use GCC 12 or later.
