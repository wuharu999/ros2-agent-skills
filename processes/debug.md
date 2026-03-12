# ROS 2 Debug Process

## Purpose

Use this process when the user is troubleshooting a ROS 2 issue in a robotic arm project.

This file defines the debugging order. Do not guess across multiple subsystems before the symptom is classified.

---

## Core Debug Order

### 1. State the exact symptom

Classify the failure before changing code.

Common categories:

- build failure
- package discovery failure
- executable not found
- launch failure
- node startup failure
- topic or service mismatch
- TF tree failure
- robot description failure
- controller load or activation failure
- MoveIt planning failure
- RViz visualization failure

State what is broken in one sentence.

---

### 2. Identify the owning package and files

Inspect the smallest relevant set first.

Examples:

- build failure
  - `package.xml`
  - `setup.py`
  - `CMakeLists.txt`
- launch failure
  - launch file
  - installed resource paths
  - executable definitions
- controller failure
  - robot description `ros2_control` block
  - controller YAML
  - launch integration
- MoveIt failure
  - MoveIt config package
  - controller mapping
  - planning group names

---

### 3. Reproduce with the narrowest command

Prefer the smallest command that exposes the failure.

Examples:

- `colcon build --packages-select <pkg>`
- `source install/setup.bash`
- `ros2 run <pkg> <executable>`
- `ros2 launch <pkg> <file.launch.py>`
- `ros2 control list_controllers`

Do not claim a fix without rerunning the relevant command.

---

### 4. Check runtime visibility

Inspect what ROS 2 actually sees:

- `ros2 pkg executables <pkg>`
- `ros2 node list`
- `ros2 topic list`
- `ros2 service list`
- `ros2 param list`
- `ros2 control list_controllers`
- `ros2 control list_hardware_interfaces`

Use observed runtime state to narrow the root cause.

---

### 5. Inspect subsystem-specific evidence

Choose the tools that match the symptom:

- `ros2 doctor`
- `ros2 topic echo <topic>`
- `ros2 topic info <topic>`
- `ros2 run tf2_tools view_frames`
- `ros2 run tf2_ros tf2_echo <frame_a> <frame_b>`
- RViz fixed frame and display status
- controller manager logs
- MoveIt logs and planning scene status

---

### 6. Fix one likely root cause at a time

Avoid mixing unrelated changes.

Good fixes are:

- minimal
- directly tied to the observed failure
- easy to verify

Bad fixes are:

- broad refactors during debugging
- renaming several packages or files without evidence
- changing launch, description, and controller files all at once unless the defect spans them

---

### 7. Verify after each fix

Verification should match the original symptom.

Examples:

- executable appears in `ros2 pkg executables`
- node appears in `ros2 node list`
- topic appears with the correct type
- TF chain becomes connected
- controller loads and becomes active
- robot model appears correctly in RViz
- MoveIt can plan for the expected group

---

### 8. Summarize the result

End each debug pass with:

- root cause
- exact file changes
- verification command or observation
- any remaining unresolved issue

---

## Debugging Rules

- inspect before editing
- change the minimum necessary files
- keep package ownership clear
- prefer evidence over intuition
- use project references if package boundaries are unclear

If a failure could come from packaging, launch, and runtime config, start with packaging and installation first. Many ROS 2 runtime failures are really install or naming failures.
