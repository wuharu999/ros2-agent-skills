# ROS 2 Implementation Process

## Purpose

Use this process when the user wants to add, modify, or integrate functionality in a ROS 2 robotic arm project.

This process is about planning and executing a working change across the right set of files.

---

## Implementation Workflow

### 1. Define the behavior change

State the requested outcome in concrete runtime terms.

Examples:

- "Add a Python node that publishes joint diagnostics"
- "Expose a launch argument to optionally start RViz"
- "Integrate a new controller for the gripper"

Do not start editing before the runtime goal is clear.

---

### 2. Identify the owning package

Decide where the change belongs before writing code.

Use `project/structure.md` and `project/package_map.md` when ownership is unclear.

Typical ownership examples:

- robot description changes -> description package
- controller config -> control or bringup package
- Python node -> Python package that owns the runtime behavior
- MoveIt planning config -> MoveIt config package
- top-level launch orchestration -> bringup package

---

### 3. List all affected files

ROS 2 changes are often multi-file even when the user mentions only one file.

Common combinations:

- new Python node
  - module file
  - `setup.py`
  - optional launch file
- new C++ node
  - source file
  - `CMakeLists.txt`
  - `package.xml`
  - optional launch file
- new controller
  - URDF or Xacro `ros2_control` block
  - controller YAML
  - launch integration
- new robot description component
  - URDF or Xacro
  - meshes
  - launch integration
  - RViz or MoveIt config when needed

---

### 4. Choose the minimal coherent implementation

Prefer small, composable changes that fit the existing package architecture.

Avoid:

- creating a new package when an existing owner is clear
- duplicating configuration across packages
- putting hardware, description, and bringup logic into the same file unless the project already does that

---

### 5. Implement with subsystem rules

Open the relevant pattern files and follow them closely.

Examples:

- Python packaging -> `patterns/setup_py.md`
- launch integration -> `patterns/launch.md`
- description changes -> `patterns/urdf.md`
- controller integration -> `patterns/ros2_control.md`
- C++ build updates -> `patterns/cmake.md`
- planning integration -> `patterns/moveit.md`

---

### 6. Verify the change

Choose the smallest useful verification:

- `colcon build --packages-select <pkg>`
- `python3 -m py_compile <file>`
- `ros2 run <pkg> <executable>`
- `ros2 launch <pkg> <launch_file>`
- runtime checks such as topic, TF, controller, or planning verification

Do not stop at file edits if a simple verification step is available.

---

### 7. Summarize impact

Explain:

- which files changed
- why each file needed to change
- what command or observation verifies success
- what follow-up work is still optional

---

## Implementation Quality Rules

- keep package ownership explicit
- prefer install-safe path handling
- keep names consistent across code, launch, and config
- write examples that are ready to adapt into production code
- avoid partial changes that leave the feature unusable

If a request touches multiple packages, call that out early and keep the dependency direction clean.
