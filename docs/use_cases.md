# Use cases

There are several use cases how to use BacPack system, which will be described in this document.
All these use cases are described in [Usage](./usage.md).

The change of data and actions between user and BacPack components shows following sequence diagram.

```mermaid
sequenceDiagram
  actor User
  participant Packager
  participant Package Repository
  participant Package Context
  participant Package Tracker
  participant Target Project

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
    User->>Target Project: Puts link of Package Tracker
    User->>Target Project: Adds desired Package to CMakeLists
    User->>Target Project: Initiates Project build
    rect
      Note right of Package Repository: Target Project build
      Target Project->>Package Tracker: Asks for desired Package
      Package Repository->>Package Tracker: Takes present Package
      Package Tracker->>Target Project: Adds Package to build
      Target Project->>Target Project: Builds itself
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
