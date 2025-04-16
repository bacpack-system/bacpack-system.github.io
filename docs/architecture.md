# System level architecture

This document describes all components of BacPack system and how they work together.

## Components

In a nutshell, BacPack system contains these components:

 - Packager
 - Package repository
 - Package context
 - Package tracker
 - target project

### Packager

Packager is a tool for building Docker Images, Packages and Apps. It takes a Package context as an
input.

The `build-image` command of Packager builds Docker Image defined in Package context.

Both `build-package` and `build-app` commands builds Package or App specified in Package context
in a Docker container based on existing Docker image (built by `build-image` command), make an
zip archive of its file and copies it to Package repository.

### Package repository

Package repository is a Git repository of Packages and Apps, which are built and copied there by
Packager.

### Package context

Package context contains definition of Docker images, Packages and Apps in a strict directory
structure (this structure is defined in Packager documentation). The Packages and Apps must use
Docker images defined in the same Package context.

### Package tracker

Package tracker defines CMake macros for downloading, caching and populating Packages/Apps from
Package repository. The target application has a link to Package tracker repo to use Packages built
in Package repository.