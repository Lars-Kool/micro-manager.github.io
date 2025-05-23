---
autogenerated: true
title: Building MM on Windows
redirect_from: /wiki/Building_MM_on_Windows
layout: page
---

## Before you start

First make sure you have enough hard disk space. 15 GB is probably
enough.

## Setting up the required tools

### Git and Subversion clients

- [Git for Windows](https://git-scm.com/download/win)
- A [Subversion client](http://subversion.apache.org/packages.html#windows),
  such as [SlikSVN](https://sliksvn.com/download/).
  GUI clients such as [TortoiseSVN](http://tortoisesvn.net/) also work well.

### Microsoft Visual Studio

You need to install the following:

- Visual Studio Community 2022, with the `Desktop development with C++`
  workload, plus
  - `MSVC v142 - VS 2019 C++ x64/x86 build tools`,
  - `Windows 10 SDK 10.0.19041`, and
  - `Windows 10 SDK 10.0.20348`

The checkbox for the MSVC v142 build tools and the Windows 10 SDKs is located
under `Desktop development with C++` in the right sidebar titled "Installation
details".

(Both versions of the Windows 10 SDK are actually needed for all parts of the
build to work correctly.)

### Java Development Kit (JDK) 8

Micro-Manager currently uses Java 8.
You can also build with a later JDK version, but your code must
compile with Java 8 if you are contributing to our codebase.

You can use either Adoptium [Temurin](https://adoptium.net/index.html?variant=openjdk8&jvmVariant=hotspot)
or Azul [Zulu](https://www.azul.com/downloads/?version=java-8-lts&os=windows&architecture=x86-64-bit&package=jdk).

You do not need the JDK if you are only building device adapters or
other C++ components.

### Apache Ant

[Apache Ant](http://ant.apache.org/) is the build tool used to automate
the full Windows build of Micro-Manager. Version 1.8.1 or later is
required (tested with 1.10.12).

Download the [binary package](http://ant.apache.org/bindownload.cgi).

You do not need Ant if you are only building device adapters of other
C++ components.

### Environment variables

For building the Java and Clojure components using Ant, the following
environment variables need to be set correctly. They can be added in
Control Panel &gt; System and Security &gt; System &gt; "Advanced system
settings" &gt; "Environment variables...".

-   `JAVA_HOME`: set to the path to your JDK installation, e.g.
    `C:\Program Files\Eclipse Adoptium\jdk-8.0.312.7-hotspot\`.
    Do not add quotes to the value, even if it contains spaces.
-   `ANT_HOME`: set to the path to your Ant installation (whereever you
    placed the extracted binary package), e.g. `C:\apache-ant-1.10.12`.
    Do not add quotes to the value, even if it contains spaces.
-   `PATH`: It is convenient to add Ant to the command search path. You
    can append `;%ANT_HOME%\bin` to the end of the System `PATH`
    varialbe, or add a User variable named `PATH` and set its value to
    `%PATH%;%ANT_HOME%\bin`.

## Preparing the source code

### Directory layout

Create a directory named `projects` (the name is not important, but we
will refer to this name below). The `projects` directory may reside
anywhere you like (but paths containing spaces and special characters
should be avoided) and will contain the subdirectories `micro-manager`,
`3rdpartypublic`, and (optionally) `3rdparty`.

All directories mentioned below that don't start with `project` are
relative to `projects\micro-manager`.

### Obtaining the source code

Checkout the [Micro-Manager Source
Code](Micro-Manager_Source_Code) inside the `projects`
directory, using `git` and `svn`.
This should result in the following directories:

```
projects\micro-manager
projects\micro-manager\mmCoreAndDevices
projects\3rdpartypublic
```

### Vendor SDKs and libraries

If you have third-party libraries (e.g. from device vendors), create the
directory `projects\3rdparty` and place them in an appropriately named
subdirectory.

If you are building a device adapter included in the Micro-Manager
Subversion repository, and it requires third-party libraries, you will
need to place them in the correct directory within `projects\3rdparty`,
as determined by the settings in the Visual Studio project file for the
adapter. Alternatively, you can locally edit the build settings in the
project file.

(There are cases where the location of a header file is hard-coded
(relative to the `3rdparty` directory) in the source (.h and .cpp)
files. This practice is discouraged, but if you encounter it, you might
have to edit the path.)

## Building Micro-Manager

### Building the C++ components

Open `micromanager.sln` (in the root of `mmCoreAndDevices`). Visual
Studio will warn you that some projects are missing (they are the
closed-source device adapters), but you can ignore that message. Choose
the correct configuration (try Release if Debug fails).
Then simply right-click the desired projects and select Build.

DeviceAdapters that depend on code in the 3rdparty directory will only
build if you download the correct libraries yourself and place them in
your own copy of the 3rdparty directory. For legal reasons, we are not
allowed to make these libraries public. In general, however, you may
simply ignore DeviceAdapters that do not build because you do not have
the correct dependencies.

Note that the built binaries are placed under the directory `build` at
the root of `mmCoreAndDevices`.

If you build MMCoreJ, `MMCoreJ_wrap.dll` will be generated.
Additionally, Java source files are generated in
`build\intermediates\Swig\MMCoreJ\`. These files are compiled into
`MMCoreJ.jar` in a separate step by Ant (see below).

You can also build all available projects by right-clicking the Solution
(at the very top of the Solution Explorer) and choosing Build.

### Building single device adapters

The easiest method to build a device adapter is to use
`micromanager.sln` as described above. However, you may want to be able
to work on a single device adapter without seeing all the other projects
(which also slow down Visual Studio). You can do so by creating your own
solution file (a solution (`.sln`) file is essentially a container for
multiple projects).

Device adapters need to be linked to the MMDevice static library (which
itself needs to be built). This is done automatically within
`micromanager.sln`. When building device adapters outside of
`micromanager.sln`, you need to include the required MMDevice project in
your solution (even though the required settings are already contained
in the device adapter project file (`.vcxproj`)). To do so:

Open the device adapter project file (e.g.
`DeviceAdapters\DemoCamera\DemoCamera.vcxproj`) in Visual Studio.

Right-click on the Solution, and select Add &gt; Existing Project...
(or, equivalently, select File &gt; Add &gt; Existing Project...).
Choose `MMDevice\MMDevice-SharedRuntime.vcxproj`.

At this point, close the solution file (File &gt; Close Solution, or
just quit Visual Studio). It will prompt you to save a solution file
(.sln). Save it in the default location (next to the device adapter
project). Then, reopen either the device adapter project or the newly
saved solution (either will have the same result). (This extra step is
necessary because Visual Studio does not correctly update its internal
state when you add projects that theoretically know their
interdependencies.)

Now select the Debug or Release configuration from the drop-down menus
(for various reasons, the Debug
configuration does not work for all device adapters). If everything is
configured correctly as described above, you should now be able to
simply select Build.

If you get an LNK1104 error involving `MMDevice-StaticRuntime.lib` (as
opposed to `MMDevice-SharedRuntime`), you are probably building one of
the few device adapters that require this special version of the
MMDevice library. Remove the MMDevice-SharedRuntime project from the
solution, and add `MMDevice\MMDevice-StaticRuntime.vcxproj` instead.
Reload the solution file, and you should be ready to build.

In Visual Studio 2010 and later, the solution is little more than a list
of project files to be built together, so our policy is not to include
solution files for each project in the repository (but see below about
the main `micromanager.sln` file).

The build binaries are placed under the directory `build`, within the
device adapter directory (actually, relative to where you saved the
solution file).

Some device adapters (those made up of more than one project) may not
build correctly by this method. In that case, build it from
`micromanager.sln` as described above.

### Building individual Java components

On Windows, Micro-Manager's Java components are built using Apache Ant.
Before running the Ant targets described below, you must first run the
following command, which downloads some Java libraries required by
Micro-Manager:

```
 ant -f buildscripts/fetchdeps.xml
 ant build-buildtools
```

Each Java (and Clojure) component has a `build.xml` (an Ant build
script):

```
 MMCoreJ_wrap\build.xml
 mmstudio\build.xml
 acqEngine\build.xml
 librariers\build.xml
 plugins\*\build.xml
 autofocus\build.xml
```

See below about the required order of building.

To build one of these components, open the Command Prompt, and change to
the directory containing the build script. Then type

```
 ant jar
```

This should produce the jar file in `build\Java`. (Ant automatically
uses the `build.xml` file in the current directory.)

For example,

```
 cd MMCoreJ_wrap
 ant jar
```

will produce `build\Java\MMCoreJ.jar`.

Some of the Java components depend on each other. When building the
components one by one, you will need to build them in the order listed
above (see below for building everything automatically).

To build `MMCoreJ.jar` (and, by transitive dependency, any of the other
Java components), you need to have built the C++ part of MMCoreJ (either
in Visual Studio as described above, or using Ant as described below)
before running Ant.

Building Clojure components (acqEngine and some of the plugins) requires
a loadable `MMCoreJ_wrap.dll`.

It is possible for the build to drop into an inconsistent state when the
source code is updated. The following might fix it (at the root of the
source tree):

```
 ant clean-all
 rmdir /s /q dependencies
```

Afterwards you will need to go back to the start of this section and
re-run `ant -f buildscripts/fetchdeps.xml`

### Building all components

The master Ant build script (`build.xml` at the root of the source tree)
can be used to build the whole application. It's usage differs from the
individual build scripts for each Java component.

Open the Command Prompt, and change to the `projects\micromanager`
directory. Type

```
 ant -p
```

This should list a number of options.

To build all C++ components (similar to building the `micromanager.sln`
Solution in Visual Studio), type

```
 ant build-cpp -Dmm.build.failonerror=false
```

Setting the `mm.build.failonerror` property to `false` allows the build
to continue even if some of the projects (e.g. device adapters that
depend on libraries you don't have) fail to build. (But you will want to
check that the ones you want did build. Look in the `build` directory.)

The command

```
 ant build -Dmm.build.failonerror=false
```

will build everything (all C++, Java, and Clojure compoenets).

There is also a `build-java` target that you can use to build just the
Java and Clojure components (it will not try to build the C++
components).

You can then type

```
 ant stage-only
```

This will construct a near-complete Micro-Manager installation in the
`stage\Release\x64` directory. The only thing lacking is the Java
Runtime; you will need to copy over the `jre` folder from inside your
JDK installation before launching `ImageJ.exe`. (Note that the
`ImageJ.cfg` file determines where the JRE is searched for.)

To recap, the steps to build everything available with the default
architecture are:

```
ant -f buildscripts\fetchdeps.xml
ant build -Dmm.build.failonerror=false
ant stage-only
(then copy over jre folder)
```

The `stage`, `run` and `package` targets are not yet fully implemented
for general use (they expect files to be present at specific paths).
