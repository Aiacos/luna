# Functions Reference

This document provides a comprehensive reference for all utility functions available in the `luna_rig.functions` module.

## Table of Contents

1. [nameFn - Naming Functions](#namefn---naming-functions)
2. [jointFn - Joint Functions](#jointfn---joint-functions)
3. [attrFn - Attribute Functions](#attrfn---attribute-functions)
4. [nodeFn - Node Functions](#nodefn---node-functions)
5. [transformFn - Transform Functions](#transformfn---transform-functions)
6. [curveFn - Curve Functions](#curvefn---curve-functions)
7. [animFn - Animation Functions](#animfn---animation-functions)
8. [rigFn - Rig Functions](#rigfn---rig-functions)
9. [deformerFn - Deformer Functions](#deformerfn---deformer-functions)
10. [Other Functions](#other-functions)

## nameFn - Naming Functions

Provides naming operations based on Luna's naming convention.

### Import

```python
import luna_rig.functions.nameFn as nameFn
```

### get_template

Returns the current naming template.

```python
template = nameFn.get_template()
# Default: "{side}_{name}_{suffix}"
```

### generate_name

Generates a unique name following the naming convention.

```python
name = nameFn.generate_name(
    name="arm",           # Base name (can be list)
    side="l",             # Side prefix
    suffix="jnt",         # Suffix
    override_index=None   # Override auto-increment
)
# Result: "l_arm_00_jnt" (increments if exists)

# Using list for name
name = nameFn.generate_name(["arm", "fk"], "l", "ctl")
# Result: "l_arm_fk_00_ctl"
```

### deconstruct_name

Parses a node name into its components.

```python
parts = nameFn.deconstruct_name(node)

# Access parts
parts.side          # "l"
parts.name          # "arm"
parts.index         # "00"
parts.suffix        # "jnt"
parts.indexed_name  # "arm_00"
parts.namespaces    # ["namespace1", "namespace2"]
```

### rename

Renames a node with new naming parts.

```python
nameFn.rename(
    node,
    side="r",           # New side (optional)
    name="leg",         # New name (optional)
    index=None,         # New index (optional)
    suffix=None         # New suffix (optional)
)
```

### add_namespaces

Adds namespaces to a name.

```python
full_name = nameFn.add_namespaces("arm_ctl", ["char", "rig"])
# Result: "char:rig:arm_ctl"
```

### get_namespace

Gets the namespace of a node.

```python
ns = nameFn.get_namespace(node)
# Result: "char:rig:" or ""
```

### remove_digits

Removes all digits from a string.

```python
result = nameFn.remove_digits("arm_01_jnt")
# Result: "arm__jnt"
```

## jointFn - Joint Functions

Provides joint manipulation operations.

### Import

```python
import luna_rig.functions.jointFn as jointFn
```

### joint_chain

Gets a joint chain from start to end joint.

```python
chain = jointFn.joint_chain(
    start_joint="spine_01_jnt",
    end_joint="spine_04_jnt"  # Optional
)
# Returns: [spine_01_jnt, spine_02_jnt, spine_03_jnt, spine_04_jnt]
```

### duplicate_chain

Duplicates a joint chain with new naming.

```python
new_chain = jointFn.duplicate_chain(
    new_joint_name="arm_ctl",       # New name (can be list)
    new_joint_side="l",             # New side
    new_joint_suffix="jnt",         # New suffix
    original_chain=chain,           # Original chain (optional)
    start_joint=None,               # Alternative: start joint
    end_joint=None,                 # Alternative: end joint
    new_parent=parent_grp           # Parent for new chain
)
```

### create_chain

Parents joints in a list to form a chain.

```python
chain = jointFn.create_chain(
    joint_list=[jnt1, jnt2, jnt3],
    reverse=False
)
```

### reverse_chain

Reverses a joint chain hierarchy.

```python
jointFn.reverse_chain(joint_list)
```

### mirror_chain

Mirrors joint chains with proper naming.

```python
jointFn.mirror_chain(chains=[root_joint])
# Uses selection if chains is empty
```

### along_curve

Creates joints along a curve.

```python
joints = jointFn.along_curve(
    curve=curve_node,
    amount=5,
    joint_name="spine",
    joint_side="c",
    joint_suffix="jnt",
    delete_curve=False,
    attach_to_curve=True
)
```

### copy_orient

Copies joint orientation from source to destination.

```python
jointFn.copy_orient(
    source=source_joint,
    destination=[dest_jnt1, dest_jnt2]
)
```

### validate_rotations

Checks for non-zero rotation values on joints.

```python
is_valid = jointFn.validate_rotations(joint_chain)
# Returns: True if all rotations are zero
```

### get_pole_vector

Calculates and creates a pole vector locator for a joint chain.

```python
pole_locator = jointFn.get_pole_vector(joint_chain)
# Returns: SpaceLocator at optimal pole vector position
```

### create_root_joint

Creates a root joint.

```python
root = jointFn.create_root_joint(
    side="c",
    name="root",
    suffix="jnt",
    parent=None,
    children=[]
)
```

### group_joint

Groups a joint with a locator and transform.

```python
result = jointFn.group_joint(
    joint,
    side=None,      # Uses joint's side if None
    name=None,      # Uses joint's name if None
    parent=None     # Uses joint's parent if None
)
# Returns object with: result.group, result.locator, result.joint
```

## attrFn - Attribute Functions

Provides attribute manipulation operations.

### Import

```python
import luna_rig.functions.attrFn as attrFn
```

### add_meta_attr

Adds metaParent attribute to nodes.

```python
attrFn.add_meta_attr(nodes)
# nodes can be single node or list
```

### lock

Locks attributes on a node.

```python
locked = attrFn.lock(
    node,
    attributes=["tx", "ty", "tz"],
    channelBox=False,
    key=False
)
# Returns: list of locked attribute names
```

### get_enums

Gets enum values from an attribute.

```python
enums = attrFn.get_enums(node.space)
# Returns: [("World", 0), ("Parent", 1), ...]
```

### add_divider

Adds a divider attribute to a node.

```python
attrFn.add_divider(node, attr_name="SETTINGS")
```

### transfer_attr

Transfers attributes from source to destination.

```python
attr_alias = attrFn.transfer_attr(
    source=source_node,
    destination=dest_node,
    attr_list=None,     # Uses keyable attrs if None
    connect=False       # Connect source to destination
)
# Returns: {source_attr: dest_attr, ...}
```

### connect_transform_attrs

Connects translate, rotate, scale between transforms.

```python
attrFn.connect_transform_attrs(source, destination)
```

## nodeFn - Node Functions

Provides node creation and manipulation operations.

### Import

```python
import luna_rig.functions.nodeFn as nodeFn
```

### create

Creates a node with proper naming.

```python
node = nodeFn.create(
    node_type="transform",
    name="arm",           # Can be list: ["arm", "fk"]
    side="l",
    suffix="grp",
    p=parent_node         # Optional parent
)
```

## transformFn - Transform Functions

Provides transform manipulation operations.

### Import

```python
import luna_rig.functions.transformFn as transformFn
```

### WorldVector

Enum for world axis vectors.

```python
transformFn.WorldVector.X.value  # [1, 0, 0]
transformFn.WorldVector.Y.value  # [0, 1, 0]
transformFn.WorldVector.Z.value  # [0, 0, 1]
```

### get_position

Gets world position of a transform.

```python
pos = transformFn.get_position(node)
# Returns: MVector
```

### get_vector

Gets vector between two transforms.

```python
vec = transformFn.get_vector(start, end)
# Returns: MVector
```

### matrix_to_list

Converts matrix to list.

```python
matrix_list = transformFn.matrix_to_list(matrix)
# Returns: [16 float values]
```

## curveFn - Curve Functions

Provides curve manipulation operations.

### Import

```python
import luna_rig.functions.curveFn as curveFn
```

### curve_from_points

Creates a NURBS curve from points.

```python
curve = curveFn.curve_from_points(
    name="spine_crv",
    degree=3,
    points=[[0,0,0], [0,1,0], [0,2,0]]
)
```

### get_curve_data

Gets curve shape data as dictionary.

```python
data = curveFn.get_curve_data(curve_shape)
# Returns: {"points": [...], "knots": [...], "degree": 3}
```

### mirror_shape

Mirrors curve shape.

```python
curveFn.mirror_shape(
    transform,
    behaviour=True,
    flip=False,
    flip_across="yz"
)
```

## animFn - Animation Functions

Provides animation-related operations.

### Import

```python
import luna_rig.functions.animFn as animFn
```

### get_playback_range

Gets current playback range.

```python
start, end = animFn.get_playback_range()
# Returns: (start_frame, end_frame)
```

### bake_animation

Bakes animation on nodes.

```python
animFn.bake_animation(nodes, time_range=(1, 100))
```

## rigFn - Rig Functions

Provides rig-related utility operations.

### Import

```python
import luna_rig.functions.rigFn as rigFn
```

### get_build_character

Gets the current build character.

```python
character = rigFn.get_build_character()
# Returns: Character or None
```

### list_controls

Lists all controls in the scene.

```python
controls = rigFn.list_controls()
# Returns: list of Control instances
```

### get_param_ctl_locator

Creates a locator for parameter control positioning.

```python
locator = rigFn.get_param_ctl_locator(
    side="l",
    anchor=joint,
    move_axis="x"
)
```

### delete_all_meta_nodes

Deletes all meta nodes in the scene.

```python
rigFn.delete_all_meta_nodes()
```

### set_node_selectable

Sets node selectability.

```python
rigFn.set_node_selectable(node, value=True)
```

### selected_control_bind_pose

Resets selected controls to bind pose.

```python
rigFn.selected_control_bind_pose()
```

### asset_bind_pose

Resets entire asset to bind pose.

```python
rigFn.asset_bind_pose()
```

## deformerFn - Deformer Functions

Provides deformer-related operations.

### Import

```python
import luna_rig.functions.deformerFn as deformerFn
```

### get_deformer

Gets a deformer from a node's history.

```python
skin = deformerFn.get_deformer(geometry, "skinCluster")
```

### list_deformers

Lists deformers of a specific type.

```python
skins = deformerFn.list_deformers("skinCluster", parent_node)
```

### is_painted

Checks if a deformer has weights initialized.

```python
if deformerFn.is_painted(skin_cluster):
    # Has weights
    pass
```

### create_skin_cluster

Creates a skin cluster.

```python
skin = deformerFn.create_skin_cluster(joints, geometry)
```

### create_blend_shape

Creates a blend shape.

```python
bs = deformerFn.create_blend_shape(targets, base)
```

## Other Functions

### surfaceFn

NURBS surface operations.

```python
import luna_rig.functions.surfaceFn as surfaceFn
```

### rivetFn

Rivet/follicle creation.

```python
import luna_rig.functions.rivetFn as rivetFn
```

### blendShapeFn

Blendshape utilities.

```python
import luna_rig.functions.blendShapeFn as blendShapeFn
```

### cmuscleFn

cMuscle system integration.

```python
import luna_rig.functions.cmuscleFn as cmuscleFn
```

### outlinerFn

Outliner display settings.

```python
import luna_rig.functions.outlinerFn as outlinerFn

# Set outliner color
outlinerFn.set_color(node, color=17)
outlinerFn.set_color(node, color=[1.0, 0.5, 0.0])
```

### common_setup

Common rig setup patterns.

```python
import luna_rig.functions.common_setup as common_setup
```

### apiFn

Maya API utilities.

```python
import luna_rig.functions.apiFn as apiFn
```

### asset_files

Asset file operations.

```python
import luna_rig.functions.asset_files as asset_files
```

## Function Usage Examples

### Example 1: Creating a Joint Chain with Controls

```python
import luna_rig
import luna_rig.functions.jointFn as jointFn
import luna_rig.functions.nameFn as nameFn
import luna_rig.functions.attrFn as attrFn

# Get joint chain
chain = jointFn.joint_chain("spine_01_jnt", "spine_04_jnt")

# Validate rotations
if not jointFn.validate_rotations(chain):
    print("Warning: Non-zero rotations found")

# Duplicate chain for controls
ctl_chain = jointFn.duplicate_chain(
    new_joint_name="spine_ctl",
    new_joint_side="c",
    original_chain=chain,
    new_parent=control_grp
)

# Add meta attributes
for jnt in chain + ctl_chain:
    attrFn.add_meta_attr(jnt)
```

### Example 2: Working with Naming

```python
import luna_rig.functions.nameFn as nameFn

# Generate unique names
grp_name = nameFn.generate_name("arm", "l", "grp")
ctl_name = nameFn.generate_name(["arm", "fk"], "l", "ctl")

# Parse existing name
node = pm.PyNode("l_arm_fk_00_ctl")
parts = nameFn.deconstruct_name(node)
print(f"Side: {parts.side}, Name: {parts.name}, Index: {parts.index}")

# Rename with new side
nameFn.rename(node, side="r")
```

### Example 3: Setting Up Deformers

```python
import luna_rig.functions.deformerFn as deformerFn

# Create skin cluster
joints = pm.ls("*_bind_jnt", type="joint")
geometry = pm.PyNode("body_geo")
skin = deformerFn.create_skin_cluster(joints, geometry)

# Check if painted
if deformerFn.is_painted(skin):
    print("Skin has weights")
```
