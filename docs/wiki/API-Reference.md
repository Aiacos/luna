# API Reference

This document provides a comprehensive API reference for Luna developers.

## Table of Contents

1. [luna Package](#luna-package)
2. [luna_rig Package](#luna_rig-package)
3. [luna_builder Package](#luna_builder-package)
4. [Quick Reference Tables](#quick-reference-tables)

---

## luna Package

Core framework package.

### luna.Config

Configuration management class.

```python
from luna import Config
```

#### Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `get(key, default=None, cached=False)` | key: str, default: Any, cached: bool | Any | Get config value |
| `set(key, value)` | key: str, value: Any | None | Set config value |
| `update(data_dict)` | data_dict: dict | None | Update multiple values |
| `reset()` | None | None | Reset to defaults |

### luna.Logger

Custom logging system.

```python
from luna import Logger
```

#### Methods

| Method | Parameters | Description |
|--------|------------|-------------|
| `info(message)` | message: str | Log info message |
| `warning(message)` | message: str | Log warning |
| `error(message)` | message: str | Log error |
| `debug(message)` | message: str | Log debug message |
| `exception(message)` | message: str | Log exception with traceback |

### luna.workspace.Project

Project management class.

```python
from luna.workspace import Project
```

#### Class Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `create(path, silent=False)` | path: str | Project | Create new project |
| `set(path, silent=False)` | path: str | Project | Set existing project |
| `get()` | None | Project | Get current project |
| `exit()` | None | None | Exit current project |
| `is_project(path)` | path: str | bool | Check if path is project |
| `refresh_recent()` | None | None | Refresh recent list |

#### Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `path` | str | Project path |
| `name` | str | Project name |
| `tag_path` | str | Tag file path |
| `meta_path` | str | Metadata file path |
| `meta_data` | dict | Project metadata |

### luna.workspace.Asset

Asset management class.

```python
from luna.workspace import Asset
```

#### Constructor

```python
Asset(project, name, typ)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| project | Project | Parent project |
| name | str | Asset name |
| typ | str | Asset type (character, prop, etc.) |

#### Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `path` | str | Asset path |
| `name` | str | Asset name |
| `controls` | str | Controls directory |
| `skeleton` | str | Skeleton directory |
| `build` | str | Build directory |
| `rig` | str | Rig scenes directory |
| `settings` | str | Settings directory |
| `weights` | object | Weights directories object |
| `data` | object | Data directories object |

---

## luna_rig Package

Rigging core package.

### luna_rig.MetaNode

Base class for all meta nodes.

```python
from luna_rig.core.meta import MetaNode
```

#### Class Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `create(meta_parent=None)` | meta_parent: MetaNode | MetaNode | Create new meta node |
| `is_metanode(node)` | node: PyNode | bool | Check if node is meta node |
| `list_nodes(of_type=None, by_tag="")` | of_type: class, by_tag: str | list | List meta nodes |

#### Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `pynode` | PyNode | Maya network node |
| `name` | str | Component name |
| `side` | str | Side (l/r/c) |
| `index` | str | Index (00, 01, etc.) |
| `indexed_name` | str | Name with index |
| `suffix` | str | Suffix |
| `meta_type` | str | Full class path |
| `meta_parent` | MetaNode | Parent meta node |
| `meta_children` | list | Child meta nodes |
| `tag` | str | Custom tag |
| `namespace_list` | list | Namespaces |

#### Instance Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `set_meta_parent(parent)` | parent: MetaNode | None | Set parent |
| `get_meta_children(of_type=None, by_tag="")` | of_type: class, by_tag: str | list | Get children |
| `set_tag(tag)` | tag: str | None | Set tag |
| `as_str(name_only=False)` | name_only: bool | str | Get class string |

### luna_rig.Component

Base component class.

```python
from luna_rig.core.component import Component
```

Extends MetaNode with:

#### Class Attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `required_plugins` | list | Plugins to load |

#### Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `settings` | dict | Component settings |
| `util_nodes` | list | Utility nodes |

### luna_rig.AnimComponent

Animatable component class.

```python
from luna_rig.core.component import AnimComponent
```

Extends Component with:

#### Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `root` | Transform | Component root |
| `group_ctls` | Transform | Controls group |
| `group_joints` | Transform | Joints group |
| `group_parts` | Transform | Parts group |
| `group_noscale` | Transform | No-scale group |
| `group_out` | Transform | Output group |
| `controls` | list[Control] | All controls |
| `bind_joints` | list[Joint] | Bind joints |
| `ctl_chain` | list[Joint] | Control chain |
| `character` | Character | Connected character |
| `out_hooks` | list[Hook] | Output hooks |
| `in_hook` | Hook | Input hook |
| `in_hook_index` | int | Input hook index |

#### Instance Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `attach_to_skeleton()` | None | None | Attach to skeleton |
| `detach_from_skeleton()` | None | None | Detach from skeleton |
| `bake_to_skeleton(time_range=None)` | time_range: tuple | None | Bake to skeleton |
| `bake_to_rig(time_range=None)` | time_range: tuple | None | Bake to rig |
| `bake_and_detach(time_range=None)` | time_range: tuple | None | Bake then detach |
| `list_controls(tag=None)` | tag: str | list | List controls |
| `select_controls()` | None | None | Select all controls |
| `key_controls()` | None | None | Key all controls |
| `to_bind_pose()` | None | None | Reset to bind pose |
| `scale_controls(scale_dict)` | scale_dict: dict | None | Scale controls |
| `add_hook(node, name)` | node: PyNode, name: str | Hook | Add output hook |
| `get_hook(index)` | index: int/Enum | Hook | Get hook by index |
| `attach_to_component(comp, hook)` | comp: Component, hook: int | None | Attach to parent |
| `connect_to_character(char=None)` | char: Character | None | Connect to character |
| `remove()` | None | None | Remove component |

### luna_rig.Control

Rig control class.

```python
from luna_rig.core.control import Control
```

#### Class Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `create(...)` | See below | Control | Create control |
| `is_control(node)` | node: PyNode | bool | Check if control |

##### create() Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| name | str | Required | Control name |
| side | str | "c" | Side |
| guide | PyNode | None | Transform to match |
| parent | Control/Transform | None | Parent |
| attributes | str | "trs" | Unlocked attrs |
| delete_guide | bool | False | Delete guide |
| match_pos | bool | True | Match position |
| match_orient | bool | True | Match orientation |
| match_pivot | bool | True | Match pivot |
| color | int | None | Color index |
| offset_grp | bool | True | Create offset |
| joint | bool | False | Create cjnt |
| shape | str | "cube" | Shape name |
| tag | str | "" | Controller tag |
| orient_axis | str | "x" | Orient axis |
| scale | float | 1.0 | Initial scale |

#### Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `transform` | Transform | Control transform |
| `group` | Transform | Root group |
| `offset` | Transform | Last offset |
| `offset_list` | list | All offsets |
| `joint` | Joint | Control joint |
| `tag_node` | Controller | Tag node |
| `shape` | dict | Shape data |
| `color` | int | Color index |
| `bind_pose` | dict | Bind pose |
| `pose` | dict | Current pose |
| `spaces` | list | Available spaces |
| `space_index` | int | Current space |
| `space_name` | str | Current space name |
| `connected_component` | AnimComponent | Parent component |
| `character` | Character | Character |

#### Instance Methods

| Method | Parameters | Returns | Description |
|--------|------------|---------|-------------|
| `scale(value, factor=None)` | value: float, factor: float | None | Scale shape |
| `move_shape(vector)` | vector: list/MVector | None | Move shape CVs |
| `orient_shape(axis)` | axis: str | None | Orient shape |
| `mirror_shape(behaviour=True)` | behaviour: bool | None | Mirror shape |
| `mirror_shape_to_opposite()` | None | None | Mirror to opposite |
| `add_space(target, name, via_matrix=True)` | target: PyNode, name: str, via_matrix: bool | None | Add space |
| `add_world_space(via_matrix=True)` | via_matrix: bool | None | Add world space |
| `switch_space(index, matching=True)` | index: int, matching: bool | None | Switch space |
| `bake_space(space_index, time_range, step=1)` | space_index: int, time_range: tuple, step: int | None | Bake space |
| `bake_custom_space(object, time_range, step=1)` | object: PyNode, time_range: tuple, step: int | None | Bake custom space |
| `write_bind_pose()` | None | None | Store bind pose |
| `to_bind_pose()` | None | None | Reset to bind |
| `set_pose(pose_dict)` | pose_dict: dict | None | Set pose |
| `insert_offset(extra_name)` | extra_name: str | Transform | Insert offset |
| `find_offset(name)` | name: str | Transform | Find offset |
| `add_wire(source)` | source: PyNode | None | Add wire line |
| `add_orient_switch(space, local)` | space: PyNode, local: PyNode | None | Add orient switch |
| `add_driven_pose(driven_dict, driver_attr, driver_value)` | driven_dict: dict, driver_attr: Attr, driver_value: float | None | Add SDK |
| `lock_attrib(exclude_attr, channel_box)` | exclude_attr: str, channel_box: bool | None | Lock attrs |
| `find_opposite()` | None | Control | Find opposite side |
| `mirror_pose(behavior, direction)` | behavior: bool, direction: str | None | Mirror pose |

### luna_rig.components

Component classes.

```python
from luna_rig.components import (
    Character,
    MetaHumanCharacter,
    FKComponent,
    HeadComponent,
    IKComponent,
    IKSplineComponent,
    FKIKComponent,
    BipedLegComponent,
    FKIKSpineComponent,
    RibbonSpineComponent,
    FootComponent,
    TwistComponent,
    HandComponent,
    EyeComponent,
    SimpleComponent,
    RibbonComponent,
    RibbonLipsComponent,
    CorrectiveComponent,
    WireComponent,
    CollarComponent,
    FKDynamicsComponent,
    IKStretchComponent,
    IKSplineStretchComponent
)
```

See [Components Reference](Components.md) for detailed documentation.

### luna_rig.functions

Utility function modules.

```python
import luna_rig.functions.nameFn as nameFn
import luna_rig.functions.jointFn as jointFn
import luna_rig.functions.attrFn as attrFn
import luna_rig.functions.nodeFn as nodeFn
import luna_rig.functions.transformFn as transformFn
import luna_rig.functions.curveFn as curveFn
import luna_rig.functions.animFn as animFn
import luna_rig.functions.rigFn as rigFn
import luna_rig.functions.deformerFn as deformerFn
import luna_rig.functions.outlinerFn as outlinerFn
import luna_rig.functions.surfaceFn as surfaceFn
import luna_rig.functions.rivetFn as rivetFn
import luna_rig.functions.blendShapeFn as blendShapeFn
```

See [Functions Reference](Functions.md) for detailed documentation.

### luna_rig.importexport

Import/export managers.

```python
from luna_rig.importexport.skin import SkinManager
from luna_rig.importexport.blendshape import BlendshapeManager
from luna_rig.importexport.control_shapes import ControlShapesManager
from luna_rig.importexport.deltamush import DeltaMushManager
from luna_rig.importexport.driven_pose import DrivenPoseManager
from luna_rig.importexport.nglayers import ngLayersManager
from luna_rig.importexport.posespace import PoseSpaceManager
```

#### Manager Methods

All managers have these methods:

| Method | Description |
|--------|-------------|
| `export_all()` | Export all in asset |
| `import_all()` | Import all for asset |
| `export_selected()` | Export selected objects |
| `import_selected()` | Import selected objects |
| `export_single(node)` | Export single node |
| `import_single(node)` | Import single node |

---

## luna_builder Package

Visual builder package.

### luna_builder.main_window.BuilderMainWindow

Main builder window.

```python
from luna_builder.main_window import BuilderMainWindow
```

#### Class Methods

| Method | Description |
|--------|-------------|
| `display()` | Show/focus window |
| `close_and_delete()` | Close and cleanup |

#### Instance Properties

| Property | Type | Description |
|----------|------|-------------|
| `current_editor` | NodeEditor | Active editor |
| `current_window` | QMdiSubWindow | Active sub window |

#### Instance Methods

| Method | Description |
|--------|-------------|
| `create_mdi_child()` | Create new editor tab |
| `find_mdi_child(file_name)` | Find tab by file |
| `on_build_new()` | New build |
| `on_build_open()` | Open build |
| `on_build_save()` | Save build |
| `on_build_save_as()` | Save build as |

### luna_builder.editor.graph_executor.GraphExecutor

Build execution.

```python
from luna_builder.editor.graph_executor import GraphExecutor
```

#### Constructor

```python
GraphExecutor(scene)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| scene | NodeScene | Scene to execute |

#### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `execute_graph()` | bool | Execute full graph |
| `execute_step()` | bool | Execute one step |
| `has_next()` | bool | Check if more steps |
| `verify_graph()` | bool | Verify graph validity |

---

## Quick Reference Tables

### Common Create Patterns

```python
# Character
char = Character.create(name="hero")

# Spine
spine = FKIKSpineComponent.create(
    meta_parent=char,
    character=char,
    start_joint="spine_01_jnt",
    end_joint="spine_04_jnt"
)

# Arm
arm = FKIKComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.CHEST,
    character=char,
    side="l",
    name="arm",
    start_joint="l_arm_01_jnt",
    end_joint="l_arm_03_jnt"
)

# Leg
leg = BipedLegComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.HIPS,
    character=char,
    side="l",
    name="leg",
    start_joint="l_leg_01_jnt",
    end_joint="l_leg_03_jnt"
)

# Finalize
char.attach_to_skeleton()
char.save_bind_pose()
```

### Common Query Patterns

```python
# Get all characters
chars = Character.list_nodes(of_type=Character)

# Get components by tag
body_comps = MetaNode.list_nodes(by_tag="body")

# Get character from component
char = component.character

# Get controls from component
ctls = component.list_controls()
fk_ctls = component.list_controls(tag="fk")

# Check if node is control
if Control.is_control(node):
    ctl = Control(node)
```

### Common Import/Export Patterns

```python
# Skin weights
SkinManager.export_all()
SkinManager.import_all()

# Control shapes
ControlShapesManager.export_all()
ControlShapesManager.import_all()
```

### Common Animation Patterns

```python
# Bake to skeleton
char.bake_to_skeleton(time_range=(1, 100))

# Switch FKIK
fkik_comp.switch_fkik(matching=True)

# Bake FKIK
fkik_comp.bake_fkik(source="fk", time_range=(1, 100))

# Switch space
control.switch_space(index=1, matching=True)

# Bake space
control.bake_space(space_index=1, time_range=(1, 100))
```

### Configuration Keys

| Key | Description | Default |
|-----|-------------|---------|
| `logging.level` | Logging level | INFO |
| `project.previous` | Previous project path | None |
| `project.recent` | Recent projects list | [] |
| `naming.templates` | Naming templates dict | {"default": "..."} |
| `naming.profile.template` | Current template | "default" |
| `naming.profile.indexPadding` | Index padding | 2 |
| `naming.profile.startIndex` | Starting index | 0 |
| `rig.io.skin.fileFormat` | Skin export format | "pickle" |
| `rig.display.lineWidth` | Control line width | 2.0 |
| `builder.historyEnabled` | Builder history | True |
| `builder.historySize` | History size | 32 |
