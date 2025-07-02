# Introduction

This documentation covers the BacPack System, a component stack developed by BringAuto s.r.o.
This system provides a set of Components designed to create an environment for easy dependency and
package management of CMake based projects.

The documentation uses terms defined in [Term definition](./term_definition.md).

## Main Components

The main Components of BacPack system:

 - [**Packager**](https://github.com/bacpack-system/packager) - CLI tool that tracks dependencies and
 builds Packages

 - [**Package Tracker**](https://github.com/bacpack-system/package-tracker) - tool that downloads
 previously built dependencies for use in CMake based projects

## Project specific Components

To use BacPack system, these additional Components must be set up that are specific to each Project
or organization:

 - **Package Repository** - A Git repository where built Packages are stored and distributed

 - **Package Context** - A directory structure containing configuration files that define how to
 build Packages, including build settings and Docker environments.

These Components are customized for each Project's specific needs - different Projects will have
different Packages to build and different build requirements.

## External tools

 - [docker](https://www.docker.com/)
 - [cmakelib](https://github.com/cmakelib/cmakelib)
