# ROS 2 Package Map Reference

## Purpose

Use this reference to decide which package owns which responsibility and to reason about cross-package changes.

This is the bridge between project architecture and implementation work.

---

## Recommended Responsibility Map

Use a package map like this as the default mental model:

- `<robot_name>_description`
  - owns robot geometry, frames, meshes, and model visualization
  - should not own hardware drivers or task logic

- `<robot_name>_bringup`
  - owns full-system launch files and startup sequencing
  - may own environment-level config that wires packages together

- `<robot_name>_control`
  - owns controller configuration and control-related runtime nodes
  - should stay aligned with joint names from the description package

- `<robot_name>_hardware`
  - owns hardware interfaces, drivers, and device communication
  - should expose clean interfaces upward without owning user-facing launch orchestration

- `<robot_name>_moveit_config`
  - owns planning groups, planning config, and execution mapping
  - depends on the robot description and active controller stack

- `<robot_name>_interfaces`
  - owns custom messages, services, and actions shared across packages

- application packages
  - own use-case-specific nodes, demos, or workflows
  - should consume lower-level packages rather than reshaping them

---

## Change Impact Map

Use this when planning edits:

- add a new Python node
  - likely touches one runtime package
  - also touches `setup.py`
  - may touch bringup if it must be launched automatically

- add a new controller
  - touches description, control config, and bringup
  - may touch MoveIt if trajectory execution changes

- rename a joint
  - touches description
  - touches controller YAML
  - touches MoveIt config
  - may touch application code and RViz config

- add simulation support such as Gazebo
  - may justify a new pattern file and possibly a dedicated simulation package
  - should not replace the canonical description package

---

## Ownership Decision Rules

When deciding where a new file belongs, ask:

1. Which package is the source of truth for this data?
2. Which runtime component reads the file first?
3. Would another package need to copy this file if it lived here?

If the answer to the third question is yes, the ownership boundary is probably wrong.

---

## Practical Use

When implementing or debugging:

- identify the owning package first
- list every dependent package that may need coordinated updates
- keep changes flowing from source-of-truth packages outward

Example:

- change joint names in the description package first
- then update controller config
- then update MoveIt config
- then update launch or application code

---

## Common Mistakes

- editing generated or downstream config instead of the source package
- putting shared config in a package that is too high-level
- changing names in one package and forgetting the dependent ones
- solving architecture confusion by copying files instead of clarifying ownership

---

## When to Open This File

Use this reference when the user asks:

- which package should own a new node or config file
- which packages are affected by a subsystem change
- how to keep the project maintainable as new domains such as Gazebo are added

For broader folder layout decisions, also open `project/structure.md`.
