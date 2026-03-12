# ROS 2 Control Pattern

## Purpose

Use this reference for `ros2_control` integration in a robotic arm project.

This includes:

- the `ros2_control` block in URDF or Xacro
- hardware interface plugins
- controller manager configuration
- controller YAML files
- controller startup from launch files

---

## Ownership and Placement

`ros2_control` usually spans several packages:

- description package
  - `ros2_control` XML in URDF or Xacro
- hardware or control package
  - hardware plugin implementation
- bringup or control package
  - controller YAML
  - launch integration

Keep clear ownership. Do not bury controller configuration inside unrelated packages.

---

## Design Rules

- joint names must match the robot description exactly
- command and state interfaces must match the hardware plugin and controller type
- load `joint_state_broadcaster` before motion controllers
- keep controller YAML readable and grouped under controller manager
- keep transport or device parameters close to the hardware plugin configuration

---

## Example Description Snippet

```xml
<ros2_control name="ArmHardware" type="system">
  <hardware>
    <plugin>my_arm_hardware/MyArmSystemHardware</plugin>
    <param name="device">/dev/ttyUSB0</param>
  </hardware>

  <joint name="shoulder_joint">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>

  <joint name="elbow_joint">
    <command_interface name="position"/>
    <state_interface name="position"/>
    <state_interface name="velocity"/>
  </joint>
</ros2_control>
```

## Example Controller YAML

```yaml
controller_manager:
  ros__parameters:
    update_rate: 250

    joint_state_broadcaster:
      type: joint_state_broadcaster/JointStateBroadcaster

    arm_controller:
      type: joint_trajectory_controller/JointTrajectoryController

arm_controller:
  ros__parameters:
    joints:
      - shoulder_joint
      - elbow_joint
    command_interfaces:
      - position
    state_interfaces:
      - position
      - velocity
```

Why this structure is good:

- the hardware plugin and joint interfaces are explicit
- controller manager owns controller type declarations
- the trajectory controller uses names that match the robot description

---

## Launch Integration Rules

When bringing up `ros2_control`, make sure the launch system:

- publishes the same robot description used by the hardware stack
- starts `ros2_control_node` or the system-specific equivalent
- spawns `joint_state_broadcaster` first
- spawns motion controllers only after controller manager is available

If the robot description used by `robot_state_publisher` differs from the one used by controller manager, expect hard-to-debug failures.

---

## Debug Checklist

When controllers fail to load or activate, check:

1. joint names match the URDF exactly
2. controller YAML matches available interfaces
3. the hardware plugin is discoverable
4. the controller manager is running
5. the robot description parameter reaches the correct node
6. the controller type matches the intended command mode

Useful commands:

- `ros2 control list_hardware_interfaces`
- `ros2 control list_controllers`
- `ros2 control load_controller --set-state active <controller_name>`

---

## Common Mistakes

- mismatched joint names
- using a trajectory controller with the wrong command interface
- forgetting `joint_state_broadcaster`
- launching controllers before controller manager is ready
- splitting the robot description into inconsistent copies

---

## Related References

- `patterns/urdf.md`
- `patterns/launch.md`
- `patterns/cmake.md`
- `project/package_map.md`
