# ROS 2 Project Structure Reference

## Purpose

Use this reference when deciding how to organize a robotic arm workspace, where files should live, and which package should own a change.

This file is about architecture and package boundaries.

---

## Recommended Package Split

A scalable robotic arm workspace usually separates responsibilities like this:

- `<robot_name>_description`
  - URDF or Xacro
  - meshes
  - RViz model configs
  - description-only launch files

- `<robot_name>_bringup`
  - top-level launch files
  - system startup orchestration
  - environment-specific config

- `<robot_name>_control`
  - controller YAML
  - control-specific launch files
  - control helper nodes

- `<robot_name>_hardware`
  - hardware interface plugins
  - drivers
  - transport-specific logic

- `<robot_name>_moveit_config`
  - MoveIt launch files
  - SRDF
  - planning and execution config

- `<robot_name>_msgs` or `<robot_name>_interfaces`
  - custom messages
  - services
  - actions

- `<robot_name>_app` or subsystem-specific runtime packages
  - task logic
  - demos
  - higher-level orchestration nodes

---

## Example Workspace Layout

```text
ros2_ws/
  src/
    my_arm_description/
      urdf/
      meshes/
      rviz/
      launch/
    my_arm_bringup/
      launch/
      config/
    my_arm_control/
      config/
      launch/
      src/ or my_arm_control/
    my_arm_hardware/
      include/
      src/
    my_arm_moveit_config/
      config/
      launch/
    my_arm_interfaces/
      msg/
      srv/
      action/
```

This layout scales because each package has a clear owner and runtime purpose.

---

## Ownership Rules

Use these rules when deciding where a file belongs:

- canonical robot geometry belongs in the description package
- top-level startup belongs in the bringup package
- hardware drivers belong in the hardware package
- controller definitions belong with control or bringup, depending on existing project ownership
- MoveIt-specific config belongs in the MoveIt package
- reusable interfaces belong in an interfaces package

Avoid "misc" packages and avoid copying the same configuration into multiple places.

---

## Dependency Direction

Prefer dependency flow like this:

- description -> no dependency on bringup or MoveIt
- control and hardware -> depend on description when needed
- MoveIt -> depends on description and active controllers
- bringup -> depends on everything it orchestrates
- application packages -> depend on the packages they use, not the reverse

Keep low-level packages reusable and avoid making description packages depend on runtime packages.

---

## Common Structural Mistakes

- putting canonical URDF files inside bringup
- mixing hardware drivers into the description package
- storing controller YAML in several packages
- letting top-level launch logic drift into every package
- creating circular package dependencies

---

## When to Open This File

Use this reference when the user asks:

- where a file should live
- whether to create a new package
- how to organize a new subsystem
- how to keep the project scalable as new components such as Gazebo are added

For ownership of a specific change, also open `project/package_map.md`.
