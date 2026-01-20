# Naming Convention

Luna uses a flexible, template-based naming convention system. This document explains how naming works and how to customize it for your pipeline.

## Table of Contents

1. [Overview](#overview)
2. [Default Template](#default-template)
3. [Name Structure](#name-structure)
4. [Configuration](#configuration)
5. [Using nameFn](#using-namefn)
6. [Standard Suffixes](#standard-suffixes)
7. [Side Conventions](#side-conventions)
8. [Examples](#examples)

## Overview

Luna's naming system:

- Uses configurable templates
- Automatically increments indices to avoid duplicates
- Supports side prefixes (l/r/c)
- Handles namespaces properly
- Provides functions for name generation and parsing

## Default Template

The default naming template is:

```
{side}_{name}_{suffix}
```

This produces names like:
- `l_arm_00_jnt`
- `c_spine_01_ctl`
- `r_leg_fk_00_grp`

## Name Structure

A typical Luna name has these components:

```
l_arm_fk_00_ctl
│ │    │  │  └── suffix (ctl, jnt, grp, etc.)
│ │    │  └───── index (00, 01, 02, etc.)
│ │    └──────── name parts (arm_fk)
│ └───────────── name (arm)
└─────────────── side (l, r, c)
```

### Components

| Component | Description | Example |
|-----------|-------------|---------|
| **side** | Side prefix indicating left/right/center | `l`, `r`, `c` |
| **name** | Base name, can include multiple parts | `arm`, `arm_fk`, `spine_twist` |
| **index** | Auto-incrementing number with padding | `00`, `01`, `02` |
| **suffix** | Type indicator | `jnt`, `ctl`, `grp`, `loc` |

## Configuration

### Viewing Current Settings

```python
from luna import Config, NamingVars

# Get current template
templates = Config.get(NamingVars.templates_dict)
current_template = Config.get(NamingVars.current_template)
start_index = Config.get(NamingVars.start_index)
padding = Config.get(NamingVars.index_padding)

print(f"Template: {templates[current_template]}")
print(f"Start Index: {start_index}")
print(f"Padding: {padding}")
```

### Customizing Templates

```python
from luna import Config, NamingVars

# Define custom templates
templates = {
    "default": "{side}_{name}_{suffix}",
    "unreal": "{name}_{side}_{suffix}",
    "short": "{side}{name}_{suffix}"
}

# Set templates
Config.set(NamingVars.templates_dict, templates)

# Use a specific template
Config.set(NamingVars.current_template, "unreal")
```

### Index Configuration

```python
from luna import Config, NamingVars

# Set starting index (default: 0)
Config.set(NamingVars.start_index, 1)

# Set index padding (default: 2)
Config.set(NamingVars.index_padding, 3)  # Results in 001, 002, etc.
```

## Using nameFn

The `nameFn` module provides all naming operations.

### Import

```python
import luna_rig.functions.nameFn as nameFn
```

### get_template()

Returns the current naming template.

```python
template = nameFn.get_template()
# Returns: "{side}_{name}_{suffix}"
```

### generate_name()

Generates a unique name following the convention.

```python
# Basic usage
name = nameFn.generate_name(
    name="arm",
    side="l",
    suffix="jnt"
)
# Returns: "l_arm_00_jnt"

# With multiple name parts
name = nameFn.generate_name(
    name=["arm", "fk"],
    side="l",
    suffix="ctl"
)
# Returns: "l_arm_fk_00_ctl"

# With specific index
name = nameFn.generate_name(
    name="spine",
    side="c",
    suffix="jnt",
    override_index="01"
)
# Returns: "c_spine_01_jnt"
```

**Auto-increment**: If a name already exists, the function automatically increments the index until it finds a unique name.

### deconstruct_name()

Parses a node name into its components.

```python
import pymel.core as pm

node = pm.PyNode("l_arm_fk_00_ctl")
parts = nameFn.deconstruct_name(node)

print(parts.side)          # "l"
print(parts.name)          # "arm_fk"
print(parts.index)         # "00"
print(parts.suffix)        # "ctl"
print(parts.indexed_name)  # "arm_fk_00"
print(parts.namespaces)    # []
```

### rename()

Renames a node with new naming parts.

```python
import pymel.core as pm

node = pm.PyNode("l_arm_00_jnt")

# Change side
nameFn.rename(node, side="r")
# Now: "r_arm_00_jnt"

# Change name
nameFn.rename(node, name="leg")
# Now: "r_leg_00_jnt"

# Change multiple parts
nameFn.rename(node, side="l", name="arm", index="01", suffix="ctl")
# Now: "l_arm_01_ctl"
```

### add_namespaces()

Adds namespaces to a name.

```python
# From string
full_name = nameFn.add_namespaces("arm_ctl", "hero")
# Returns: "hero:arm_ctl"

# From list
full_name = nameFn.add_namespaces("arm_ctl", ["hero", "rig"])
# Returns: "hero:rig:arm_ctl"

# From PyNode
full_name = nameFn.add_namespaces(pm.PyNode("arm_ctl"), "hero")
# Returns: "hero:arm_ctl"
```

### get_namespace()

Gets the namespace string from a node.

```python
ns = nameFn.get_namespace("hero:rig:arm_ctl")
# Returns: "hero:rig:"

ns = nameFn.get_namespace("arm_ctl")
# Returns: ""
```

### remove_digits()

Removes all digit characters from a string.

```python
result = nameFn.remove_digits("arm_01_jnt")
# Returns: "arm__jnt"
```

## Standard Suffixes

Luna uses these standard suffixes:

### Joints

| Suffix | Description |
|--------|-------------|
| `jnt` | Regular joint |
| `bnd` | Bind joint (deformation) |
| `drv` | Driver joint |
| `cjnt` | Control joint |

### Controls

| Suffix | Description |
|--------|-------------|
| `ctl` | Control transform |
| `grp` | Group node |
| `ofs` | Offset group |

### Deformers

| Suffix | Description |
|--------|-------------|
| `skin` | Skin cluster |
| `bs` | Blend shape |
| `clst` | Cluster |
| `ffd` | Lattice (FFD) |

### Constraints

| Suffix | Description |
|--------|-------------|
| `prc` | Parent constraint |
| `ptc` | Point constraint |
| `orc` | Orient constraint |
| `scc` | Scale constraint |
| `aic` | Aim constraint |
| `pvc` | Pole vector constraint |

### IK

| Suffix | Description |
|--------|-------------|
| `ikh` | IK handle |
| `eff` | IK effector |
| `crv` | Spline curve |

### Utility

| Suffix | Description |
|--------|-------------|
| `loc` | Locator |
| `null` | Empty transform |
| `comp` | Component root |
| `out` | Output node |
| `meta` | Meta node |

### Math Nodes

| Suffix | Description |
|--------|-------------|
| `md` | Multiply divide |
| `pma` | Plus minus average |
| `rev` | Reverse |
| `clmp` | Clamp |
| `cnd` | Condition |
| `rmp` | Remap |

### Surfaces

| Suffix | Description |
|--------|-------------|
| `geo` | Geometry |
| `nrb` | NURBS surface |
| `crv` | NURBS curve |
| `fol` | Follicle |

## Side Conventions

### Standard Sides

| Side | Full Name | Color Convention |
|------|-----------|------------------|
| `l` | Left | Blue (6, 18) |
| `r` | Right | Red (4, 13) |
| `c` | Center | Yellow (17, 22) |

### Opposite Side Mapping

```python
from luna import static

# Get opposite side
opposite = static.OppositeSide["l"].value  # "r"
opposite = static.OppositeSide["r"].value  # "l"
opposite = static.OppositeSide["c"].value  # "c"
```

### Side Detection

Controls automatically use side-based coloring:

```python
import luna_rig

# Left side control - automatically blue
ctl = luna_rig.Control.create(side="l", ...)

# Right side control - automatically red
ctl = luna_rig.Control.create(side="r", ...)

# Center control - automatically yellow
ctl = luna_rig.Control.create(side="c", ...)
```

## Examples

### Complete Naming Example

```python
import pymel.core as pm
import luna_rig.functions.nameFn as nameFn

# Generate joint names
spine_joints = []
for i in range(5):
    name = nameFn.generate_name("spine", "c", "jnt", override_index=str(i).zfill(2))
    jnt = pm.createNode("joint", n=name)
    spine_joints.append(jnt)
# Creates: c_spine_00_jnt, c_spine_01_jnt, c_spine_02_jnt, c_spine_03_jnt, c_spine_04_jnt

# Generate control names
for jnt in spine_joints:
    parts = nameFn.deconstruct_name(jnt)
    ctl_name = nameFn.generate_name(
        [parts.name, "fk"],
        parts.side,
        "ctl",
        override_index=parts.index
    )
    # Creates: c_spine_fk_00_ctl, c_spine_fk_01_ctl, etc.
```

### Working with Namespaces

```python
import pymel.core as pm
import luna_rig.functions.nameFn as nameFn

# Reference a character
pm.createReference("/path/to/hero.ma", namespace="hero")

# Work with namespaced objects
node = pm.PyNode("hero:l_arm_00_jnt")
parts = nameFn.deconstruct_name(node)

print(parts.namespaces)  # ["hero"]
print(parts.name)        # "arm"

# Generate name with namespace
name = nameFn.generate_name("arm_fk", "l", "ctl")
full_name = nameFn.add_namespaces(name, parts.namespaces)
# Returns: "hero:l_arm_fk_00_ctl"
```

### Custom Pipeline Integration

```python
from luna import Config, NamingVars

# Configure for Unreal pipeline
def configure_unreal_naming():
    Config.set(NamingVars.templates_dict, {
        "default": "{side}_{name}_{suffix}",
        "unreal": "{name}_{side}_{suffix}"
    })
    Config.set(NamingVars.current_template, "unreal")
    Config.set(NamingVars.start_index, 0)
    Config.set(NamingVars.index_padding, 2)

configure_unreal_naming()

# Now names will be like: arm_l_00_jnt instead of l_arm_00_jnt
```

### Batch Renaming

```python
import pymel.core as pm
import luna_rig.functions.nameFn as nameFn

def batch_rename_side(old_side, new_side):
    """Rename all joints from one side to another."""
    for jnt in pm.ls(type="joint"):
        try:
            parts = nameFn.deconstruct_name(jnt)
            if parts.side == old_side:
                nameFn.rename(jnt, side=new_side)
        except:
            pass

# Example: Change all left side joints to right
batch_rename_side("l", "r")
```
