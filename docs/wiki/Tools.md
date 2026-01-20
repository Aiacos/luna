# Tools Reference

This document covers the tools provided by Luna for animation, skin weights, and other rig-related operations.

## Table of Contents

1. [Animation Tools](#animation-tools)
2. [Skin Tools](#skin-tools)
3. [Import/Export Tools](#importexport-tools)
4. [Control Tools](#control-tools)
5. [Project Tools](#project-tools)

## Animation Tools

### Animation Baker

A comprehensive tool for baking animation between rig and skeleton.

#### Launching

```python
from luna.tools.anim_baker import AnimBakerDialog

AnimBakerDialog.display()
```

Or from Luna menu: **Luna > Tools > Animation Baker**

#### Modes

The Animation Baker has three modes:

##### 1. Skeleton Mode

Bake animation between rig controls and skeleton joints.

**Operations**:
- **Bake to Skeleton**: Bake control animation to bind joints
- **Bake to Rig**: Bake skeleton animation to controls (reverse)
- **Bake and Detach**: Bake to skeleton then remove constraints
- **Remove Rig**: Bake and remove entire rig

**Usage**:
1. Select components to bake (or leave empty for all)
2. Set time range
3. Click desired operation

```python
# Programmatic equivalent
character.bake_to_skeleton(time_range=(1, 100))
character.bake_and_detach(time_range=(1, 100))
character.remove(time_range=(1, 100))
```

##### 2. FKIK Mode

Bake between FK and IK on FKIK components.

**Options**:
- **Source**: FK or IK
- **Pole Vector**: Include PV in bake
- **Step**: Frame step for baking

**Usage**:
1. Select FKIK components
2. Choose source (FK or IK)
3. Set options
4. Click Bake

```python
# Programmatic equivalent
fkik_component.bake_fkik(
    source="fk",        # or "ik"
    time_range=(1, 100),
    bake_pv=True,
    step=1
)
```

##### 3. Space Mode

Bake space switches for controls.

**Options**:
- **Control**: Select control to bake
- **Target Space**: Choose from control's spaces or custom
- **Custom Object**: Any transform for custom space
- **Step**: Frame step

**Usage**:
1. Set the control
2. Choose target space
3. Set time range
4. Click Bake

```python
# Programmatic equivalent
control.bake_space(
    space_index=1,          # or space_name="World"
    time_range=(1, 100),
    step=1
)

# Custom space
control.bake_custom_space(
    space_object=locator,
    time_range=(1, 100),
    step=1
)
```

### Custom Space Tool

Create and manage custom space switches.

```python
from luna.tools.custom_space_tool import CustomSpaceTool

CustomSpaceTool.display()
```

**Features**:
- Add space targets to controls
- Remove space targets
- Switch spaces with matching

### Transfer Keyframes

Transfer animation between characters or rigs.

```python
from luna.tools.transfer_keyframes import TransferKeyframes

# Transfer between components
source_comp.copy_keyframes(
    time_range=(1, 100),
    target_component=target_comp,
    time_offset=0.0
)

# Transfer between characters
source_char.copy_keyframes(
    time_range=(1, 100),
    target_component=target_char,
    time_offset=0.0
)
```

### Driven Pose Exporter

Export and import SDK (Set Driven Key) poses.

```python
from luna.tools.driven_pose_exporter import DrivenPoseExporter

DrivenPoseExporter.display()
```

### SDK Corrective Exporter

Export and import corrective shapes driven by SDK.

```python
from luna.tools.sdk_corrective_exporter import SDKCorrectiveExporter

SDKCorrectiveExporter.display()
```

## Skin Tools

### Skin Manager

Manages skin cluster weights export/import.

```python
from luna_rig.importexport.skin import SkinManager

# Export all skin weights under geometry group
SkinManager.export_all()

# Import all skin weights
SkinManager.import_all()

# Export selected geometry
SkinManager.export_selected()

# Import selected geometry
SkinManager.import_selected()
```

### SkinCluster Class

Low-level skin cluster operations.

```python
from luna_rig.importexport.skin import SkinCluster

# Create from existing deformer
skin = SkinCluster(skin_cluster_node)

# Export weights
skin.export_data("/path/to/weights.skin", fmt="pickle")  # or "json"

# Import weights
SkinCluster.import_data(
    file_path="/path/to/weights.skin",
    geometry="body_geo",
    fmt="pickle"
)
```

**File Formats**:
- `pickle`: Binary format, faster, smaller files
- `json`: Human-readable, better for version control

### Config

Set default skin export format:

```python
from luna import Config, RigVars

Config.set(RigVars.skin_export_format, "pickle")  # or "json"
```

## Import/Export Tools

### Blendshape Manager

Export and import blendshape weights.

```python
from luna_rig.importexport.blendshape import BlendshapeManager

# Export
BlendshapeManager.export_all()
BlendshapeManager.export_selected()

# Import
BlendshapeManager.import_all()
BlendshapeManager.import_selected()
```

### Control Shapes Manager

Export and import control shapes.

```python
from luna_rig.importexport.control_shapes import ControlShapesManager

# Export all control shapes
ControlShapesManager.export_all()

# Import all control shapes
ControlShapesManager.import_all()
```

### Delta Mush Manager

Export and import delta mush weights.

```python
from luna_rig.importexport.deltamush import DeltaMushManager

# Export
DeltaMushManager.export_all()

# Import
DeltaMushManager.import_all()
```

### Driven Pose Manager

Export and import driven poses (SDK).

```python
from luna_rig.importexport.driven_pose import DrivenPoseManager

# Export
DrivenPoseManager.export_all()

# Import
DrivenPoseManager.import_all()
```

### ngLayers Manager

Export and import ngSkinTools layers.

```python
from luna_rig.importexport.nglayers import ngLayersManager

# Export (requires ngSkinTools)
ngLayersManager.export_all()

# Import
ngLayersManager.import_all()
```

### PoseSpace Manager

Export and import pose space deformer data.

```python
from luna_rig.importexport.posespace import PoseSpaceManager

# Export
PoseSpaceManager.export_all()

# Import
PoseSpaceManager.import_all()
```

## Control Tools

### Shape Manager Operations

#### Copy/Paste Shape

```python
from luna_rig.core.shape_manager import ShapeManager

# Copy shape (uses selection if no arg)
ShapeManager.copy_shape()

# Paste shape to selected
ShapeManager.paste_shape(pm.selected())
```

#### Copy/Paste Color

```python
# Copy color from selected
ShapeManager.copy_color()

# Paste color to selected
ShapeManager.paste_color()
```

#### Set Shape from Library

```python
# Set shape on control
ShapeManager.set_shape_from_lib(control.transform, "circle")
```

#### Save Custom Shape

```python
# Save current shape to library
ShapeManager.save_shape(transform, "my_custom_shape")
```

#### Set Color

```python
# By index
ShapeManager.set_color(controls, 17)

# By name
ShapeManager.set_color(controls, "blue")
```

#### Set Line Width

```python
ShapeManager.set_line_width(control.transform, 2.0)
```

### Control Operations

#### Bind Pose

```python
import luna_rig

# Reset selected controls to bind pose
luna_rig.functions.rigFn.selected_control_bind_pose()

# Reset entire asset to bind pose
luna_rig.functions.rigFn.asset_bind_pose()

# Reset single control
control = luna_rig.Control("l_arm_fk_00_ctl")
control.to_bind_pose()

# Write current pose as bind pose
control.write_bind_pose()
```

#### Mirror Pose

```python
control = luna_rig.Control("l_arm_fk_00_ctl")

# Mirror to opposite side
control.mirror_pose(
    behavior=True,
    direction="source",      # or "destination"
    skip_translate=[False, False, False],
    skip_rotate=[False, False, False]
)
```

#### Mirror Shape

```python
# Mirror shape
control.mirror_shape(behaviour=True)

# Mirror shape to opposite side control
control.mirror_shape_to_opposite()
```

## Project Tools

### Project Management

```python
from luna.workspace import Project

# Create new project
project = Project.create("/path/to/project")

# Set existing project
project = Project.set("/path/to/project")

# Get current project
current = Project.get()

# Exit project
Project.exit()

# Check if path is a project
if Project.is_project("/path"):
    pass

# Refresh recent projects list
Project.refresh_recent()
```

### Asset Management

```python
from luna.workspace import Asset, Project

# Ensure project is set
Project.set("/path/to/project")

# Create/access asset
asset = Asset(
    project=Project.get(),
    name="hero",
    typ="character"
)

# Get current asset
current = Asset.get()

# Access asset paths
print(asset.controls)    # Control shapes dir
print(asset.skeleton)    # Skeleton files dir
print(asset.build)       # Build files dir
print(asset.rig)         # Rig scene files dir
print(asset.weights.skin)  # Skin weights dir

# Get versioned file paths
print(asset.latest_skeleton_path)
print(asset.new_skeleton_path)
print(asset.latest_rig_path)
print(asset.new_rig_path)
print(asset.latest_build_path)
print(asset.new_build_path)
```

### Configuration

```python
from luna import Config

# Get setting
value = Config.get("my_setting", default="default")

# Get cached (faster for frequent access)
value = Config.get("my_setting", cached=True)

# Set setting
Config.set("my_setting", "new_value")

# Update multiple settings
Config.update({"key1": "val1", "key2": "val2"})

# Reset to defaults
Config.reset()
```

## Menu Access

All tools are accessible from the Luna menu in Maya:

```
Luna
├── Builder                 # Open Luna Builder
├── Set Project            # Set current project
├── Set Asset              # Set current asset
├── Tools
│   ├── Animation Baker    # Animation baking tool
│   ├── Transfer Keyframes
│   └── Custom Space Tool
├── Import/Export
│   ├── Skin Weights
│   │   ├── Export All
│   │   ├── Import All
│   │   ├── Export Selected
│   │   └── Import Selected
│   ├── Blendshapes
│   ├── Control Shapes
│   ├── Delta Mush
│   └── Driven Poses
├── Controls
│   ├── Copy Shape
│   ├── Paste Shape
│   ├── Copy Color
│   ├── Paste Color
│   └── Mirror Shape
└── Help
    ├── Documentation
    └── About
```

## Marking Menu

Luna provides a context-sensitive marking menu (Shift + Right-click):

**On Control Selection**:
- Switch FKIK (for FKIK components)
- Select all component controls
- Key all controls
- Reset to bind pose
- Component-specific actions

**On Component Selection**:
- Attach/detach skeleton
- Bake to skeleton
- Remove component

**On Character Selection**:
- Save bind poses
- Create selection sets
- Remove rig
