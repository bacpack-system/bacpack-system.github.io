# Use cases

There are several use cases how to use BacPack system, which will be described in this document.
All these use cases are described further in [Usage](./example_usage.md).

The change of data and actions between user and BacPack components shows following sequence diagram.

```mermaid
sequenceDiagram
  actor User
  participant Packager
  participant Package Repository
  participant Package Context
  participant Package Tracker
  participant Project

  rect
    Note right of User: Use case:<br/> Add Package to Context
    User->>Package Context: Creates Package definition
    User->>Packager: Initiates a Package build
    rect
      Note right of Packager: Packager builds Package
      Package Context->>Packager: Takes the Package definition
      Packager->>Package Repository: Puts built Package
    end
  end
  rect
    Note right of User: Use case:<br/> Use Package in project
    User->>Project: Puts link of Package Tracker
    User->>Project: Adds desired Package to CMakeLists
    User->>Project: Initiates Project build
    rect
      Note right of Package Repository: Project build
      Project->>Package Tracker: Asks for desired Package
      Package Repository->>Package Tracker: Takes present Package
      Package Tracker->>Project: Adds Package to build
      Project->>Project: Project build
    end
  end
```

## Add Package to Package context

If you want to take an advantage of BacPack system dependency management, you can consider adding
a Package to Package Context. This Package can be build by Packager and hosted in a Package
Repository. Then you can easily add the Package to your project by adding it in a CMakeLists.

## Use already built Packages in my CMake based project

If you want to use a Package which was built with Packager and is uploaded in a Package Repository,
you need to use a Package Tracker macros to add this Package to your application. You also need to
set Package Tracker repo url in `CMLibStorage.cmake` in the root directory of your application.
