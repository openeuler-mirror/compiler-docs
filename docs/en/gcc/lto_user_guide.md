# Introduction to Link-Time Optimization

In traditional compilation, GCC compiles and optimizes individual source files (compilation units) to generate .o object files containing assembly code. The linker then processes these `.o` files, resolving symbol tables and performing relocations to create the final executable. However, the linker, which has access to cross-file function call information, operates on assembly code and cannot perform compilation optimizations. Conversely, the compilation stage capable of optimizations lacks global cross-file information. While this approach improves efficiency by recompiling only modified units, it misses many cross-file optimization opportunities.

Link-Time Optimization (LTO) addresses this limitation by enabling optimizations during the linking phase, leveraging cross-compilation-unit call information. To achieve this, LTO preserves the Intermediate Representation (IR) required for optimizations until linking. During linking, the linker invokes the LTO plugin to perform whole-program analysis, make better optimization decisions, and generate more efficient IR. This optimized IR is then converted into object files with assembly code, and the linker completes the standard linking process.

# Enabling LTO in Version Builds

## Background

Many international communities have adopted LTO in their version builds to achieve better performance and smaller binary sizes. LTO is emerging as a key area for exploring compilation optimization opportunities. Starting with version 24.09, openEuler will introduce LTO in its version builds.

## Solution

To enable LTO during package builds, we will add `-flto -ffat-lto-objects` to the global compilation options in the macros of **openEuler-rpm-config**. The `-flto` flag enables Link-Time Optimization, while `-ffat-lto-objects` generates fat object files containing both LTO object information and the assembly information required for regular linking. During the build process, LTO object information is used for optimizations. However, since LTO object files are not compatible across GCC versions, we remove the LTO-related fields from `.o/.a` files before packaging them into `.rpm` files, retaining only the assembly code needed for regular linking. This ensures that static libraries remain unaffected.

## Scope of Enablement

Due to the significant differences between LTO and traditional compilation workflows, and to minimize the impact on version quality, LTO is currently enabled for only 500+ packages. The list of these packages is available in **/usr/lib/rpm/%{_vendor}/lto_white_list**. These whitelisted applications have been successfully built and passed their test suites with LTO enabled. The LTO compilation options (`-flto -ffat-lto-objects`) are applied only when building whitelisted applications; otherwise, they are omitted.

In future innovation releases, we will work with application maintainers to expand the scope of LTO enablement.

## Notes

The current hot-patching mechanism is incompatible with LTO, causing hot patches to fail when LTO is enabled. We have agreed with the hot-patching team on a solution, which will be implemented in future releases.
