# MoveIt Pattern

## Purpose

Use this reference for MoveIt integration in a ROS 2 robotic arm project.

This includes:

- MoveIt config packages
- planning group definitions
- SRDF alignment
- kinematics and planning pipeline configuration
- controller mapping for trajectory execution
- MoveIt launch integration

---

## Ownership and Placement

MoveIt assets usually belong in a dedicated package such as:

- `<robot_name>_moveit_config`

That package typically owns:

- MoveIt launch files
- SRDF and kinematics config
- planning pipeline config
- controller execution config
- RViz planning configuration

Keep MoveIt-specific config separate from the canonical robot description package.

---

## Design Rules

- keep planning group names stable and human-readable
- make SRDF groups match URDF joint names exactly
- keep controller execution config aligned with the active `ros2_control` controllers
- treat the description package as the source of truth for robot geometry
- keep MoveIt launch files thin when a generated config package already provides the heavy lifting

---

## Example Launch Pattern

```python
from launch import LaunchDescription
from launch_ros.actions import Node
from moveit_configs_utils import MoveItConfigsBuilder


def generate_launch_description():
    moveit_config = (
        MoveItConfigsBuilder(
            "my_arm",
            package_name="my_arm_moveit_config",
        )
        .to_moveit_configs()
    )

    move_group = Node(
        package="moveit_ros_move_group",
        executable="move_group",
        output="screen",
        parameters=[moveit_config.to_dict()],
    )

    return LaunchDescription([move_group])
```

Why this structure is good:

- it reuses the generated MoveIt config package cleanly
- configuration stays centralized
- the launch file stays small and maintainable

---

## Integration Rules

MoveIt changes often require checking:

- URDF or Xacro joint names
- SRDF group definitions
- kinematics solver config
- planning pipeline settings
- trajectory execution controller mapping
- launch integration with `ros2_control`

If planning succeeds but execution fails, the issue is often in controller mapping or `ros2_control` state, not in SRDF.

---

## Debug Checklist

When MoveIt is not behaving correctly, check:

1. the robot description and SRDF describe the same joints and groups
2. the MoveIt config package is loading the intended files
3. the planning group name matches the requested group
4. the controller execution mapping points to an active controller
5. TF and current joint states are available

Useful signals:

- MoveIt startup logs
- active controller list
- current joint state topic
- RViz MotionPlanning panel status

---

## Common Mistakes

- SRDF group names that drift from the URDF
- controller execution mapping that points to the wrong controller
- assuming planning failure is a geometry issue when TF or joint states are missing
- editing the description in the MoveIt package instead of the description package

---

## Related References

- `patterns/urdf.md`
- `patterns/ros2_control.md`
- `patterns/launch.md`
- `project/structure.md`
