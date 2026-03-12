# Gazebo Pattern

## Purpose

Use this reference for Gazebo-based simulation in a ROS 2 robotic arm project.

This includes:

- simulator launch files
- robot spawning into the world
- simulated sensor configuration
- bridge configuration between simulator transport and ROS 2
- controller startup in simulation
- world and environment ownership

Keep this file focused on simulation-specific guidance. The canonical robot model still belongs in the description package.

---

## Ownership and Placement

Gazebo integration usually spans several packages:

- `<robot_name>_description`
  - canonical URDF or Xacro
  - mesh assets
  - simulation toggles in Xacro when needed

- `<robot_name>_bringup` or `<robot_name>_simulation`
  - Gazebo launch files
  - spawn logic
  - bridge configuration
  - world selection

- `<robot_name>_control`
  - controller YAML reused by simulation when appropriate

If simulation becomes large, prefer a dedicated simulation package instead of overloading bringup.

---

## Design Rules

- keep one canonical robot description and derive simulation variants from it
- keep simulation-only parameters explicit instead of forking the whole robot model
- separate world startup, robot spawn, and controller startup into readable launch sections
- keep bridge configuration versioned with the simulation launch logic
- reuse controller config only when the simulated interfaces match the real interfaces
- treat simulator integration as an environment layer, not as the source of truth for the robot

---

## Typical Simulation Flow

The common runtime sequence is:

1. start Gazebo with the selected world
2. publish or generate the robot description
3. spawn the robot entity into the simulator
4. start control components needed for simulation
5. start bridges for topics, clocks, or sensors if required
6. verify TF, joint states, and controllers

This order matters. Many simulation failures come from starting controllers or bridges before the robot exists in the simulator.

---

## Launch Pattern

Gazebo launch files should usually make these responsibilities visible:

- world selection
- simulator start
- robot description source
- spawn action
- controller startup
- optional bridge or sensor nodes

Prefer splitting large simulation startup into focused included launch files when the project supports both hardware and simulation bringup.

---

## Example Structure

```python
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument, IncludeLaunchDescription
from launch.launch_description_sources import PythonLaunchDescriptionSource
from launch.substitutions import LaunchConfiguration, PathJoinSubstitution
from launch_ros.actions import Node
from launch_ros.substitutions import FindPackageShare


def generate_launch_description():
    world = LaunchConfiguration("world")

    gazebo = IncludeLaunchDescription(
        PythonLaunchDescriptionSource(
            PathJoinSubstitution(
                [FindPackageShare("ros_gz_sim"), "launch", "gz_sim.launch.py"]
            )
        ),
        launch_arguments={"gz_args": ["-r ", world]}.items(),
    )

    spawn_robot = Node(
        package="ros_gz_sim",
        executable="create",
        arguments=[
            "-name",
            "my_arm",
            "-topic",
            "robot_description",
        ],
        output="screen",
    )

    return LaunchDescription(
        [
            DeclareLaunchArgument(
                "world",
                default_value=PathJoinSubstitution(
                    [FindPackageShare("my_arm_simulation"), "worlds", "empty.sdf"]
                ),
            ),
            gazebo,
            spawn_robot,
        ]
    )
```

Why this structure is good:

- the world is configurable
- simulator startup is separated from spawn logic
- robot spawning uses the published description instead of duplicating the model
- the file stays readable as simulation features grow

---

## Integration Rules

Gazebo work often touches several other references:

- `patterns/urdf.md`
  - simulation tags or Xacro arguments
- `patterns/launch.md`
  - orchestration and launch composition
- `patterns/ros2_control.md`
  - simulated controller startup and interface alignment
- `project/structure.md`
  - deciding whether simulation deserves its own package

If a change is simulation-only, avoid contaminating hardware-only launch paths with simulator assumptions.

---

## Debug Checklist

When Gazebo simulation is not behaving correctly, check:

1. the world file exists and is installed
2. the simulator launch package and executable names are correct
3. the robot description is being published before spawn
4. the entity actually appears in the simulator
5. `/clock`, joint states, and TF are present as expected
6. controllers are using interfaces compatible with the simulated system
7. bridges are configured for the topics the rest of the stack expects

Useful checks:

- `ros2 topic list`
- `ros2 topic echo /clock`
- `ros2 node list`
- `ros2 control list_controllers`

Also inspect simulator logs and entity listings when the robot fails to spawn.

---

## Common Mistakes

- duplicating the robot model for simulation instead of deriving it from the canonical description
- starting controllers before the robot is spawned
- using hardware controller assumptions that do not match simulated interfaces
- hiding simulation-only behavior inside generic bringup without clear launch arguments
- forgetting to install world files, bridge config, or simulation launch files

---

## When to Open This File

Use this reference when the user asks to:

- add Gazebo simulation support
- spawn the robot in simulation
- debug simulator startup or spawning
- bridge Gazebo topics into ROS 2
- organize simulation packages cleanly as the project grows

For package ownership, also open `project/package_map.md`.
