# Package Manager C Language Target Support

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-swiftpm-c-language-targets.md)
* Author(s): [Daniel Dunbar](https://github.com/ddunbar)
* Status: **Review**
* Review manager: TBD

## Introduction

This is a proposal for adding initial package manager support for the C, C++,
Objective-C, and Objective-C++ languages (henceforth, simply referred to as "C"
languages). This proposal is limited in scope to only supporting targets
consisting entirely of C languages; there is no provision for supporting targets
which include both C and Swift sources.

## Motivation

Swift has easy interoperability with C based languages through the use of the
Clang modules system. We would like Swift packages to be able to include C
targets which can be exposed to Swift directly as part of a single package.

This gives developers a simple mechanism for "falling back" to C when they need
to access APIs which are inadequately or poorly bridged to Swift, or when they
need to implement behavior which is better done in low-level C than Swift.

## Proposed solution

Our proposed solution extends the convention based system to allow targets
composed only of C sources. The conventions will be amended as follows:

1. Any existing directory which defines a target will be allowed to contain a
   set of C sources, recognized by file extension. If a target contains any C
   sources, it must only contain C sources.

2. C targets *may* have a special subdirectory named `Includes` or `include`
   (the include directory). Only one such name may be used.

   The headers in this directory are presumed to be the "exported" interface of
   the C target, and will be made available for use in other targets.

3. As with Swift targets, we will use the presence of a source file named
   `main.c` (or `main.cpp`, etc.) to indicate an executable versus a library.

The following example layout would define a package containing a C library and a
Swift target:

    example/src/include/foo/foo.h
    example/src/foo/foo.c
    example/src/foo/util.h
    example/src/bar/bar.swift

In this example, the `util.h` would be something internal to the implementation
of the `foo` target, while `include/foo/foo.h` would be the exported API to the
`foo` library.

The package manager will naturally gain support for building these targets:

1. The package manager will (a) construct a synthesized module map including all
   of the exported API headers, for use in Swift, and (b) will provide a header
   search path to this directory to any targets which transitively depend on
   this one.

2. Most packages are encouraged to include the package or target name as a
   subfolder of the include directory, to clarify use. However, this is not
   required and it may be useful for legacy projects whose headers have
   traditionally been installed directly into `/usr/include` to not use this
   convention. This allows client code of those projects to be source compatible
   with versions which use the installed library.

There are several obvious features related to C language support which *are not*
addressed with this proposal:
   
1. We anticipate the need to declare that only particular targets should have
   their API exported to downstream **packages** (for example, the package above
   might want to export the `bar` target to clients, and keep the C target as an
   implementation detail).

2. No provision is made in this proposal for controlling compiler arguments. We
   will support the existing debug and release configurations using a fixed set
   of compiler flags. We expect future proposals to accomodate the need to
   modify those flags.

3. We intend for the feature to be built in such a way as to support any
   standard compliant C compiler, but our emphasis will largely be on supporting
   the use of Clang as that compiler (and of course our modules support will
   require Clang).

## Impact on existing packages

There is no serious impact on existing packages, as this was previously
unsupported. We will begin trying to build C family sources in existing
packages, but this is likely to be desirable.

It is worth considering the impact on existing C language projects which do not
follow the conventions above.

* Most projects will not conform to these conventions. However, this is expected
  of any "simple" convention; we don't think that there is any other
  straightforward convention that would allow a significant percentage of
  existing C language projects to "just work".

  We do anticipate allowing certain overrides to be present in the manifest file
  describing a C target, to allow some projects to work with the package manager
  with only the addition of a correct `Package.swift` file. We will determine
  the exact overrides allowed once we are able to test options against existing
  C projects.

* Existing source code *using* existing projects (e.g., a source file using
  `libpng`) may be able to use well formed packages without modification. This
  is viewed as a significant advantage, as it will potentially help upstream
  projects ingest proper package manager support into their main tree.

## Alternatives considered

We could avoid supporting C language targets directly, and rely on external
build systems and features to integrate them into Swift packages. We may wish to
add such features independently from this proposal, but we think it is
worthwhile to have some native support for C targets. This will make it easy to
integrate small amounts of C code into what are otherwise Swift projects, and is
in line with a long term direction of making the Swift package manager useful
for non-Swift projects.
