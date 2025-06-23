# Introduction

This documentation covers the BacPack System, a component stack developed by BringAuto s.r.o.
This system provides a set of components designed to create an environment for easy dependency and
package management of CMake based projects.

## Main components

The main components of BacPack system:

 - [packager](https://github.com/bacpack-system/packager)
 - [package-tracker](https://github.com/bacpack-system/package-tracker)

## Project specific components

Packager and Package Tracker also interact with other components, which are use case specific and
must be created for each project. These components are listed below

 - Package Repository ([BA example](https://gitea.bringauto.com/fleet-protocol/package-repository))
 - Package Context ([BA example](https://github.com/bringauto/packager-fleet-protocol-context))

The location of these components is use case specific, but BringAuto mainly uses given links for
these components.

## External tools

 - docker
 - [cmakelib](https://github.com/cmakelib/cmakelib)
