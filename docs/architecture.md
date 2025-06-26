# System level architecture

This document describes all components of BacPack system and how they work together.

## Components

In a nutshell, BacPack system contains these components:

 - Packager
 - Package Repository
 - Package Context
 - Package Tracker
 - Project

The interactions and relationships between these components are shown on next diagram.

```mermaid
---
  config:
    class:
      hideEmptyMembersBox: true
---
classDiagram
    class Packager
    class PackageContext

    namespace Docker {
        class DockerImage
        class DockerContainer
    }

    class PackageRepository
    class Package
    class Project
    class PackageTracker

    %% CORE RELATIONSHIPS

    %% Packager uses PackageContext and PackageRepository (Association)
    Packager --> PackageContext : reads Configs from
    Packager --> PackageRepository : stores Packages in
  
    %% Packager builds DockerImage
    Packager --> DockerImage : builds

    %% PackageRepository aggregates Packages (Aggregation - packages are stored independently)
    PackageRepository o-- "0..*" Package : stores

    %% Docker relationships (Composition - container lifecycle tied to image)
    DockerImage *-- "0..1" DockerContainer : creates

    %% Packager creates packages through build process (Association)
    Packager ..> DockerContainer : uses for building

    %% EXAMPLE USAGE RELATIONSHIPS

    %% Project uses PackageTracker (Composition - project owns its tracker)
    Project *-- PackageTracker : uses

    %% PackageTracker interacts with repository and creates sysroot (Association)
    PackageTracker --> PackageRepository : retrieves Packages from
```

### Main components

#### Packager

Packager is a tool for building Packages and Apps. It takes a Package Context as an input.

Both `build-package` and `build-app` commands build Package or App specified in Package Context
in a Docker container based on existing Docker image built by `build-image` command, create a zip
archive of its files and copy it to Package Repository.

With `create-sysroot` command, Packager creates a sysroot directory from Packages in Package
Repository for given target platform.

```mermaid
---
  config:
    class:
      hideEmptyMembersBox: true
---
classDiagram
    class Packager {
      -context: Context
      -readDockerfiles(context: Context)
      -readAppConfigs(context: Context)
      -readPackageConfigs(context: Context)
      -readAppConfigs(context: Context)
      +buildImage(context: Context, imageName: string)
      +buildPackage(context: Context, packageName: string, options: BuildOptions)
      +buildApp(context: Context, appName: string, options: BuildOptions)
      +createSysroot(context: Context, imageName: string, repository: PackageRepository, sysrootDir: string)
    }
    class PackageContext

    namespace Docker {
        class DockerImage
        class DockerContainer
    }

    class PackageRepository

    %% CORE RELATIONSHIPS

    %% Packager uses PackageContext and PackageRepository (Association)
    Packager --> PackageContext : reads Configs from
    Packager --> PackageRepository : stores Packages in
  
    %% Packager builds DockerImage
    Packager --> DockerImage : builds

    %% Docker relationships (Composition - container lifecycle tied to image)
    DockerImage *-- "0..1" DockerContainer : creates

    %% Packager creates packages through build process (Association)
    Packager ..> DockerContainer : uses for building

    %% Packager creates sysroot
    Packager --> Sysroot : creates and manages
```

#### Package Tracker

Package Tracker provides CMake macros that handle downloading, caching, and integrating
Packages/Apps from Package Repository. Projects link to Package Tracker repository to use Packages
built in Package Repository.

The Package Tracker links to a Package Repository. This link must be changed to work with project
specific Package Repository.

```mermaid
---
  config:
    class:
      hideEmptyMembersBox: true
---
classDiagram
  class PackageTracker {
        -repositoryUrl: string
        -localCachePath: string
        +downloadPackage(packageName: string, version: string): Package
        +setupSysroot(packages: Package[]): Sysroot
        +resolvePackageDependencies(packageName: string): Package[]
  }

  Project --> PackageTracker : uses

  %% PackageTracker interacts with repository and creates sysroot (Association)
  PackageTracker --> PackageRepository : retrieves Packages from
```

### Project specific components

#### Package Repository

Package Repository is a Git repository storage of Packages and Apps, which are built and copied
there by Packager.

```mermaid
---
  config:
    class:
      hideEmptyMembersBox: true
---
classDiagram
    class PackageRepository {
      -gitRepositoryPath: string
      -isLfsEnabled: boolean
    }
    class Package {
      -packageFiles: file[]
    }

    %% CORE RELATIONSHIPS

    %% Packager uses PackageRepository (Association)
    Packager --> PackageRepository : stores Packages in

    %% PackageRepository aggregates Packages (Aggregation - packages are stored independently)
    PackageRepository o-- "0..*" Package : stores

    %% EXAMPLE USAGE RELATIONSHIPS

    %% PackageTracker interacts with repository and creates sysroot (Association)
    PackageTracker --> PackageRepository : retrieves Packages from
```

#### Package Context

Package Context contains definitions of Docker images, Packages and Apps in a [strict directory
structure](https://github.com/bacpack-system/packager/blob/master/doc/ContextStructure.md). The
Packages and Apps must use Docker images defined in the same Package Context.

```mermaid
---
  config:
    class:
      hideEmptyMembersBox: true
---
classDiagram
    namespace PackageContext {
        class Context {
            -contextDirectory: string
        }

        class PackageConfig {
            -env: Map~string, string~
            -dependsOn: string[]
            -gitUri: string
            -gitRevision: string
            -cmakeListDir: string
            -cmakeDefines: Map~string, string~
            -packageName: string
            -versionTag: string
            -platformMode: string
            -isLibrary: boolean
            -isDevLib: boolean
            -isDebug: boolean
            -dockerImageNames: string[]
        }

        class AppConfig {
            -env: Map~string, string~
            -gitUri: string
            -gitRevision: string
            -cmakeListDir: string
            -cmakeDefines: Map~string, string~
            -packageName: string
            -versionTag: string
            -platformMode: string
            -isLibrary: boolean
            -isDevLib: boolean
            -isDebug: boolean
            -dockerImageNames: string[]
        }

        class DockerConfig {
            -imageName: string
            -dockerfilePath: string
        }
    }

    namespace Docker {
        class DockerImage
    }

    %% CORE RELATIONSHIPS

    %% Packager uses Context
    Packager --> Context : reads Configs from

    %% Context aggregates configuration files (Aggregation - Configs exist as files)
    Context o-- "0..*" PackageConfig : contains
    Context o-- "0..*" AppConfig : contains
    Context o-- "1..*" DockerConfig : contains

    %% Docker relationships (Composition - container lifecycle tied to image)
    DockerConfig --> DockerImage : defines

    %% Package dependencies (Dependency)
    PackageConfig ..> PackageConfig : depends on
```

### External tools

#### cmakelib

Dependency tracking library for CMake. It defines macros for dependency tracking and features
caching for efficient use and building of dependencies.
