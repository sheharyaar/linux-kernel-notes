- [Finding Bugs](#finding-bugs)
  - [Code Coverage Tools](#code-coverage-tools)
    - [gcov](#gcov)
    - [kcov](#kcov)
    - [References](#references)
  - [Static Tools](#static-tools)
    - [Sparse](#sparse)
    - [Smatch](#smatch)
    - [Coccinelle](#coccinelle)
    - [References](#references-1)
  - [Test Automation Tools](#test-automation-tools)
    - [KUnit](#kunit)
    - [kselftest](#kselftest)
    - [Linux Test Project](#linux-test-project)
    - [Autotest and (Avocado)](#autotest-and-avocado)
    - [References](#references-2)
  - [Dynamic Tools](#dynamic-tools)
    - [References](#references-3)
  - [Extras](#extras)
    - [KernelCI](#kernelci)

# Finding Bugs

The Linux Kernel is a complex piece of software, and it's not uncommon to find bugs in it. There are several tools and techniques that can be used to find bugs in the kernel.

This document will cover some of the tools and techniques that can be used to find bugs in the kernel. For susbsystem specific tools, refer to the respective subsystem documentation.

## Code Coverage Tools

### gcov

- GCC's coverage testing tool, which can be used with the kernel to get global or per-module coverage. 
- Unlike KCOV, it does not record per-task coverage. Coverage data can be read from debugfs, and interpreted using the usual gcov tooling.


Configure the kernel with:
```conf
CONFIG_DEBUG_FS=y
CONFIG_GCOV_KERNEL=y
```
and to get coverage data for the entire kernel:

```conf
CONFIG_GCOV_PROFILE_ALL=y
```
Note that kernels compiled with profiling flags will be significantly larger and run slower. Also CONFIG_GCOV_PROFILE_ALL may not be supported on all architectures.

Profiling data will only become accessible once debugfs has been mounted:`mount -t debugfs none /sys/kernel/debug`


For more information on how to use gcov, check [Using gcov with Linux Kernel - Kernel Docs](https://www.kernel.org/doc/html/latest/dev-tools/gcov.html)

### kcov

- KCOV: code coverage for fuzzing is a feature which can be built in to the kernel to allow capturing coverage on a per-task level. 
- It's therefore useful for fuzzing and other situations where information about code executed during, for example, a single syscall is useful.

To enable KCOV, configure the kernel with: `CONFIG_KCOV=y`

To enable comparison operands collection, set: `CONFIG_KCOV_ENABLE_COMPARISONS=y`

Coverage data only becomes accessible once debugfs has been mounted: `mount -t debugfs none /sys/kernel/debug`

For more information on how to use kcov, check [KCOV: code coverage for fuzzing - Kernel Docs](https://www.kernel.org/doc/html/latest/dev-tools/kcov.html)

### References

- [Using gcov with Linux Kernel - Kernel Docs](https://www.kernel.org/doc/html/latest/dev-tools/gcov.html)
- [KCOV: code coverage for fuzzing - Kernel Docs](https://www.kernel.org/doc/html/latest/dev-tools/kcov.html)

## Static Tools

Static analysis tools are able to analyze the source code to find problems without running the program.

Beware, though, that static analysis tools suffer from **false positives**. Errors and warns need to be **evaluated carefully before attempting to fix them**.

### Sparse

- Written by Linus Torvalds and integrated with the Linux source code.
- Designed to find possible coding faults in the kernel.
- Via annotations, the tool performs several semantic validations in the source code.
- Install it using your distro's package manager
- To run it in the kernel use `make C=1` or `make C=2` to see more warnings.

```makefile
...
# Use 'make C=1' to enable checking of only re-compiled files.
# Use 'make C=2' to enable checking of *all* source files, regardless
```

### Smatch

- Smatch extends Sparse and provides additional checks for programming logic mistakes such as missing breaks in switch statements, unused return values on error checking, forgetting to set an error code in the return of an error path, etc. 
- Smatch also has tests against more serious issues such as integer overflows, null pointer dereferences, and memory leaks.
- Install smatch using your distro's package manager.

### Coccinelle

- Coccinelle is often used to aid refactoring and collateral evolution of source code, but it can also help to avoid certain bugs that occur in common code patterns. 
- The types of tests available include API tests, tests for correct usage of kernel iterators, checks for the soundness of free operations, analysis of locking behavior, and further tests known to help keep consistent kernel usage.
- Install Coccinelle using your distro's package manager.

To use Coccinelle in the Linux Kernel, you can just use `make coccicheck`
Options for coccicheck are :

- Mode: The mode to use is specified by setting the MODE variable with MODE=<mode>.
  - `patch` proposes a fix, when possible.
  - `report` generates a list in the following format: file:line:column-column: message
  - `context` highlights lines of interest and their context in a diff-like style. Lines of interest are indicated with `-`.
  - `org` generates a report in the Org mode format of Emacs.
  - `chain` tries the previous modes in the order above until one succeeds.
  - `rep+ctxt` runs successively the report mode and the context mode. It should be used with the C option (described later) which checks the code on a file basis.

- To enable verbose messages set the `V=` variable. Eg : `make coccicheck MODE=report V=1`
- To change the parallelism, set the `J=` variable.
- To apply Coccinelle to a specific directory, `M=` can be used. Eg : `make coccicheck M=drivers/net/wireless/`

For more options, check : [Coccinelle - Kernel Docs](https://www.kernel.org/doc/html/latest/dev-tools/coccinelle.html)

### References

- [Kernel Testing Guide](https://www.kernel.org/doc/html/latest/dev-tools/testing-overview.html#)
- [Sparse - Kernel Docs](https://www.kernel.org/doc/html/v5.5/dev-tools/sparse.html)
- [Sparse - KernelNewbies](https://kernelnewbies.org/Sparse)
- [Coccinelle - Kernel Docs](https://www.kernel.org/doc/html/latest/dev-tools/coccinelle.html)

## Test Automation Tools

### KUnit

- KUnit (KUnit - Linux Kernel Unit Testing) is an entirely in-kernel system for "white box" testing: because test code is part of the kernel, it can access internal structures and functions which aren't exposed to userspace.

- KUnit tests therefore are best written against small, self-contained parts of the kernel, which can be tested in isolation. For example, a KUnit test might test an individual kernel function (or even a single codepath through a function, such as an error handling case), rather than a feature as a whole.

- This also makes KUnit tests very fast to build and run, allowing them to be run frequently as part of the development process.

### kselftest

- kselftest (Linux Kernel Selftests) is largely implemented in userspace, and tests are normal userspace scripts or programs.

- This makes it easier to write more complicated tests, or tests which need to manipulate the overall system state more (e.g., spawning processes, etc.).
  
- It's not possible to call kernel functions directly from kselftest. This means that only kernel functionality which is exposed to userspace somehow (e.g. by a syscall, device, filesystem, etc.) can be tested with kselftest. 
  
- To work around this, some tests include a **companion kernel module** which **exposes more information or functionality**. If a test runs mostly or entirely within the kernel, however, KUnit may be the more appropriate tool.

- kselftest is therefore suited well to tests of whole features, as these will expose an interface to userspace, which can be tested, but not implementation details. This aligns well with 'system' or 'end-to-end' testing.

### Linux Test Project

- The Linux Test Project (LTP) is a project that offers a set of automated tests that validates not only the functionality of the Linux kernel but also the reliability, robustness, and stability of the operating system. 
- The project is developed and maintained by companies like IBM, Cisco, Fujitsu, SUSE and Red Hat. 
- The tests are written in C and shell script, and run directly on the target system.

### Autotest and (Avocado)

- Autotest is a framework for fully automated testing. It is designed primarily to test the Linux kernel, though it is useful for many other functions such as qualifying new hardware. 
- It's an open-source project under the GPL and is used and developed by a number of organizations, including Google, IBM, Red Hat, and many others.
- The focus of the tool is not to implement the tests themselves, but to provide an infrastructure to automate the execution of tests implemented by other projects. For example, Autotest uses the test cases implemented by the Linux Test Project. 
- Avocado is a newer project being built by the same team that built Autotest.  Native tests are written in Python and they follow the `unittest` pattern, but any executable can serve as a test.

### References

- [KUnit - Linux Kernel Unit Testing - Kernel Docs](https://www.kernel.org/doc/html/latest/dev-tools/kunit/index.html)
- [Linux Kernel Selftest](https://www.kernel.org/doc/html/latest/dev-tools/kselftest.html)
- [Linux Test Project - Docs](https://linux-test-project.github.io/)
- [Autotest - Docs](https://autotest.github.io/)
- [Avocado - Docs](https://avocado-framework.github.io/)

## Dynamic Tools

### References

- [A better workflow for kernel debugging - Blog by Ricardo B. Marliere](https://marliere.net/posts/2/)

## Extras

### KernelCI

- KernelCI is a community-based project for testing the Linux kernel. It is a distributed system that runs tests on real hardware and reports the results.
- Provides a dashboard to visualize the results of the tests.
- The project is supported by several companies and organizations, including Linaro, Collabora, Red Hat, Google, and many others.

Homepage : [KernelCI](https://kernelci.org/)