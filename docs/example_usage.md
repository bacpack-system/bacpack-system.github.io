# Example usage

This document demonstrates the use cases defined in [Use Cases](./use_cases.md) using an [example
project](https://github.com/bacpack-system/example-project) which uses `curl` dependency to print
a html content of `https://www.example.com` webpage. Below are steps to define a `curl` Package
and include it in example project.

## Define a Package

The example project uses a `curl` as a dependency, which depends on `zlib`. Firstly the definition
files - Configs of these Packages needs to be created inside a Package Context. These Packages are
defined in [this example Package Context](https://github.com/bacpack-system/example-context). The
basic structure of Package Context and Configs is further described below. The complete structure
of Package Context is decribed in
[Packager documentation](https://github.com/bacpack-system/packager/blob/master/doc).

### Structure of Package Context

```plaintext
<context_directory>/
  docker/
    <docker_image_name>/
      Dockerfile
    ...
  package/
    <package_group_name>/
      <package_config_a>.json
      <package_config_b>.json
    ...
    curl/
      curl_release.json
      curl_debug.json
    zlib/
      zlib_release.json
      zlib_debug.json
  app/
    <app_group_name>/
      <app_config_a>.json
      <app_config_b>.json
    ...
```

Each Package is built inside a Docker container specified by Dockerfile. All used Dockerfiles are
defined in a `docker` directory. 

In a `package`/`app` directory, the Package/App definition files - Configs are present. Each
Package can have more variations, that is why there is a concept of Package group in a structure.
The `package_group_name` is the actual name of a Package and in this directory there can be more
variations of this Package. For example, one of the variation can be building with Release or Debug
build types.

### Config structure

The release variation Config of `curl` Package is following:

```json
{
  "Env": {},
  "DependsOn": [
    "zlib"
  ],
  "Git": {
    "URI": "https://github.com/curl/curl.git",
    "Revision": "curl-7_79_1"
  },
  "Build": {
    "CMake": {
      "Defines": {
        "CMAKE_BUILD_TYPE": "Release"
      }
    }
  },
  "Package": {
    "Name": "curl",
    "VersionTag": "v7.79.1",
    "PlatformString": {
      "Mode": "auto"
    },
    "IsLibrary": true,
    "IsDevLib": true,
    "IsDebug": false
  },
  "DockerMatrix": {
    "ImageNames": [
      "ubuntu2404",
      "fedora41"
    ]
  }
}
```

All fields of Config are described in a Packager documentation. Only important
fields for the purpose of this example will be described further. The `DependsOn` field contains
all dependency Packages which must be defined in the same Package Context. The `Git/URI` must be an
uri to a CMake based git repository with tag specified by `Git/Revision`. In `Build/CMake/Defines`
can be specified all CMake options. The `Package/Name` field must be same as name of
`package_group_name`. And all images in `DockerMatrix/ImageNames` must be defined in `docker`
directory of same Package Context.

## Package build

Firstly, the target Docker image must be build. Then if not already created, the Package Repository
must be created. Finally the `curl` Package can be build.

### Build Docker image

If the used image is not built on system, it must be build with `build-image` Packager command.
The command for building `ubuntu2404` image is:

```bash
bap-builder build-image --context /path/to/package/context --image-name ubuntu2404
```

### Create a Package Repository

For the purpose of this example the local Package Repository will be used, but usually the Package
Repository is present upstream and cloned locally.

The creation of Package Repository is basically creating an empty git repository. Following
commands will create it in current directory:

```bash
mkdir package_repository && cd package_repository
git init
```

### Build a Package

The Packager command `build-package` for building Packages requires these parameters:

 - context
 - image-name
 - output-dir

The `context` parameter specifies the Package Context path, the `image-name` selects the target
image for Package and `output-dir` points to the Package Repository.

The command for building `curl` Package is:

```bash
bap-builder build-package --context /path/to/package/context --image-name ubuntu2404 --output-dir /path/to/package/repository --name curl --build-deps
```

This command builds a Package `curl` defined in Context for `ubuntu2404` image, creates an archive
of this Package and copies it to the output-dir (Package Repository). The command with
`--build-deps` flag also builds all dependencies of given Package. In this case it also builds the
`bzip` Package. Other flags and settings of Packager are described in its
[documentation](https://github.com/bacpack-system/packager/tree/master/doc).

## Add Package to a project build

The `curl` Package can be now used for building
[example project](https://github.com/bacpack-system/example-project).

### Install cmakelib

Firstly the cmakelib (link in [Introduction](./index.md)) must be installed. Follow the README
to install it.

### Set the Package Tracker

In the project root directory the `CMLibStorage.cmake` needs to be added with following content:

```cmake
SET(STORAGE_LIST DEP)
SET(STORAGE_LIST_DEP "https://github.com/bacpack-system/package-tracker.git")
```

This will links the project with Package Tracker specific for BringAuto, which enables adding
Packages from BringAuto's specific Package Repository (The Package Tracker points to Package
Repository). Usually the previously built Packages would be used, but for simplicity, the same
Packages from BringAuto's specific Package Repository are used instead. These Packages are exactly
the same as Packages defined in [example Package Context](https://github.com/bacpack-system/example-context).

!!! info

    The local Package Repository can't be easily used, because currently the BacPack system does
    not support it. That is why the upstream Package Repository is used. The usage of local Package
    Repository will be added in future releases.

### Configure CMakeLists

The dependencies must be defined in `cmake/Dependencies.cmake` like this:

```cmake
BA_PACKAGE_LIBRARY(curl v7.79.1)
BA_PACKAGE_LIBRARY(zlib v1.2.11 OUTPUT_PATH_VAR ZLIB_ROOT)
```

This file must be included in project's CMakeLists.txt and then the Package can be included
with `FIND_PACKAGE`. But first the cmakelib with required components must be added.
Example CMake code:

```cmake
# Including 'cmake/Dependencies.cmake' file
INCLUDE(cmake/Dependencies.cmake)

# Adding cmakelib with components
FIND_PACKAGE(CMLIB COMPONENTS CMDEF CMUTIL STORAGE REQUIRED)

# Adding bzip and curl Package
FIND_PACKAGE(BZIP REQUIRED)
FIND_PACKAGE(CURL REQUIRED)
```

!!! note

    Each of the cmakelib components has its own git repository and adds specific functionality.
    Any or none of following components can be used:

     - [CMDEF](https://github.com/cmakelib/cmakelib-component-cmdef) - adds wrappers for basic CMake features
     - [CMUTIL](https://github.com/cmakelib/cmakelib-component-cmutil) - Provides functionality for other cmakelib components
     - [STORAGE](https://github.com/cmakelib/cmakelib-component-storage) - mechanism for storing and retrieving build dependencies

### Build a project

At this moment, the project can be build with `cmake` the usual way.