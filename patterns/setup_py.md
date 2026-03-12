# ROS 2 `setup.py` Pattern

## Purpose

Use this reference when creating, modifying, or reviewing `setup.py` for a Python ROS 2 package.

In a ROS 2 Python package, `setup.py` is responsible for:

- installing the Python module
- registering ROS 2 resource markers
- installing `package.xml`
- installing launch, config, URDF, or RViz files when the package owns them
- exposing nodes through `console_scripts`

Default assumption:

- target ROS 2 distro is Jazzy unless the project says otherwise

---

## Ownership Rules

`setup.py` belongs to Python packages only.

Use it when the package contains:

- Python nodes
- Python helper modules
- launch files owned by that package
- runtime config files that must be installed

If the package is primarily C++, use `patterns/cmake.md` instead.

---

## Design Rules

- keep executable names stable and descriptive
- install every runtime asset the package expects at launch time
- use `find_packages(exclude=["test"])`
- keep `package.xml` as the source of dependency truth; `setup.py` handles Python packaging and install layout
- avoid absolute paths in `data_files`

---

## Example

```python
from glob import glob
import os

from setuptools import find_packages, setup


package_name = "my_arm_bringup"


setup(
    name=package_name,
    version="0.1.0",
    packages=find_packages(exclude=["test"]),
    data_files=[
        (
            "share/ament_index/resource_index/packages",
            [f"resource/{package_name}"],
        ),
        (f"share/{package_name}", ["package.xml"]),
        (
            os.path.join("share", package_name, "launch"),
            glob("launch/*.launch.py"),
        ),
        (
            os.path.join("share", package_name, "config"),
            glob("config/*.yaml"),
        ),
    ],
    install_requires=["setuptools"],
    zip_safe=True,
    maintainer="Your Name",
    maintainer_email="you@example.com",
    description="Bringup package for a ROS 2 robotic arm.",
    license="Apache-2.0",
    tests_require=["pytest"],
    entry_points={
        "console_scripts": [
            "hardware_monitor = my_arm_bringup.hardware_monitor:main",
        ],
    },
)
```

Why this structure is good:

- package metadata is explicit
- runtime assets are installed into package share
- console scripts match the Python module entry point
- the package remains relocatable after installation

---

## Checklist

When editing `setup.py`, verify:

- the Python import package exists
- `resource/<package_name>` exists
- `package.xml` is installed
- every launch or config directory used at runtime is installed
- every `console_scripts` target uses `<module>:main`
- executable names match launch files and documentation

---

## Common Mistakes

- forgetting the resource marker file
- forgetting to install launch or config files
- mismatching the console script name and launch executable
- installing from hardcoded paths
- editing `setup.py` without updating the actual Python module layout

---

## Related References

- `processes/implement.md`
- `patterns/launch.md`
- `project/package_map.md`
