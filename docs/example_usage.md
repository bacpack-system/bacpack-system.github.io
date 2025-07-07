# Example usage

The goal of this document is to demonstrate some of the use cases defined in
[Use Cases](./use_cases.md). During this tutorial an example CMake based project will be created.
This Project will use a dependency - another CMake based project. The dependency will be built and
added to the Project using BacPack system. Finally the newly created Project will be added to
BacPack system as an App and built.

The [example Project](https://github.com/bacpack-system/example-project) will use `curl` dependency
for simply printing a html content of `https://www.example.com` webpage. In the following steps,
the `curl` Package will be defined and built. Then the example Project will be created and
configured to use the `curl` Package.

## Install dependencies

Install the required dependencies for using BacPack system - [docker](https://www.docker.com/) and
[cmakelib](https://github.com/cmakelib/cmakelib).

## Create a Package Context

The Package Context is a structure that contains all necessary information for building Packages
and Apps. It contains definitions of Docker images, Packages and Apps in a [strict directory
structure](https://github.com/bacpack-system/packager/blob/master/doc/ContextStructure.md). The
Packages and Apps must use only Docker images defined in the same Package Context.

The Package Context has following mandatory subdirectories:

 - package - contains Package definitions
 - app - contains App definitions
 - docker - contains Docker image definitions - Dockerfiles

To track changes in Package Context, it is recommended to create a git repository for it. The
command for creating a Package Context as a git repository is:

```bash
mkdir context && cd context && mkdir docker package app && git init
```

The [example Package Context](https://github.com/bacpack-system/example-context)
contains complete Package Context for this tutorial with all `curl` and `zlib` Configs,
`example-project` app and `ubuntu2404` and `fedora41` Dockerfiles.

??? example "Package Context directory structure"

    The following text shows the directory structure of the Package Context after all steps in
    this tutorial. The `package`, `app` and `docker` directories are mandatory. The
    `<package_group_name>` and `<app_group_name>` refers to Package group described in next
    section. The `...` means that the directory can contain more files, but they are not relevant
    for the purpose of this example. The structure also shows the right place for `curl` and
    `zlib` Packages.

    ```plaintext
    <context_directory>
    ├── docker
    │   ├── <docker_image_name>
    │   │   └── Dockerfile
    │   ├── ubuntu2404
    │   │   └── Dockerfile
    │   └── fedora41
    │       └── Dockerfile
    │       ...
    ├── app
    │   ├── <app_group_name>
    │   │   ├── app_config_a.json
    │   │   └── app_config_b.json
    │   └── example-project
    │       ├── example_project_debug.json
    │       └── example_project_release.json
    │       ...
    └── package
        ├── <package_group_name>
        │   ├── package_config_a.json
        │   └── package_config_b.json
        ├── curl
        │   ├── curl_debug.json
        │   └── curl_release.json
        └── zlib
            ├── zlib_debug.json
            └── zlib_release.json
            ...
    ```

## Define a Package

The example Project uses `curl` as a dependency, which depends on `zlib`. The definition files
of these Packages (Configs) need to be created inside a Package Context.

BacPack system supports building Packages for multiple target platforms. For this the Docker images
are used. So in order to build the Packages, the Docker image (build environment) must be defined
and created.

The following sections describe how to define Docker images and Packages.

### Define a Docker image

The Packages are built inside a Docker container created from image specified by Dockerfile.
The image defines a build environment for building Packages. Defined Docker image must comply
with some [requirements](https://github.com/bacpack-system/packager/blob/master/doc/DockerContainerRequiremetns.md),
briefly:

 - CMake must be installed,
 - SSH must be configured with root login and password `1234`,
 - and `uname` must be installed.

Additionally all required tools for building supported Packages must be installed (for example
compilers).

All defined Docker images must be placed in the `docker` directory of the Package Context. In this
directory, each Docker image is defined in its own directory. The name of this directory is the
name of the Docker image (see [Create a Package Context](#create-a-package-context)). This name is
used in the Package Configs to specify the Docker image to use for building the Package. The
Dockerfile must be named `Dockerfile`.

??? example "Dockerfile example"
    The following Dockerfile is used for building `curl` and `zlib` Packages for Fedora 41. It
    installs all required tools and fulfills all
    [requirements](https://github.com/bacpack-system/packager/blob/master/doc/DockerContainerRequiremetns.md).
    The path to this Dockerfile is `context/docker/fedora41/Dockerfile`. The `fedora41` is the
    name of the Docker image and is used in the Package Configs.

    ```docker
    FROM fedora:41

    USER root
    RUN echo root:1234 | chpasswd

    RUN dnf -y update && \
        dnf -y install  \
        automake binutils gcc gcc-c++ git kernel-devel lsb-release make openssh-server openssl-devel patchelf wget \
        && dnf clean all

    RUN wget "https://github.com/Kitware/CMake/releases/download/v3.30.3/cmake-3.30.3-linux-x86_64.sh" -O cmake.sh && \
        chmod +x cmake.sh && \
        ./cmake.sh --skip-license --prefix=/usr/local && \
        rm ./cmake.sh

    RUN dnf -y update && \
        dnf -y remove \
          wget

    RUN git clone https://github.com/cmakelib/cmakelib.git /cmakelib
    RUN echo "export CMLIB_DIR=/cmakelib" >> /root/.bashrc

    RUN sed -ri 's/#?PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
    RUN mkdir -p /run/sshd

    RUN ssh-keygen -A

    ENTRYPOINT ["/usr/sbin/sshd", "-D", "-o", "ListenAddress=0.0.0.0"]

    ```

### Define a Package Config

Package definition - Config is a JSON file that defines the Package. The Config contains all
necessary information for building the Package - the source code repository, the Docker image to
use, the CMake options, etc. The Config structure is described in
[Packager documentation](https://github.com/bacpack-system/packager/blob/master/doc/ConfigStructure.md).

Some of the important fields of Config are:

 - `DependsOn` - list of dependency Packages, all Packages in the list must be defined in the same Package Context
 - `Git/URI` - URI to a CMake based git repository with source code of the Package
 - `Git/Revision` - tag or branch to use for build
 - `Build/CMake/Defines` - CMake options
 - `Package/Name` - name of the Package
 - `DockerMatrix/ImageNames` - list of Docker images to build the Package for

Each Package can have multiple variations, which are defined by different Configs. The variations
can be for example different build types (Release, Debug), different versions, different target
architectures (x86, x64), etc. All variations are part of a so-called Package group. The Package
group is a directory containing all variations of the Package. The name of the Package group must
be equal to the name of the Package (`Package/Name` field in Config). The name of the Config file
is the name of the variation.

There are also 2 types of Configs - Package Configs and App Configs. The only difference between
Packages and Apps is that Apps do not support dependencies managed by the BacPack system. Therefore,
App Configs do not have a `DependsOn` field. All other fields are the same for both Package and App
Configs.

!!! info
    Packages can depend only on different Packages in the same Package Context, but not on Apps.
    In general, Apps can't form any dependency relationships.
    
    The dependency relationships of Packages cannot form a cycle.

??? example "`curl` Config example"
    The following Config defines `curl` Package. It defines its dependency on `zlib` Package,
    the source code repository, name of the Package, the CMake options and the Docker images to
    use. It sets the build type to Release and `IsDebug` to false, so it is a release variation of
    `curl` Package. The path to this Config is `context/package/curl/curl_release.json`.

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

??? example "`zlib` Config example"
    The following Config defines release variation of `zlib` Package. The path to this Config is
    `context/package/zlib/zlib_release.json`.

    ```json
    {
      "Env": {},
      "Git": {
        "URI": "https://github.com/madler/zlib.git",
        "Revision": "v1.2.11"
      },
      "Build": {
        "CMake": {
          "Defines": {
            "CMAKE_BUILD_TYPE": "Release"
          }
        }
      },
      "Package": {
        "Name": "zlib",
        "VersionTag": "v1.2.11",
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

## Package build

Firstly, the target Docker image must be build. Then if not already created, the Package Repository
must be created. Finally the `curl` Package can be build.

### Install Packager

Packager is a tool for building Packages and Apps. It is available as a
[binary release for Linux](https://github.com/bacpack-system/packager/releases) or it can be built
from source. The build instructions are in the
[Packager repository](https://github.com/bacpack-system/packager). The name of the built binary is
`bap-builder`, which is used in next sections.

### Build Docker image

If the used image is not built on the system, it must be built with the `build-image` Packager command.
Following command builds a Docker image based on Dockerfile in given Package Context.

```bash
bap-builder build-image \
            --context context \
            --image-name fedora41
```

### Create a Package Repository

A Package Repository is a storage for Packages built by Packager. It must be a git repository.

For the purpose of this example, a local Package Repository will be used, but usually the Package
Repository is present upstream and cloned locally.

The creation of Package Repository is basically creating an empty git repository. The following
commands will create it in the current directory:

```bash
mkdir package_repo && cd package_repo && git init
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
bap-builder build-package \
            --context context \
            --image-name fedora41 \
            --output-dir package_repo \
            --name curl --build-deps
```

This command builds a Package `curl` defined in Context for `fedora41` image, creates an archive
of this Package and copies it to the output-dir (Package Repository). The command with
`--build-deps` flag also builds all dependencies of the given Package. In this case it also builds
the `zlib` Package. Other flags and settings of Packager are described in its
[documentation](https://github.com/bacpack-system/packager/tree/master/doc).

The `curl` Package can now be used as a dependency for projects.

## Add Package to a Project

The [example Project](https://github.com/bacpack-system/example-project) will be built in the
following steps. The Project uses `curl` Package built in previous steps as a dependency.

### Install cmakelib

First, cmakelib (link in [Introduction](./index.md)) must be installed. Follow the README to
install it.

### Set the Package Tracker

In the Project root directory, the `CMLibStorage.cmake` needs to be added with the following content:

```cmake
SET(STORAGE_LIST DEP)
SET(STORAGE_LIST_DEP "https://github.com/bacpack-system/package-tracker.git")
```

!!! info

    The `CMLibStorage.cmake` file is a part of STORAGE component of cmakelib. So this component
    must be enabled with cmakelib. The `STORAGE_LIST` variable defines the list of storages and
    `STORAGE_LIST_<STORAGE>` defines the URL for each storage. In this case the Package Tracker
    is used.

This links the Project with Package Tracker, which by default enables adding Packages from
BringAuto's specific Package Repository (The Package Tracker points to Package Repository). Usually
the previously built Packages would be used, but for simplicity, the same Packages from BringAuto's
specific Package Repository are used instead. These Packages are exactly the same as Packages
defined in [example Package Context](https://github.com/bacpack-system/example-context).

!!! note

    The local Package Repository can't be easily used when building a Project, because currently
    the BacPack system supports only upstream Package Repositories. The usage of local Package
    Repository will be added in future releases.

### Configure CMakeLists

Following code shows needed configuration of `CMakeLists.txt` for using `curl` and `zlib` Package.

First, the cmakelib with required components must be added. Then the Packages must be defined with
`BA_PACKAGE_LIBRARY` macro. Finally the Package can be included with `FIND_PACKAGE`.

```cmake
# Add cmakelib with components
FIND_PACKAGE(CMLIB COMPONENTS CMDEF CMUTIL STORAGE REQUIRED)

# Define curl and zlib Packages
BA_PACKAGE_LIBRARY(curl v7.79.1)
BA_PACKAGE_LIBRARY(zlib v1.2.11 OUTPUT_PATH_VAR ZLIB_ROOT)

# Find zlib and curl Package
FIND_PACKAGE(ZLIB REQUIRED)
FIND_PACKAGE(CURL REQUIRED)
```

!!! note

    The `ZLIB_ROOT` is a helper variables for `zlib` Package, because `zlib` does not provide a
    CMake config file.

!!! note

    Each of the cmakelib components has its own git repository and adds specific functionality.
    Any or none of the following components can be used:

     - [CMDEF](https://github.com/cmakelib/cmakelib-component-cmdef) - adds wrappers for basic CMake features
     - [CMUTIL](https://github.com/cmakelib/cmakelib-component-cmutil) - Provides functionality for other cmakelib components
     - [STORAGE](https://github.com/cmakelib/cmakelib-component-storage) - mechanism for storing and retrieving build dependencies

### Build a Project

At this point, the Project can be built with `cmake` in the usual way:

```bash
mkdir -p _build && cd _build
cmake ..
make -j 8
```

## Create an App from Project

The [example Project](https://github.com/bacpack-system/example-project) with dependencies was
built in previous step. Now the built Project may be added to a Package Context as an App, which
can then be built and distributed in the same way as Packages.

### Modify CMakeLists

As previously mentioned, the Apps can't have any dependencies. So the example Project must be
built in a way that it has all its dependencies packaged with it. To achieve this, the
`CMakeLists.txt` of the Project must be modified to use macros for installing like this:

```cmake
# Install created target
CMDEF_INSTALL(TARGET example-project)

# Install all shared library dependencies needed for json_target
# and update RUNPATH.
BA_PACKAGE_DEPS_IMPORTED(example-project)
```

With this code, the `example-project` target and its dependencies are installed and the RUNPATH is
updated. When building with Packager, the files will be installed inside a Docker container and then
they will be extracted and packaged into an App archive.  

### Add App Config to Package Context

Now the App can be added to a Package Context.

??? example "example App Config"
    Following JSON is the Release variation App Config for the example Project. The Config is
    very similar to previously defined Package Configs in this example. The only important changes
    are to Git URI and Package name to reflect this example Project.

    ```json
    {
      "Env": {},
      "Git": {
        "URI": "https://github.com/bacpack-system/example-project.git",
        "Revision": "v1.0.0"
      },
      "Build": {
        "CMake": {
          "Defines": {
            "CMAKE_BUILD_TYPE": "Release"
          }
        }
      },
      "Package": {
        "Name": "example-project",
        "VersionTag": "v1.0.0",
        "PlatformString": {
          "Mode": "auto"
        },
        "IsLibrary": false,
        "IsDevLib": false,
        "IsDebug": false
      },
      "DockerMatrix": {
        "ImageNames": [
          "fedora41"
        ]
      }
    }
    ```

### Build the App

This App can be built using Packager with following command:

```bash
bap-builder build-app \
            --context context \
            --image-name fedora41 \
            --output-dir package_repo \
            --name example-project
```

After this command, the Packager creates a zip archive of the App. If it is extracted, the directory
structure looks like this:

```plaintext
example-project_v1.0.0_x86-64-fedora-41
├── bin
│   └── example-project
└── lib
    ├── cmake
    │   ├── example-project.cmake
    │   └── example-project-release.cmake
    ├── libcrypto.so -> libcrypto.so.3.2.4
    ├── libcrypto.so.3 -> libcrypto.so.3.2.4
    ├── libcrypto.so.3.2.4
    ├── libcurl.so
    ├── libssl.so -> libssl.so.3.2.4
    ├── libssl.so.3 -> libssl.so.3.2.4
    ├── libssl.so.3.2.4
    ├── libz.so
    ├── libz.so.1
    └── libz.so.1.2.11
```

As can be seen, the extracted structure contains the `example-project` binary in `bin` directory
and all its dependencies in `lib` directory. Now the App can be easily extracted to a system based
on fedora41 image.