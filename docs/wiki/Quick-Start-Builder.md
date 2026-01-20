# Quick Start: Luna Builder Tutorial

This is a step-by-step tutorial to create your first rig using Luna Builder. Follow these instructions exactly to avoid common errors.

## Prerequisites

Before starting, ensure you have:
- Maya 2020 or later running
- Luna integrated via mayaLib (Luna button in MayaLib menu)
- A skeleton ready to rig (or use the example below)

## Common Errors and Solutions

| Error | Cause | Solution |
|-------|-------|----------|
| "Asset is not set!" | No project/asset configured | Create a project first (Step 1) |
| "At least one input node is required!" | Missing Input node | Add Input node to your graph |
| Empty node palette | Plugins not loaded | Restart Maya or use Help > Reload Plugins |

---

## Step 1: Create a Luna Project

**This step is REQUIRED before using Luna Builder.**

### Option A: Using Python

```python
from luna.workspace import Project

# Create a new project (choose your path)
project = Project.create("C:/Users/YourName/Documents/LunaProjects/MyFirstRig")

# Verify project is set
print(f"Project set: {Project.get().path}")
```

### Option B: Using Luna Menu

1. **Luna > Project > Set Project**
2. Navigate to or create a folder for your project
3. Click **Set Project**

---

## Step 2: Create an Asset

An Asset represents what you're rigging (character, prop, etc.).

### Using Python

```python
from luna.workspace import Asset, Project

# Create an asset inside your project
asset = Asset(
    project=Project.get(),
    name="tutorial_character",
    typ="character"  # or "prop", "vehicle", etc.
)

# Set as current asset
asset.set()

print(f"Asset created: {asset.path}")
```

### Verify Asset is Set

In Luna Builder, the **Workspace** panel (right side) should show:
- Project name
- Asset name
- Asset type

If it shows "No project set" or similar, go back to Step 1.

---

## Step 3: Create a Simple Skeleton

If you don't have a skeleton, create this simple arm:

```python
import pymel.core as pm

# Create a simple arm skeleton
pm.select(clear=True)
shoulder = pm.joint(name="l_arm_01_jnt", position=(2, 15, 0))
elbow = pm.joint(name="l_arm_02_jnt", position=(5, 15, 0))
wrist = pm.joint(name="l_arm_03_jnt", position=(8, 15, 0))

# Orient joints
pm.joint(shoulder, edit=True, orientJoint="xyz", secondaryAxisOrient="yup", children=True)

pm.select(clear=True)
print("Skeleton created!")
```

---

## Step 4: Open Luna Builder

1. Click the **Luna** button in MayaLib menu, OR
2. Run in Python:
   ```python
   from luna_builder.main_window import BuilderMainWindow
   BuilderMainWindow.display()
   ```

---

## Step 5: Create a New Build

1. **File > New** (or Ctrl+N)
2. A new empty canvas tab appears

---

## Step 6: Build Your First Graph

### 6.1 Add Input Node (REQUIRED)

1. In the **Nodes Palette** (left panel), find **Graph > Input**
2. Drag **Input** node to the canvas
3. This is the starting point of your build

### 6.2 Add Character Node

1. Drag **Components > Character** to the canvas
2. Connect: **Input [exec out]** → **Character [exec in]**
3. Select the Character node
4. In **Attributes** panel, set:
   - **Name**: `tutorial`

### 6.3 Add FKIK Arm Component

1. Drag **Components > FKIK** to the canvas
2. Connect:
   - **Character [exec out]** → **FKIK [exec in]**
   - **Character [character out]** → **FKIK [character in]**
3. Select the FKIK node
4. In **Attributes** panel, set:
   - **Name**: `arm`
   - **Side**: `l`
   - **Start Joint**: `l_arm_01_jnt` (click the `>` button to pick from scene)
   - **End Joint**: `l_arm_03_jnt`
   - **Hook**: `0`

### 6.4 Add Output Node (OPTIONAL but recommended)

1. Drag **Graph > Output** to the canvas
2. Connect: **FKIK [exec out]** → **Output [exec in]**

Your graph should look like:

```
[Input] → [Character] → [FKIK] → [Output]
              ↓
          [character]
              ↓
          [FKIK character input]
```

---

## Step 7: Execute the Build

1. **Graph > Execute** (or Ctrl+E)
2. Watch the Maya viewport - your rig should be created!
3. Check the Script Editor for any errors

### Expected Result

- A character rig hierarchy in the Outliner
- FK controls on the arm
- IK handle and pole vector control
- FK/IK switch attribute

---

## Step 8: Save Your Build

1. **File > Save As** (or Ctrl+Shift+S)
2. Navigate to your asset's build folder:
   ```
   {project}/characters/tutorial_character/build/
   ```
3. Save as `tutorial_character.rig`

---

## Complete Example Script

If you want to do everything via Python:

```python
import pymel.core as pm
from luna.workspace import Project, Asset

# 1. Create project
project = Project.create("C:/Users/YourName/Documents/LunaProjects/Tutorial")

# 2. Create asset
asset = Asset(project=project, name="arm_tutorial", typ="character")
asset.set()

# 3. Create skeleton
pm.select(clear=True)
shoulder = pm.joint(name="l_arm_01_jnt", position=(2, 15, 0))
elbow = pm.joint(name="l_arm_02_jnt", position=(5, 15, 0))
wrist = pm.joint(name="l_arm_03_jnt", position=(8, 15, 0))
pm.joint(shoulder, edit=True, orientJoint="xyz", secondaryAxisOrient="yup", children=True)
pm.select(clear=True)

# 4. Create rig via Python (alternative to Builder)
from luna_rig.components import Character, FKIKComponent

character = Character.create(name="arm_tutorial")

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

character.attach_to_skeleton()
print("Rig created successfully!")
```

---

## Troubleshooting

### "Asset is not set!" Error

This is the most common error. It means Luna Builder doesn't know where to save/load data.

**Solution:**
```python
from luna.workspace import Project, Asset

# Check if project is set
project = Project.get()
if not project:
    print("No project! Creating one...")
    project = Project.create("C:/path/to/your/project")

# Check if asset is set
from luna.workspace import Asset
# Create and set an asset
asset = Asset(project=project, name="my_asset", typ="character")
asset.set()
```

### "At least one input node is required!"

Every Luna Builder graph needs an **Input** node as the starting point.

**Solution:** Add an Input node and connect it to your first component.

### Nodes Not Appearing in Palette

If the node palette is empty:

1. **Help > Reload Editor Plugins**
2. Or restart Maya

### Joints Not Found

If joint names aren't being found:

1. Make sure joints exist in the scene
2. Check spelling matches exactly
3. Use the `>` button in Attributes to pick from scene

---

## Next Steps

- Read the full [Builder Documentation](Builder.md)
- Explore [Components](Components.md) for all available rig parts
- Learn about [Naming Convention](Naming-Convention.md) customization
- Check [Functions](Functions.md) for utility operations

---

## Video Reference

The Luna Builder workflow is demonstrated in the original Luna repository. Key concepts:

1. **Project/Asset Structure**: Luna requires a project to organize files
2. **Node Graph**: Visual programming for rig creation
3. **Execution**: Builds run top-to-bottom, Input to Output
4. **Iteration**: Save builds and re-execute to update rigs
