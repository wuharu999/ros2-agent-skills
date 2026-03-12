# ROS 2 URDF and Xacro Pattern

## Purpose

Use this reference for robot description files:

- URDF
- Xacro
- mesh references
- link and joint definitions
- frame layout
- `ros2_control` attachment points

For most real robotic arms, prefer Xacro once the description contains repeated structures, variants, or simulation-specific options.

---

## Ownership and Placement

Robot description files usually belong in a description package such as:

- `<robot_name>_description`

Typical folders:

- `urdf/`
- `meshes/`
- `rviz/`
- `config/` when description-related parameters are needed

Keep the canonical robot model in one place. Do not duplicate the URDF across bringup, control, and MoveIt packages.

---

## Design Rules

- keep link, joint, and frame names consistent across the project
- use SI units
- provide inertial data for moving links
- keep visual and collision geometry aligned
- place fixed world or base attachments explicitly
- use Xacro macros to remove duplication, not to hide structure
- keep `ros2_control` integration near the joints it configures

---

## Minimal Example

```xml
<?xml version="1.0"?>
<robot name="my_arm" xmlns:xacro="http://www.ros.org/wiki/xacro">
  <link name="base_link"/>

  <link name="shoulder_link">
    <inertial>
      <origin xyz="0 0 0.05" rpy="0 0 0"/>
      <mass value="1.2"/>
      <inertia ixx="0.01" ixy="0.0" ixz="0.0" iyy="0.01" iyz="0.0" izz="0.005"/>
    </inertial>
    <visual>
      <origin xyz="0 0 0.05" rpy="0 0 0"/>
      <geometry>
        <cylinder radius="0.04" length="0.10"/>
      </geometry>
    </visual>
    <collision>
      <origin xyz="0 0 0.05" rpy="0 0 0"/>
      <geometry>
        <cylinder radius="0.04" length="0.10"/>
      </geometry>
    </collision>
  </link>

  <joint name="shoulder_joint" type="revolute">
    <parent link="base_link"/>
    <child link="shoulder_link"/>
    <origin xyz="0 0 0.08" rpy="0 0 0"/>
    <axis xyz="0 0 1"/>
    <limit lower="-3.14" upper="3.14" effort="20.0" velocity="2.0"/>
  </joint>
</robot>
```

Why this structure is good:

- the kinematic chain is explicit
- inertia is present for the moving link
- visual and collision geometry are aligned
- the joint origin and axis are clear

---

## Integration Rules

URDF or Xacro changes often affect:

- `robot_state_publisher`
- TF
- `joint_state_publisher` or hardware state interfaces
- `ros2_control`
- RViz display config
- MoveIt SRDF and controller mappings

If a joint name changes, assume controller config, MoveIt config, and visualization may also need updates.

---

## Debug Checklist

When the robot model behaves incorrectly, check:

1. frame names are unique and consistent
2. parent and child link relationships are correct
3. joint axes match the intended motion
4. mesh paths resolve from installed package share
5. inertial data exists for moving links
6. `ros2_control` joint names match the URDF exactly

Useful commands:

- `check_urdf <file>`
- `ros2 run tf2_tools view_frames`
- RViz robot model and TF displays

---

## Common Mistakes

- missing inertial data
- wrong joint axis
- hidden frame offsets that break control and planning
- inconsistent naming between URDF, controller YAML, and SRDF
- hardcoded mesh paths outside the package

---

## Related References

- `patterns/ros2_control.md`
- `patterns/moveit.md`
- `patterns/launch.md`
- `project/structure.md`
