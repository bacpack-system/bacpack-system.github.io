# Introduction

This is a documentation for BacPack System stack of BringAuto s.r.o.

## Overview

The BacPack system is created to build and manage dependencies of CMake based projects.

The main projects of BacPack system:

 - packager ([https://github.com/bacpack-system/packager](https://github.com/bacpack-system/packager))
 - package-tracker ([https://github.com/bacpack-system/package-tracker](https://github.com/bacpack-system/package-tracker))

The system is also using this project:

 - cmakelib ([https://github.com/cmakelib/cmakelib](https://github.com/cmakelib/cmakelib))

The system also uses following components, which are just Git repositories with defined structure:

 - Package Repository ([https://gitea.bringauto.com/fleet-protocol/package-repository](https://gitea.bringauto.com/fleet-protocol/package-repository))
 - Package Context ([https://github.com/bringauto/packager-fleet-protocol-context](https://github.com/bringauto/packager-fleet-protocol-context))

The location of these components is use case specific, but BringAuto mainly uses given links for
these components.
