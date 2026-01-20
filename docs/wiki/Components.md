# Components Reference

This document provides a comprehensive reference for all rig components available in Luna.

## Table of Contents

1. [Component Overview](#component-overview)
2. [Character](#character)
3. [Spine Components](#spine-components)
4. [FK Components](#fk-components)
5. [IK Components](#ik-components)
6. [FKIK Components](#fkik-components)
7. [Body Components](#body-components)
8. [Special Components](#special-components)
9. [Creating Custom Components](#creating-custom-components)

## Component Overview

All components in Luna inherit from the base classes:

```
MetaNode
└── Component
    ├── Character
    └── AnimComponent
        ├── FKComponent
        │   └── HeadComponent
        ├── IKComponent
        │   └── IKSplineComponent
        ├── FKIKComponent
        │   └── BipedLegComponent
        ├── FKIKSpineComponent
        ├── RibbonSpineComponent
        ├── FootComponent
        ├── TwistComponent
        ├── HandComponent
        ├── EyeComponent
        ├── SimpleComponent
        ├── RibbonComponent
        ├── CorrectiveComponent
        ├── WireComponent
        ├── CollarComponent
        ├── FKDynamicsComponent
        └── Stretch Components
```

## Character

The root component that creates the main rig hierarchy.

### Import

```python
from luna_rig.components import Character
```

### Creation

```python
character = Character.create(
    meta_parent=None,      # Usually None for character
    name="my_character",   # Character name
    tag="character"        # Optional tag
)
```

### Character Hierarchy

```
rig/                           # Top rig node
└── c_character_00_ctl         # Root control
    ├── control_rig/           # All rig components go here
    ├── deformation_rig/       # Skeleton
    ├── geometry/              # Character geometry
    ├── locators/              # Utility locators
    │   └── world_space        # World space locator
    └── util/                  # Utility nodes
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `root_control` | `Control` | Main character control |
| `control_rig` | `Transform` | Control rig group |
| `deformation_rig` | `Transform` | Skeleton group |
| `geometry_grp` | `Transform` | Geometry group |
| `locators_grp` | `Transform` | Locators group |
| `world_locator` | `Locator` | World space locator |
| `util_grp` | `Transform` | Utility nodes group |
| `root_motion` | `Joint` | Root motion joint |
| `clamped_size` | `float` | Size for control scaling |
| `controls` | `list` | All controls in rig |
| `bind_joints` | `list` | All bind joints |

### Key Methods

```python
# Find character by name
hero = Character.find("hero")

# List all controls
all_ctls = character.list_controls()
fk_ctls = character.list_controls(tag="fk")

# List bind joints
bind_joints = character.list_bind_joints()
bind_joints = character.list_bind_joints(by_tag="body")

# List geometry
geometry = character.list_geometry()

# Get character size
height = character.get_size(axis="y")

# Bind pose operations
character.save_bind_pose()
character.to_bind_pose()

# Skeleton operations
character.attach_to_skeleton()
character.bake_to_skeleton(time_range=(1, 100))
character.bake_and_detach(time_range=(1, 100))

# Root motion
character.add_root_motion(
    follow_object=hips_control,
    root_joint=None,  # Creates if None
    children=[],
    up_axis="y"
)

# Selection sets
character.create_controls_set(name="controls_set")
character.create_bind_joints_set(name="bind_joints_set")
character.create_skeleton_set(name="skeleton_set")
character.create_geometry_set(name="geometry_set")
character.create_selection_sets()  # Creates all sets

# Publish mode (hide non-animator elements)
character.set_publish_mode(True)

# Remove rig
character.remove(time_range=(1, 100))
```

### MetaHumanCharacter

Special character variant for MetaHuman rigs.

```python
from luna_rig.components import MetaHumanCharacter

mh_char = MetaHumanCharacter.create(
    name="meta_human_character",
    tag="character"
)
```

## Spine Components

### FKIKSpineComponent

Advanced spine with FK and IK controls.

```python
from luna_rig.components import FKIKSpineComponent

spine = FKIKSpineComponent.create(
    meta_parent=character,
    hook=0,
    character=character,
    side="c",
    name="spine",
    start_joint="spine_01_jnt",
    end_joint="spine_04_jnt",
    up_axis="y",
    forward_axis="x",
    tag="body"
)
```

#### Hooks

```python
class Hooks(Enum):
    ROOT = 0
    HIPS = 1
    MID = 2
    CHEST = 3
```

#### Properties

| Property | Description |
|----------|-------------|
| `root_control` | Root/COG control |
| `hips_control` | Hips control |
| `mid_control` | Mid spine control |
| `chest_control` | Chest control |
| `fk1_control` | First FK control |
| `fk2_control` | Second FK control |

### RibbonSpineComponent

Ribbon-based spine setup.

```python
from luna_rig.components import RibbonSpineComponent

spine = RibbonSpineComponent.create(
    meta_parent=character,
    hook=0,
    character=character,
    side="c",
    name="spine",
    start_joint="spine_01_jnt",
    end_joint="spine_04_jnt",
    tag="body"
)
```

## FK Components

### FKComponent

Pure FK chain component.

```python
from luna_rig.components import FKComponent

fk = FKComponent.create(
    meta_parent=parent_comp,
    hook=0,
    character=character,
    side="l",
    name="arm_fk",
    start_joint="l_arm_01_jnt",
    end_joint="l_arm_03_jnt",
    add_end_ctl=True,       # Control on end joint
    lock_translate=True,     # Lock translation on controls
    tag="body"
)
```

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `fk_controls` | `list[Control]` | FK controls |
| `fk_end_control` | `Control` | End joint control (if created) |

#### Methods

```python
# Add auto-aim feature
fk.add_auto_aim(follow_control, mirrored_chain=False)
```

### HeadComponent

Specialized FK component for head/neck with orient switch.

```python
from luna_rig.components import HeadComponent

head = HeadComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.CHEST,
    character=character,
    side="c",
    name="head",
    start_joint="neck_01_jnt",
    end_joint="head_end_jnt",
    head_joint_index=-2,     # Index of head joint in chain
    lock_translate=False,
    tag="body"
)
```

#### Properties

| Property | Description |
|----------|-------------|
| `head_control` | Head control |
| `neck_controls` | List of neck controls |

#### Methods

```python
# Add world/local orient switch
head.add_orient_attr()
```

## IK Components

### IKComponent

Pure IK chain component.

```python
from luna_rig.components import IKComponent

ik = IKComponent.create(
    meta_parent=parent_comp,
    hook=0,
    character=character,
    side="l",
    name="arm_ik",
    start_joint="l_arm_01_jnt",
    end_joint="l_arm_03_jnt",
    tag="body"
)
```

### IKSplineComponent

IK spline for flexible chains.

```python
from luna_rig.components import IKSplineComponent

ik_spline = IKSplineComponent.create(
    meta_parent=parent_comp,
    hook=0,
    character=character,
    side="c",
    name="tentacle",
    start_joint="tentacle_01_jnt",
    end_joint="tentacle_10_jnt",
    tag="body"
)
```

## FKIK Components

### FKIKComponent

FK/IK blend component with matching.

```python
from luna_rig.components import FKIKComponent

arm = FKIKComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.CHEST,
    character=character,
    side="l",
    name="arm",
    start_joint="l_arm_01_jnt",
    end_joint="l_arm_03_jnt",
    ik_world_orient=False,   # IK control matches joint orient
    default_state=1,         # 0=FK, 1=IK
    param_locator=None,      # Custom param control position
    tag="body"
)
```

#### Hooks

```python
class Hooks(Enum):
    START_JNT = 0
    END_JNT = 1
```

#### Properties

| Property | Type | Description |
|----------|------|-------------|
| `ik_control` | `Control` | IK control |
| `pv_control` | `Control` | Pole vector control |
| `fk_controls` | `list[Control]` | FK controls |
| `param_control` | `Control` | Parameter control |
| `handle` | `IkHandle` | IK handle |
| `fkik_state` | `float` | Current FK/IK state (0-1) |
| `matching_helper` | `Transform` | Helper for IK matching |

#### Methods

```python
# Get FK/IK state
state = arm.fkik_state

# Set FK/IK state
arm.fkik_state = 0  # FK mode
arm.fkik_state = 1  # IK mode

# Switch with matching
arm.switch_fkik(matching=True)

# Manual matching
arm.match_ik_to_fk()
arm.match_fk_to_ik()

# Bake FKIK
arm.bake_fkik(source="fk", time_range=(1, 100), bake_pv=True)
arm.bake_fkik(source="ik", time_range=(1, 100))

# Hide last FK control
arm.hide_last_fk()

# Add FK orient switch
arm.add_fk_orient_switch()
```

### BipedLegComponent

Specialized FKIK component for biped legs with foot setup.

```python
from luna_rig.components import BipedLegComponent

leg = BipedLegComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.HIPS,
    character=character,
    side="l",
    name="leg",
    start_joint="l_leg_01_jnt",
    end_joint=None,          # Auto-detect
    ik_world_orient=True,
    default_fkik_state=1,
    tag="body"
)
```

#### Methods

```python
# Build reverse foot
leg.build_foot(
    reverse_chain="l_foot_rv_01_jnt",
    foot_locators_grp="l_foot_locators",
    foot_roll_axis="ry"
)

# Create twist joints
upper_twist, lower_twist = leg.create_twist(
    hip_joints_count=2,
    shin_joints_count=2,
    mirrored_chain=False
)

# Get foot component
foot = leg.get_foot()
```

## Body Components

### HandComponent

Hand with automatic finger setup.

```python
from luna_rig.components import HandComponent

hand = HandComponent.create(
    meta_parent=arm,
    character=character,
    side="l",
    name="hand",
    hook=arm.Hooks.END_JNT,
    tag="body"
)
```

#### Properties

| Property | Description |
|----------|-------------|
| `fingers` | List of finger FKComponents |

#### Methods

```python
# Add individual finger
finger = hand.add_fk_finger(
    start_joint="l_index_01_jnt",
    end_joint=None,
    name="index",
    end_control=False
)

# Setup all five fingers at once
fingers = hand.five_finger_setup(
    thumb_start="l_thumb_01_jnt",
    index_start="l_index_01_jnt",
    middle_start="l_middle_01_jnt",
    ring_start="l_ring_01_jnt",
    pinky_start="l_pinky_01_jnt",
    tip_control=False
)
```

### FootComponent

Reverse foot setup (typically created by BipedLegComponent).

```python
from luna_rig.components import FootComponent

foot = FootComponent.create(
    meta_parent=leg,
    hook=0,
    character=character,
    side="l",
    name="foot",
    reverse_chain="l_foot_rv_heel_jnt",
    foot_locators_grp="l_foot_locators",
    ik_control=leg.ik_control,
    tag="body"
)
```

### CollarComponent

Shoulder/collar bone setup.

```python
from luna_rig.components import CollarComponent

collar = CollarComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.CHEST,
    character=character,
    side="l",
    name="collar",
    start_joint="l_clavicle_jnt",
    tag="body"
)
```

### EyeComponent

Eye with aim and FK controls.

```python
from luna_rig.components import EyeComponent

eye = EyeComponent.create(
    meta_parent=head,
    hook=head.Hooks.HEAD,
    character=character,
    side="l",
    name="eye",
    aim_locator="l_eye_aim_loc",
    eye_joint="l_eye_jnt",
    aim_vector=[0, 0, 1],
    up_vector=[0, 1, 0],
    target_wire=True,        # Wire line to aim target
    tag="face"
)
```

#### Properties

| Property | Description |
|----------|-------------|
| `aim_control` | Aim target control |
| `fk_control` | FK eye control |

## Special Components

### TwistComponent

Twist joint distribution.

```python
from luna_rig.components import TwistComponent

twist = TwistComponent.create(
    meta_parent=arm,
    hook=arm.Hooks.START_JNT,
    character=character,
    side="l",
    name="arm_twist",
    start_joint="l_arm_01_jnt",
    end_joint="l_arm_02_jnt",
    num_joints=3,
    tag="body"
)
```

### RibbonComponent

General ribbon setup for flexible deformation.

```python
from luna_rig.components import RibbonComponent

ribbon = RibbonComponent.create(
    meta_parent=parent_comp,
    hook=0,
    character=character,
    side="c",
    name="ribbon",
    start_joint="joint_01",
    end_joint="joint_05",
    tag="body"
)
```

### SimpleComponent

Minimal component for simple setups.

```python
from luna_rig.components import SimpleComponent

simple = SimpleComponent.create(
    meta_parent=parent_comp,
    hook=0,
    character=character,
    side="c",
    name="prop",
    joint="prop_jnt",
    tag="prop"
)
```

### CorrectiveComponent

Corrective blendshape driver.

```python
from luna_rig.components import CorrectiveComponent

corrective = CorrectiveComponent.create(
    meta_parent=parent_comp,
    hook=0,
    character=character,
    side="l",
    name="arm_corrective",
    tag="corrective"
)
```

### WireComponent

Wire deformer component.

```python
from luna_rig.components import WireComponent

wire = WireComponent.create(
    meta_parent=parent_comp,
    hook=0,
    character=character,
    side="c",
    name="wire_setup",
    tag="deformer"
)
```

### FKDynamicsComponent

FK chain with dynamics simulation.

```python
from luna_rig.components import FKDynamicsComponent

dynamics = FKDynamicsComponent.create(
    meta_parent=parent_comp,
    hook=0,
    character=character,
    side="c",
    name="tail_dynamics",
    start_joint="tail_01_jnt",
    end_joint="tail_05_jnt",
    tag="body"
)
```

### Stretch Components

#### IKStretchComponent

Stretchy IK setup.

```python
from luna_rig.components import IKStretchComponent

stretch = IKStretchComponent.create(
    meta_parent=arm,
    hook=0,
    character=character,
    side="l",
    name="arm_stretch",
    tag="body"
)
```

#### IKSplineStretchComponent

Stretchy IK spline setup.

```python
from luna_rig.components import IKSplineStretchComponent

spline_stretch = IKSplineStretchComponent.create(
    meta_parent=spine,
    hook=0,
    character=character,
    side="c",
    name="spine_stretch",
    tag="body"
)
```

## Creating Custom Components

To create a custom component, inherit from `AnimComponent`:

```python
import pymel.core as pm
import luna_rig
from luna_rig.functions import jointFn, attrFn, nameFn
from luna.utils import enumFn

class MyCustomComponent(luna_rig.AnimComponent):
    """Custom component documentation."""

    # Required plugins (optional)
    required_plugins = ["myPlugin"]

    # Define hooks
    class Hooks(enumFn.Enum):
        BASE = 0
        TIP = 1

    # Custom properties
    @property
    def my_control(self):
        return luna_rig.Control(self.pynode.myControl.get())

    @classmethod
    def create(cls,
               meta_parent=None,
               hook=0,
               character=None,
               side="c",
               name="custom",
               start_joint=None,
               end_joint=None,
               tag=""):

        # Create base instance
        instance = super(MyCustomComponent, cls).create(
            meta_parent=meta_parent,
            side=side,
            name=name,
            character=character,
            tag=tag
        )

        # Add custom attributes to meta node
        instance.pynode.addAttr("myControl", at="message")

        # Get joint chain
        joint_chain = jointFn.joint_chain(start_joint, end_joint)
        for jnt in joint_chain:
            attrFn.add_meta_attr(jnt)

        # Create control chain
        ctl_chain = jointFn.duplicate_chain(
            new_joint_name=[instance.indexed_name, "ctl"],
            new_joint_side=instance.side,
            original_chain=joint_chain,
            new_parent=instance.group_joints
        )

        # Create controls
        my_ctl = luna_rig.Control.create(
            name=instance.indexed_name,
            side=instance.side,
            guide=joint_chain[0],
            parent=instance.group_ctls,
            shape="cube"
        )

        # Store data
        instance._store_bind_joints(joint_chain)
        instance._store_ctl_chain(ctl_chain)
        instance._store_controls([my_ctl])

        # Connect custom attrs
        my_ctl.transform.metaParent.connect(instance.pynode.myControl)

        # Add hooks
        instance.add_hook(ctl_chain[0], "base")
        instance.add_hook(ctl_chain[-1], "tip")

        # Connect to character and parent
        instance.connect_to_character(parent=True)
        instance.attach_to_component(meta_parent, hook)

        return instance

    def attach_to_component(self, other_comp, hook_index=None):
        super(MyCustomComponent, self).attach_to_component(
            other_comp, hook_index
        )
        if self.in_hook:
            pm.parentConstraint(
                self.in_hook.transform,
                self.controls[0].group,
                mo=1
            )
```

### Component Creation Checklist

1. Inherit from appropriate base class
2. Define `Hooks` enum if needed
3. Add custom message attributes to meta node
4. Process joint chain
5. Create control chain
6. Create controls
7. Store bind joints, ctl chain, controls
8. Connect custom attributes
9. Add output hooks
10. Connect to character
11. Attach to parent component
12. Scale controls appropriately
