# Introduction

This documentation covers the BacPack System, a component stack developed by BringAuto s.r.o.
This system provides a set of components designed to create an environment for easy dependency and
package management of CMake based projects.

## Main components

The main components of BacPack system:

 - [**Packager**](https://github.com/bacpack-system/packager) - CLI tool that tracks dependencies and
 builds Packages

 - [**Package Tracker**](https://github.com/bacpack-system/package-tracker) - tool that downloads
 previously built dependencies for use in CMake based projects

## Project specific components

To use BacPack system, these additional components must be set up that are specific to each project
or organization:

 - **Package Repository** - A Git repository where built Packages are stored and distributed

 - **Package Context** - A directory structure containing configuration files that define how to
 build Packages, including build settings and Docker environments.

These components are customized for each project's specific needs - different projects will have
different Packages to build and different build requirements.

## External tools

 - docker
 - [cmakelib](https://github.com/cmakelib/cmakelib)
