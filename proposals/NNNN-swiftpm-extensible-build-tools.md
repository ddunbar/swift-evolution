# Package Manager Extensible Build Tools

* Proposal: [SE-NNNN](https://github.com/ddunbar/swift-evolution/blob/master/proposals/NNNN-swiftpm-extensible-build-tools.md)
* Author(s): [Daniel Dunbar](https://github.com/ddunbar)
* Status: **WIP**
* Review manager: **N/A**

**This is a WORK IN PROGRESS.** *It is currently just intended to be used for discussion.*

## Introduction

This is a proposal for adding package manager support for extensible build
tools, i.e. executing tools at *build* time which were themselves produced by
some other package, and whose behavior the package manager does not understood
directly, but rather interacts through a well-defined, Swift, protocol.

We expect this behavior to greatly enhance our capacity for building complex
software packages.

## Motivation

There are large bodies of existing, complex, software projects which cannot be
described directly in SwiftPM, but which other projects wish to depend upon.

The package manager currently supports two mechanisms by which non-package
manager projects can be used in other packages:

* The system module map feature allows defining support for a package which
  already exists in an installed form on the system. In conjunction with system
  package managers, this can be used to configure an environment in which a
  package can depend on such a body of software.

* The C language targets feature allows the package manager to build and include
  C family targets. This can provide similar features to the previous bullet,
  but through use of the standard C compiler features (like header and library
  search paths). The external project again needs to be installed using a
  different package manager.

These mechanisms have two major downsides:

1. The usability of a package depending upon them requires interaction with a
   system package manager. This increases developer friction.

2. The system packages are global. This means project builds are no longer self
   contained. That introduces greater complexity when managing the dependencies
   of multiple projects which need to run in the same environment, but may have
   conflicting dependencies. This is compounded by the package manager itself
   not understanding much about the system dependencies.

The package manager also currently has no support for running third-party tools
during the build process. For example, the (Swift protobuf
compiler)[https://github.com/apple/swift-protobuf] is used to generate Swift
code automatically for models defined in ancilliary sources. Currently there is
no way for a package to directly cause this compiler to be run during the build,
rather the user must manually invoke the compiler before invoking the package
manager.

## Proposed solution



## Detailed design

**TBD**

## Alternatives considered

We considered allowing a more straight-forward capability for the package
manager to simply run "shell scripts" (essentially arbitrary invoke command
lines) at points during the build. We rejected this approach because:

1. Even this approach requires us to either explicitly or implicit document and
   commit to supporting a specific file system layout for the build artifacts
   (so that they scripts can interact with them). Adding support in this way
   makes it hard for script authors to know what is explicitly officially
   supported and what simply happens to work due to the current implementation
   details of the tool. That in turn could make it hard to evolve the tool if we
   wanted to change a behavior which numerous scripts had grown to depend on.

2. It is hard for us to enforce that scripts don't do things that are
   unsupported, since the script by design is intended to interact directly with
   the file system. This has similar problems as #1 and makes it harder for
   package authors to write "correct" packages.


Another alternative is to do nothing, and requiring all behaviors be explicitly
supported through some well-modeled behavior defined in the package
manifest. While we aim to support as many common behaviors as possible, we also
want to support as many complex behaviors as possible and recognize that we need
an extensible mechanism to support "special cases". Our intention is that even
once we add extensible build tools, that we will continue to add explicit
manifest support for behaviors that we see become common in the ecosystem, in
order to keep packages simple to understand and author.



Although it is possible to the more straightforward "shell script" capability is
simpler to implement and could be added to the existing package manager without
significant implementation work, we have felt that this would ultimately be more
likely to harm than help the ecosystem. We have several reasons for believing
this:

1. We know the package manager is currently missing critical features which
   would be needed by many packages. One of our design tenants has been that we
   should design so that roughly 80% of packages can be written with a
   _straightforward_, _simple_, and _clean_ manifest that does not require
   advanced features. If we were to add a straightforward, but complex, script
   based extension mechanism, we expect that far too many packages would begin
   to take advantage of it due to these missing features. Due to the opaque
   nature of shell-script extensions, this would be very hard to then migrate
   past once we did gain the appropriate features, because the tools would be in
   a poor position to understand what the shell script did.

2. The package manager currently always builds individual packages into discrete
   sandboxes, including the transitive closure of the package
   dependencies. While this works well for sandboxing effects, it is inherently
   not scalable when many separate packages are being worked on by the same
   developer. This approach also makes it hard for continuous integration
   systems which need to perform very reliable tests on many different packages.

   Our intention is to solve these problems by leveraging
   [reproducible build](https://reproducible-builds.org) techniques to expose
   the same user interface as we do today, but transparently cache shared build
   artifacts under the hood. This will rely on the ability of the package
   manager to have perfect knowledge of exactly what content is used by a
   particular part of a build. Shell script based hook mechanisms make this very
   difficult, since (a) by their nature they pull in a large number of
   dependencies (the shell, the tools used in the shell script, etc.), and (b)
   it is hard for the tool to reason about them.
