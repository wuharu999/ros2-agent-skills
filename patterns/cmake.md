# ROS 2 `CMakeLists.txt` Pattern

## Purpose

Use this reference for C++ ROS 2 packages and any package built with `ament_cmake`.

This includes:

- executables
- libraries
- include directories
- install rules
- dependency declarations

---

## Ownership Rules

Use `CMakeLists.txt` when the package contains:

- C++ nodes
- C++ libraries
- plugins
- hardware interfaces
- non-Python executables

Do not mix `setup.py` packaging rules into a CMake-only package unless the project intentionally uses a hybrid layout.

---

## Design Rules

- keep targets small and named by responsibility
- declare dependencies at the target level with `ament_target_dependencies`
- install executables into `lib/${PROJECT_NAME}`
- install launch, config, and description assets when the package owns them
- keep warnings and compile features explicit
- let `package.xml` and `CMakeLists.txt` agree on dependencies

---

## Example

```cmake
cmake_minimum_required(VERSION 3.8)
project(my_arm_control)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)

add_executable(joint_state_monitor src/joint_state_monitor.cpp)
target_compile_features(joint_state_monitor PUBLIC cxx_std_17)
ament_target_dependencies(joint_state_monitor
  rclcpp
  sensor_msgs
)

install(
  TARGETS joint_state_monitor
  DESTINATION lib/${PROJECT_NAME}
)

install(
  DIRECTORY launch config
  DESTINATION share/${PROJECT_NAME}
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
```

Why this structure is good:

- dependencies are explicit
- the executable is installable with standard ROS 2 layout
- runtime assets are packaged with the executable
- the package is ready for linting and integration into a larger workspace

---

## Checklist

When editing `CMakeLists.txt`, verify:

- every source target exists
- required dependencies are found with `find_package`
- each target declares dependencies with `ament_target_dependencies`
- install rules cover executables and runtime assets
- `package.xml` contains matching build and exec dependencies

---

## Common Mistakes

- forgetting to install the executable
- forgetting to install launch or config files
- linking a dependency without declaring it in `package.xml`
- using global include paths instead of target-scoped configuration
- building one oversized executable instead of separate focused nodes

---

## Related References

- `processes/implement.md`
- `patterns/launch.md`
- `patterns/ros2_control.md`
- `project/package_map.md`
