# Use cases

There are several use cases how to use BacPack system, which will be described in this document.
All these use cases are described in [Usage](./usage.md).

## Use already built Packages in my CMake based project

If you want to use a Package which was built with Packager and is uploaded in a Package repository,
you need to use a Package tracker macros to add this Package to your application. You also need to
set Package tracker repo url in `CMLibStorage.cmake` in the root directory in your application.

## Add Packages to Package context

If you want to take an advantage of BacPack system dependency management, you can consider adding
a Package to Package context. This Package can be build by Packager and hosted in a Package
repository. Then you can easily add the Package to your project by adding it in a CMakeLists.
