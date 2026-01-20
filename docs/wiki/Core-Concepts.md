# Core Concepts

This document explains the fundamental concepts that form the foundation of the Luna rigging framework.

## Table of Contents

1. [Meta-Node System](#meta-node-system)
2. [MetaNode Class](#metanode-class)
3. [Component Class](#component-class)
4. [AnimComponent Class](#animcomponent-class)
5. [Control Class](#control-class)
6. [Hook System](#hook-system)
7. [Shape Manager](#shape-manager)

## Meta-Node System

Luna's rig system is built on the Meta-Node pattern, based on the 2015 GDC talk by David Hunt and Forrest Soderlind.

### Core Principles

1. **Network Node Storage**: Every rig element is represented by a Maya network node
2. **Class Path Storage**: The `metaType` attribute stores the full Python class path
3. **Dynamic Instantiation**: When accessing a node, Luna evaluates the class string to return the correct instance type
4. **Parent-Child Relationships**: Maintained via `metaParent` and `metaChildren` attributes

### How It Works

```python
# When creating a component
instance = FKComponent.create(...)

# Luna creates a network node with:
# - metaType = "luna_rig.components.fk_component.FKComponent"
# - metaParent (message attribute)
# - metaChildren (multi message attribute)
# - tag (string attribute)

# When retrieving a component from an existing node
node = pm.PyNode("l_arm_00_meta")
component = MetaNode(node)
# Returns an FKComponent instance, not a generic MetaNode
```

### Benefits

- Query and manipulate rigs programmatically
- Maintain rig structure information in scene
- Enable tools to understand rig hierarchy
- Support serialization and reconstruction

## MetaNode Class

The base class for all rig elements.

### Creating a MetaNode

```python
from luna_rig.core.meta import MetaNode

# Create new MetaNode
meta_node = MetaNode.create(meta_parent=None)

# Create from existing network node
meta_node = MetaNode("my_network_node")
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `pynode` | `pm.PyNode` | The underlying Maya network node |
| `name` | `str` | Component name (extracted from naming convention) |
| `side` | `str` | Component side (c, l, r) |
| `index` | `str` | Component index (00, 01, etc.) |
| `indexed_name` | `str` | Name with index (e.g., "arm_00") |
| `suffix` | `str` | Component suffix |
| `meta_type` | `str` | Full class path string |
| `meta_parent` | `MetaNode` | Parent MetaNode instance |
| `meta_children` | `list` | List of child MetaNode instances |
| `tag` | `str` | Custom tag string |
| `namespace_list` | `list` | List of namespaces |

### Key Methods

```python
# Check if a node is a MetaNode
if MetaNode.is_metanode(node):
    meta = MetaNode(node)

# List all meta nodes in scene
all_nodes = MetaNode.list_nodes()

# Filter by type
characters = MetaNode.list_nodes(of_type=luna_rig.components.Character)

# Filter by tag
body_components = MetaNode.list_nodes(by_tag="body")

# Set parent relationship
meta_node.set_meta_parent(parent_node)

# Get children with filters
children = meta_node.get_meta_children(of_type=FKComponent, by_tag="body")

# Set custom tag
meta_node.set_tag("my_custom_tag")

# Get class path string
class_str = meta_node.as_str()
# Returns: "luna_rig.core.meta.MetaNode"

name_only = meta_node.as_str(name_only=True)
# Returns: "MetaNode"
```

### Getter Methods

All properties have corresponding getter methods for use with the Builder:

```python
meta_node.get_name()
meta_node.get_side()
meta_node.get_index()
meta_node.get_meta_type()
meta_node.get_tag()
meta_node.get_meta_parent()
```

## Component Class

Extends MetaNode with component-specific functionality.

### Class Hierarchy

```
MetaNode
└── Component
    ├── Character
    └── AnimComponent
        └── (all body components)
```

### Creating a Component

```python
from luna_rig.core.component import Component

# Create component
component = Component.create(
    meta_parent=parent_component,
    side="c",
    name="my_component",
    tag="custom"
)
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `settings` | `dict` | Component settings attributes {attr: value} |
| `util_nodes` | `list` | Connected utility nodes |

### Class Attributes

```python
class MyComponent(Component):
    required_plugins = ["myPlugin"]  # Plugins required by this component
```

### Key Methods

```python
# Load required plugins
Component.load_required_plugins()

# Attach to another component
component.attach_to_component(other_component)

# Store a setting attribute
component._store_settings(some_attribute)

# Store utility nodes
component._store_util_nodes([node1, node2])

# Remove component
component.remove()
```

## AnimComponent Class

The primary component class for animation rig elements. Extends Component with control and joint management.

### Standard Hierarchy

Every AnimComponent creates this hierarchy:

```
{name}_comp/           # Component root (root property)
├── {name}_ctls/       # Controls group (group_ctls)
├── {name}_jnts/       # Joint group (group_joints)
├── {name}_parts/      # Parts group (group_parts)
│   └── {name}_noscale/  # No-scale group (group_noscale)
└── {name}_out/        # Output/hooks group (group_out)
```

### Creating an AnimComponent

```python
from luna_rig.core.component import AnimComponent

anim_comp = AnimComponent.create(
    meta_parent=character,
    side="l",
    name="arm",
    hook=0,
    character=character,
    tag="body"
)
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `root` | `Transform` | Component root group |
| `group_ctls` | `Transform` | Controls group |
| `group_joints` | `Transform` | Joints group |
| `group_parts` | `Transform` | Parts group |
| `group_noscale` | `Transform` | No-scale group |
| `group_out` | `Transform` | Output/hooks group |
| `controls` | `list[Control]` | All component controls |
| `bind_joints` | `list[Joint]` | Bind skeleton joints |
| `ctl_chain` | `list[Joint]` | Control joint chain |
| `character` | `Character` | Connected character |
| `out_hooks` | `list[Hook]` | Output attachment points |
| `in_hook` | `Hook` | Input attachment point |
| `in_hook_index` | `int` | Index of input hook |

### Key Methods

#### Skeleton Operations

```python
# Attach component to skeleton
anim_comp.attach_to_skeleton()

# Detach from skeleton
anim_comp.detach_from_skeleton()

# Bake animation to skeleton
anim_comp.bake_to_skeleton(time_range=(1, 100))

# Bake then detach
anim_comp.bake_and_detach(time_range=(1, 100))
```

#### Control Operations

```python
# List controls (optionally by tag)
all_ctls = anim_comp.list_controls()
fk_ctls = anim_comp.list_controls(tag="fk")

# Select all controls
anim_comp.select_controls()

# Key all controls
anim_comp.key_controls()

# Reset to bind pose
anim_comp.to_bind_pose()

# Scale controls
anim_comp.scale_controls({control: 1.5})
```

#### Hook Operations

```python
# Add output hook
hook = anim_comp.add_hook(node, "hook_name")

# Get hook by index
hook = anim_comp.get_hook(0)
# Or using Enum
hook = anim_comp.get_hook(SomeComponent.Hooks.CHEST)
```

#### Connection Operations

```python
# Attach to parent component
anim_comp.attach_to_component(parent_comp, hook_index=0)

# Connect to character
anim_comp.connect_to_character(character_component=char)

# Remove component
anim_comp.remove()
```

### Internal Storage Methods

```python
# Store bind joints
anim_comp._store_bind_joints(joint_list)

# Store control chain
anim_comp._store_ctl_chain(joint_list)

# Store controls
anim_comp._store_controls(control_list)
```

## Control Class

Represents animator-facing rig controls with automatic hierarchy creation.

### Control Hierarchy

```
{name}_grp                    # Group (Control.group)
└── {name}_ofs (optional)     # Offset (Control.offset)
    └── {name}_ctl            # Transform (Control.transform)
        └── {name}_cjnt       # Joint (Control.joint, optional)
```

### Creating a Control

```python
from luna_rig.core.control import Control

ctl = Control.create(
    name="arm_fk",
    side="l",
    guide=joint,                # Transform to match
    parent=parent_ctl,          # Parent control or transform
    attributes="tr",            # Unlocked attributes
    delete_guide=False,
    match_pos=True,
    match_orient=True,
    match_pivot=True,
    color=None,                 # Auto based on side
    offset_grp=True,
    joint=True,                 # Create control joint
    shape="cube",
    tag="fk",
    orient_axis="x",
    scale=1.0
)
```

### Accessing Existing Control

```python
# From transform name
ctl = Control("l_arm_fk_00_ctl")

# From Controller node
ctl = Control(controller_node)

# From shape
ctl = Control(nurbs_curve_shape)
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `transform` | `Transform` | The control transform |
| `group` | `Transform` | Root group |
| `offset` | `Transform` | Last offset node |
| `offset_list` | `list` | All offset nodes |
| `joint` | `Joint` | Control joint (if created) |
| `tag_node` | `Controller` | Maya controller tag node |
| `shape` | `dict` | Current shape data |
| `color` | `int` | Color index |
| `bind_pose` | `dict` | Stored bind pose |
| `pose` | `dict` | Current pose |
| `spaces` | `list` | Available spaces |
| `space_index` | `int` | Current space index |
| `space_name` | `str` | Current space name |
| `connected_component` | `AnimComponent` | Parent component |
| `character` | `Character` | Connected character |

### Shape Operations

```python
# Set shape from library
ctl.shape = "circle"

# Scale shape
ctl.scale(2.0)
ctl.scale(1.5, factor=0.8)

# Move shape CVs
ctl.move_shape([0, 1, 0])

# Orient shape
ctl.orient_shape("x")  # or "y", "z", "-x", "-y", "-z"

# Mirror shape
ctl.mirror_shape(behaviour=True)

# Mirror to opposite side control
ctl.mirror_shape_to_opposite()
```

### Space Switching

```python
# Add space targets
ctl.add_space(target_transform, "Parent", via_matrix=True)
ctl.add_world_space(via_matrix=True)

# Switch space with matching
ctl.switch_space(index=1, matching=True)

# Bake space switch
ctl.bake_space(space_index=1, time_range=(1, 100))

# Bake to custom object
ctl.bake_custom_space(space_object, time_range=(1, 100))
```

### Pose Operations

```python
# Write current pose as bind pose
ctl.write_bind_pose()

# Reset to bind pose
ctl.to_bind_pose()

# Set pose from dict
ctl.set_pose({"tx": 0, "ty": 1, "tz": 0})
```

### Offset Operations

```python
# Insert extra offset
new_offset = ctl.insert_offset(extra_name="extra")

# Find offset by name
offset = ctl.find_offset("extra")
```

### Additional Features

```python
# Add wire line to source
ctl.add_wire(source_transform)

# Add orient switch
ctl.add_orient_switch(space_target, local_parent)

# Add driven pose (SDK)
ctl.add_driven_pose(
    driven_dict={"rx": 45, "ry": 0},
    driver_attr=driver.attr,
    driver_value=1.0
)

# Lock attributes
ctl.lock_attrib(exclude_attr="tr", channel_box=False)

# Find opposite side control
opposite = ctl.find_opposite()

# Mirror pose
ctl.mirror_pose(behavior=True, direction="source")

# Check if node is a control
if Control.is_control(node):
    ctl = Control(node)
```

## Hook System

Hooks are attachment points for connecting components.

### How Hooks Work

1. Each AnimComponent can define output hooks
2. Child components connect to parent hooks via `attach_to_component()`
3. Hooks are indexed using integers or Enum classes

### Defining Hooks

```python
class FKIKSpineComponent(SpineComponent):
    class Hooks(enumFn.Enum):
        ROOT = 0
        HIPS = 1
        MID = 2
        CHEST = 3
```

### Creating Hooks

```python
# Hooks are created internally by AnimComponent
hook = anim_component.add_hook(transform_node, "hook_name")
```

### Using Hooks

```python
# Get hook by index
hook = spine.get_hook(0)

# Get hook using Enum
hook = spine.get_hook(spine.Hooks.CHEST)

# Attach child component
arm = FKIKComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.CHEST,  # or just 3
    ...
)
```

### Hook Class Properties

| Property | Type | Description |
|----------|------|-------------|
| `transform` | `Transform` | Hook transform node |
| `as_object` | `DependNode` | Object hook is constrained to |
| `component` | `AnimComponent` | Parent component |
| `children` | `list` | Connected child MetaNodes |
| `index` | `int` | Index in parent's hook list |

## Shape Manager

Manages control shapes from the shape library.

### Using ShapeManager

```python
from luna_rig.core.shape_manager import ShapeManager

# Set shape from library
ShapeManager.set_shape_from_lib(control.transform, "cube")

# Get current shape data
shape_data = ShapeManager.get_shapes(control.transform)

# Apply shape data
ShapeManager.apply_shape(transform, shape_list)

# Save shape to library
ShapeManager.save_shape(transform, "my_custom_shape")
```

### Copy/Paste Operations

```python
# Copy shape (uses selection if no arg)
ShapeManager.copy_shape(transform)

# Paste shape
ShapeManager.paste_shape([transform1, transform2])

# Copy color
ShapeManager.copy_color()

# Paste color
ShapeManager.paste_color()
```

### Color Operations

```python
# Set color by index
ShapeManager.set_color(nodes, 17)

# Set color by name
ShapeManager.set_color(nodes, "blue")

# Get color index
color = ShapeManager.get_color(transform)
```

### Line Width

```python
# Set line width
ShapeManager.set_line_width(transform, 2.0)
```

### Available Shapes

Common shapes in the library:
- `cube`, `circle`, `circleCrossed`, `circle_pointed`
- `root`, `hips`, `chest`
- `poleVector`, `small_cog`
- `markerDiamond`, `character`
- And many more in `res/shapes/`

### Adding Custom Shapes

1. Create a NURBS curve in Maya
2. Save to library:
   ```python
   ShapeManager.save_shape(my_curve, "my_custom_shape")
   ```
3. Use in Control creation:
   ```python
   Control.create(..., shape="my_custom_shape")
   ```
