# ROS 2 Launch Pattern

## Purpose

Use this reference for `.launch.py` files and launch-level orchestration in a ROS 2 robotic arm project.

This includes:

- starting nodes
- passing parameters
- loading robot description
- including subordinate launch files
- optionally starting RViz, controllers, or MoveIt

---

## Ownership and Placement

Place launch files in the package that owns startup orchestration.

Typical layout:

- `<robot_name>_bringup`
  - top-level bringup
  - multi-node startup
  - environment-specific options
- `<robot_name>_description`
  - description-only launch files for RViz or state publisher
- `<robot_name>_moveit_config`
  - MoveIt-specific launch entry points

Do not scatter top-level bringup logic across unrelated packages.

---

## Design Rules

- use package-share resolution for installed files
- expose only meaningful launch arguments
- keep node groups readable by subsystem
- prefer inclusion of focused launch files over one oversized monolith
- keep executable names aligned with installed entry points or binaries
- use launch arguments for environment selection, not for hiding structural problems

---

## Recommended Structure

Most launch files should contain:

- imports grouped clearly
- `DeclareLaunchArgument` for external options
- package-share path resolution
- node actions grouped by subsystem
- optional includes for larger subsystems
- a single `generate_launch_description()` entry point

---

## Example

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.substitutions import Command, LaunchConfiguration, PathJoinSubstitution
from launch_ros.actions import Node
from launch_ros.parameter_descriptions import ParameterValue
from launch_ros.substitutions import FindPackageShare


def generate_launch_description():
    use_sim_time = LaunchConfiguration("use_sim_time")

    xacro_path = PathJoinSubstitution(
        [FindPackageShare("my_arm_description"), "urdf", "arm.urdf.xacro"]
    )
    robot_description = ParameterValue(
        Command(["xacro", " ", xacro_path]),
        value_type=str,
    )

    state_publisher = Node(
        package="robot_state_publisher",
        executable="robot_state_publisher",
        output="screen",
        parameters=[
            {
                "robot_description": robot_description,
                "use_sim_time": use_sim_time,
            }
        ],
    )

    return LaunchDescription(
        [
            DeclareLaunchArgument("use_sim_time", default_value="false"),
            state_publisher,
        ]
    )
```

Why this structure is good:

- it is install-safe
- it exposes one user-facing option
- it keeps description generation close to the consumer node
- it avoids hardcoded workspace paths

---

## Debug Checklist

When launch behavior fails, check in this order:

1. package builds and installs correctly
2. launch file is installed into the package share directory
3. package and executable names are correct
4. parameter and config paths resolve from package share
5. included launch files exist and are installed
6. expected nodes actually appear at runtime

Useful commands:

- `ros2 launch <pkg> <file.launch.py>`
- `ros2 pkg executables <pkg>`
- `ros2 node list`
- `ros2 param list`

---

## Common Mistakes

- hardcoded workspace paths
- wrong executable name for a Python console script
- forgetting to install the launch file
- overloading one launch file with unrelated responsibilities
- passing parameters with the wrong type
- mixing description, control, and planning setup without clear grouping

---

## Related References

- `processes/debug.md`
- `processes/implement.md`
- `patterns/setup_py.md`
- `patterns/urdf.md`
- `patterns/ros2_control.md`
- `patterns/moveit.md`
- `project/structure.md`
