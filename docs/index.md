# Introduction

This is a documentation for BacPack System stack of BringAuto s.r.o.

## Overview

The BacPack system is created to build and manage dependencies of CMake based projects.

The main projects of BacPack system:

 - [packager](https://github.com/bacpack-system/packager)
 - [package-tracker](https://github.com/bacpack-system/package-tracker)

The system is also using this project:

 - [cmakelib](https://github.com/cmakelib/cmakelib)

Packager and Package Tracker also interacts with other components, which are use case specific and
must be created by user. These components are listed below

 - Package Repository ([BA example](https://gitea.bringauto.com/fleet-protocol/package-repository))
 - Package Context ([BA example](https://github.com/bringauto/packager-fleet-protocol-context))

The location of these components is use case specific, but BringAuto mainly uses given links for
these components.
