# Getting Started with Luna

This guide will help you install Luna and get started with your first rig.

## Table of Contents

1. [System Requirements](#system-requirements)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Project Setup](#project-setup)
5. [First Rig](#first-rig)
6. [Next Steps](#next-steps)

## System Requirements

- **Autodesk Maya**: 2019 or later (2020+, 2022+, 2023+ supported)
- **Python**: Maya's built-in Python 3 (Maya 2022+) or Python 2.7 (Maya 2019-2020)
- **Operating System**: Windows, Linux, or macOS

## Installation

### Method 1: Drag and Drop Installer (Recommended)

The easiest way to install Luna is using the drag-and-drop installer:

1. Download or clone the Luna repository:
   ```bash
   git clone https://github.com/Aiacos/luna.git
   ```

2. Open Maya

3. Drag `drag_drop_installer.py` into the Maya viewport

4. Follow the installation prompts to confirm the modules directory path

5. Restart Maya

6. Enable `luna_plugin.py` in the Plugin Manager:
   - Go to **Windows > Settings/Preferences > Plug-in Manager**
   - Find `luna_plugin.py` and check both **Loaded** and **Auto load**

### Method 2: Manual Installation

For more control over the installation:

1. Clone the repository:
   ```bash
   git clone https://github.com/Aiacos/luna.git
   ```

2. Add Luna to your Maya environment by editing `Maya.env`:

   **Windows**:
   ```
   PYTHONPATH = C:/path/to/luna
   MAYA_MODULE_PATH = C:/path/to/luna
   ```

   **Linux/macOS**:
   ```
   PYTHONPATH = /path/to/luna
   MAYA_MODULE_PATH = /path/to/luna
   ```

3. Add to your `userSetup.py`:
   ```python
   import luna
   import luna_rig
   ```

4. Restart Maya

### Verifying Installation

After installation, verify Luna is working:

```python
# In Maya Script Editor
import luna
import luna_rig
print(luna.__version__)  # Should print version number
```

You should also see the **Luna** menu in Maya's menu bar.

## Quick Start

### Using the Luna Menu

Once installed, Luna adds a menu to Maya with quick access to:

- **Luna Builder**: Open the visual node editor
- **Project Management**: Set or create projects
- **Tools**: Animation baker, skin tools, etc.
- **Configuration**: Adjust Luna settings

### Using the Marking Menu

Luna provides a context-sensitive marking menu for quick access to common operations:

1. Select a rig control
2. Hold **Shift + Right-click** to access the marking menu
3. Access component-specific actions

## Project Setup

Luna uses a project/asset structure to organize your rigging work.

### Creating a Project

```python
from luna.workspace import Project

# Create a new project
project = Project.create("/path/to/my_project")

# Or set an existing project
project = Project.set("/path/to/existing_project")

# Get current project
current = Project.get()
print(current.path)
```

### Creating an Asset

```python
from luna.workspace import Asset, Project

# Ensure a project is set
Project.set("/path/to/my_project")

# Create or access an asset
asset = Asset(
    project=Project.get(),
    name="hero_character",
    typ="character"
)

print(asset.path)           # Full asset path
print(asset.skeleton)       # Skeleton files directory
print(asset.weights.skin)   # Skin weights directory
print(asset.build)          # Build files directory
```

### Project Structure

```
my_project/
├── luna.proj                # Project tag file
├── my_project.meta          # Project metadata
├── characters/
│   └── hero/
│       ├── hero.meta
│       ├── controls/        # Control shape files
│       ├── skeleton/        # Skeleton files
│       ├── build/           # Build files (.rig)
│       ├── rig/             # Rig scene files
│       ├── settings/
│       ├── weights/
│       │   ├── skin/
│       │   ├── blendshape/
│       │   └── delta_mush/
│       └── data/
│           ├── blendshapes/
│           └── driven_poses/
└── props/
    └── ...
```

## First Rig

Let's create a simple FK/IK arm rig to get familiar with Luna.

### Prerequisites

Create a skeleton in Maya with these joints:
- `c_spine_01_jnt` (root)
- `l_arm_01_jnt` (shoulder)
- `l_arm_02_jnt` (elbow)
- `l_arm_03_jnt` (wrist)

### Step 1: Import Luna

```python
import pymel.core as pm
import luna_rig
from luna_rig.components import Character, FKIKComponent
```

### Step 2: Create the Character

```python
# Create the character component (rig root)
character = Character.create(name="my_character")

# This creates:
# - Main rig hierarchy
# - Root control
# - Groups for geometry, skeleton, controls, etc.
```

### Step 3: Create the Arm Component

```python
# Create FKIK arm component
arm = FKIKComponent.create(
    meta_parent=character,
    hook=0,
    character=character,
    side="l",
    name="arm",
    start_joint="l_arm_01_jnt",
    end_joint="l_arm_03_jnt",
    default_state=1,  # Start in IK mode
    tag="body"
)
```

### Step 4: Finalize the Rig

```python
# Attach rig to skeleton
character.attach_to_skeleton()

# Save bind poses
character.save_bind_pose()

# Create selection sets
character.create_selection_sets()
```

### Complete Script

```python
import pymel.core as pm
import luna_rig
from luna_rig.components import Character, FKIKComponent

# Create character
character = Character.create(name="my_character")

# Create FKIK arm
arm = FKIKComponent.create(
    meta_parent=character,
    hook=0,
    character=character,
    side="l",
    name="arm",
    start_joint="l_arm_01_jnt",
    end_joint="l_arm_03_jnt",
    default_state=1,
    tag="body"
)

# Finalize
character.attach_to_skeleton()
character.save_bind_pose()
character.create_selection_sets()

print("Rig created successfully!")
```

## Using Luna Builder

For a visual approach to rigging, use Luna Builder:

1. Open Luna Builder: **Luna > Builder**

2. Create a new build: **File > New**

3. Add nodes from the palette:
   - Drag **Input** node to the canvas
   - Drag **Character** node and connect it to Input
   - Drag component nodes (FK, FKIK, etc.) and configure them

4. Execute the build: **Graph > Execute** or press **Ctrl+E**

5. Save your build: **File > Save As** (creates a `.rig` file)

See the [Builder Documentation](Builder.md) for detailed instructions.

## Next Steps

Now that you have Luna installed and created your first rig, explore these topics:

- [Architecture](Architecture.md) - Understand how Luna is structured
- [Core Concepts](Core-Concepts.md) - Learn about MetaNode, Component, Control
- [Components](Components.md) - Explore all available rig components
- [Functions](Functions.md) - Discover utility functions
- [Naming Convention](Naming-Convention.md) - Configure naming templates

## Troubleshooting

### Luna menu not appearing

1. Check Plugin Manager for `luna_plugin.py`
2. Verify PYTHONPATH includes Luna directory
3. Check Script Editor for import errors

### Import errors

```python
# Debug import issues
import sys
print([p for p in sys.path if 'luna' in p.lower()])
```

### Control shapes not visible

- Check Maya display layers
- Verify control curve visibility
- Try adjusting line width in config

### "Failed to evaluate class string" error

This occurs when a metaType references a class that cannot be imported. Ensure Luna modules are imported before accessing meta nodes:

```python
import luna
import luna_rig  # Must import before accessing existing rigs
```

For more help, see the [API Reference](API-Reference.md) or report issues on GitHub.
