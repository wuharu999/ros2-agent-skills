# ROS 2 Teaching Process

## Purpose

Use this process when the user wants to learn, review, or understand a ROS 2 concept, file, or subsystem in the context of a robotic arm project.

This process defines how to explain things, not what a specific file type should contain.

---

## Teaching Workflow

Follow this order:

1. Identify the target
   - file type
   - subsystem
   - runtime component
   - package responsibility

2. Place it in project context
   - where it lives in the workspace
   - which package owns it
   - which tool or node reads it

3. Explain the role
   - what the file or subsystem does
   - what inputs it expects
   - what outputs or side effects it produces

4. Connect it to the robot pipeline
   - description
   - control
   - planning
   - launch
   - visualization

5. Show a minimal example
   - keep examples short
   - use realistic names
   - follow ROS 2 best practices

6. Call out common mistakes
   - focus on the failures users actually hit in projects

7. End with practical next steps
   - what to inspect
   - what to edit
   - how to verify understanding in the running system

---

## Required Explanation Shape

When teaching a file, separate the explanation into these ideas:

- what it is
- where it belongs
- who reads it
- what other files it depends on
- what breaks when it is wrong

Do not explain a file in isolation if it is normally used with others.

Examples:

- a launch file should be connected to installed executables and parameter files
- a URDF should be connected to TF, `robot_state_publisher`, and controllers
- `setup.py` should be connected to package installation and `ros2 run`

---

## Project-Aware Teaching Rules

- use robotic arm examples instead of abstract "talker/listener" examples when possible
- prefer package names like `<robot_name>_description` or `<robot_name>_bringup` in examples
- explain runtime flow, not just syntax
- point out ownership boundaries between packages
- make the relationship between configuration files and runtime nodes explicit

---

## Example Quality Rules

Code examples should be:

- minimal
- syntactically valid
- easy to adapt into a real package
- install-safe and package-share aware when paths are involved

Avoid examples that:

- hardcode absolute paths
- skip install rules
- hide required supporting files
- use outdated ROS 2 patterns when a cleaner option exists

---

## Teaching Output Checklist

Before finishing a teaching response, make sure it includes:

- the reason this file or subsystem exists
- where it usually lives in the project
- one minimal example or pseudo-structure
- at least one common failure mode
- the next related file the user should understand
