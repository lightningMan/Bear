Build EAR
=========

Bear is a tool that generates a compilation database for clang tooling.

The [JSON compilation database][JSONCDB] is used in the clang project
to provide information on how a single compilation unit is processed.
With this, it is easy to re-run the compilation with alternate
programs.

One way to get a compilation database is to use `cmake` as the build
tool.  Passing `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` to cmake generates
the `compile_commands.json` file into the current directory.

For non-cmake projects, Bear generates the JSON file during the build process.

The concept behind Bear is: to execute the original build command and
intercept the `exec` calls issued by the build tool. To achieve that,
Bear uses the `LD_PRELOAD` or `DYLD_INSERT_LIBRARIES` mechanisms provided
by the dynamic linker.

Bear has two components: the library and the binary. The library
redefines the `exec` methods to be used by all child processes. The
executable enables the use of the library for child processes and
writes the output file.

  [JSONCDB]: http://clang.llvm.org/docs/JSONCompilationDatabase.html


How to build
------------

Bear should be quite portable on UNIX operating systems. It has been
tested on FreeBSD, GNU/Linux and OS X.

### Prerequisites

1. an ANSI **C compiler**, to compile the sources.
2. **cmake**, to configure the build process.
3. **make**, to run the build.  The makefiles are generated by `cmake`.
4. **python** is a runtime dependency. The `bear` command is written in
   Python. (version >= 2.7)
5. **lit** is an optional dependency to run the functional tests

### Build commands

Ideally, you should build Bear in a separate build directory.

    cmake $BEAR_SOURCE_DIR
    make all
    make install # to install
    make check   # to run tests
    make package # to make packages

You can configure the build process with passing arguments to cmake.

How to use
----------

After installation the usage is like this:

    bear make

The output file called `compile_commands.json` found  in current directory.

For more options you can check the man page or pass `--help` parameter.

Known issues
------------

### Environment overriding caused problems

Because Bear uses `LD_PRELOAD` or `DYLD_INSERT_LIBRARIES` environment variables,
it does not append to it, but overrides it. So builds which are using these
variables might not work. (I don't know any build tool which does that, but
please let me know if you do.)

### Empty compilation database on OS X Captain or Fedora

Security extension/modes on different operating systems might disable library
preloads. This case Bear behaves normally, but the result compilation database
will be empty. (Please make sure it's not the case when reporting bugs.)
Notable examples for enabled security modes are: OS X 10.11 (check with
`csrutil status | grep 'System Integrity Protection'`), and Fedora, CentOS, RHEL
(check with `sestatus | grep 'SELinux status'`).

Workaround could be to disable the security feature while running Bear. (This
might involve reboot of your computer, so might be heavy workaround.) The other
option could be to use tools which are using compiler wrappers. (It injects a
fake compiler which does record the compiler invocation and calls the real
compiler too.) An example for such tool might be [scan-build][scanbuild]. The
build system shall respect `CC` and `CXX` environment variables.

  [scanbuild]: https://github.com/rizsotto/scan-build

Problem reports
---------------

If you find a bug in this documentation or elsewhere in the program or would
like to propose an improvement, please use the project's [github issue
tracker][ISSUES]. Please describing the bug and where you found it. If you
have a suggestion how to fix it, include that as well. Patches are also
welcome.

  [ISSUES]: https://github.com/rizsotto/Bear/issues
