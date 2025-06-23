# Use cases

This document describes several use cases for the BacPack System. All these use cases are described
further in [Usage](./example_usage.md).

The following sequence diagram illustrates data flow and interactions between user and BacPack
components for listed use cases.

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

After adding a Package Config to Package Context, the Package can be built by Packager and hosted
in a Package Repository. The Package can then be easily added to projects by including it in
CMakeLists.

## Use already built Packages in my CMake based project

To use a Package that was built with Packager and uploaded to a Package Repository, use Package
Tracker macros to add this Package to the application. The Package Tracker repository URL must also
be set in `CMLibStorage.cmake` in the root directory of the application.
