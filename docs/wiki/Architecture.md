# Luna Architecture

This document provides a comprehensive overview of Luna's architecture, design patterns, and how the various packages interact.

## Table of Contents

1. [Package Overview](#package-overview)
2. [Core Framework (luna)](#core-framework-luna)
3. [Rigging Core (luna_rig)](#rigging-core-luna_rig)
4. [Visual Builder (luna_builder)](#visual-builder-luna_builder)
5. [Design Patterns](#design-patterns)
6. [Data Flow](#data-flow)

## Package Overview

Luna is organized into four main packages:

```
luna/
├── luna/              # Core framework
│   ├── core/          # Config, logger, callbacks
│   ├── interface/     # Menu, marking menu, HUD
│   ├── static/        # Constants, directories, colors
│   ├── tools/         # Animation tools
│   ├── utils/         # Utility functions
│   └── workspace/     # Project and asset management
│
├── luna_rig/          # Rigging core
│   ├── core/          # MetaNode, Component, Control
│   ├── components/    # Pre-built rig components
│   ├── functions/     # Utility modules
│   └── importexport/  # Weight and data I/O
│
├── luna_builder/      # Visual node editor
│   ├── editor/        # Node editor core
│   ├── rig_nodes/     # Visual node classes
│   └── menus/         # Builder menus
│
└── luna_configer/     # Configuration UI
```

## Core Framework (luna)

The `luna` package provides the foundation for the entire framework.

### core/config.py

Manages Luna's configuration using a JSON-based key-value store.

```python
from luna import Config

# Get/set configuration values
value = Config.get("my_setting", default="default")
Config.set("my_setting", "new_value")

# Cached access for frequently used values
value = Config.get("my_setting", cached=True)
```

**Key Features**:
- Persistent JSON storage
- Cached access for performance
- Default value support
- Update multiple values at once

### core/logger.py

Custom logging system with Qt signal support for UI integration.

```python
from luna import Logger

Logger.info("Information message")
Logger.warning("Warning message")
Logger.error("Error message")
Logger.debug("Debug message")
```

### core/callbacks.py

Manages Maya event callbacks for scene events.

### interface/

User interface components:

| Module | Purpose |
|--------|---------|
| `menu.py` | Luna main menu in Maya |
| `marking_menu.py` | Context-sensitive right-click menu |
| `hud.py` | Heads-up display for project/asset info |
| `shared_widgets.py` | Reusable Qt widgets |

### static/

Constants and static values:

| Module | Purpose |
|--------|---------|
| `vars.py` | Configuration variable names |
| `directories.py` | Path definitions |
| `colors.py` | Color constants |
| `names.py` | Naming constants |

### workspace/

Project and asset management:

| Module | Purpose |
|--------|---------|
| `project.py` | Project class for workspace management |
| `asset.py` | Asset class for character/prop management |

### tools/

Animation and utility tools:

| Module | Purpose |
|--------|---------|
| `anim_baker.py` | Skeleton/FKIK/Space baking |
| `custom_space_tool.py` | Custom space switching |
| `transfer_keyframes.py` | Keyframe transfer utilities |
| `driven_pose_exporter.py` | Driven pose export/import |

### utils/

Utility functions:

| Module | Purpose |
|--------|---------|
| `fileFn.py` | File operations |
| `enumFn.py` | Enum utilities |
| `pysideFn.py` | Qt/PySide helpers |
| `maya_utils.py` | Maya-specific utilities |

## Rigging Core (luna_rig)

The `luna_rig` package contains all rigging functionality.

### core/meta.py - MetaNode

The foundation of Luna's rig system. Every rig element is represented by a Maya network node with a `metaType` attribute.

```
MetaNode
│
└── Network node with:
    ├── metaType (string)     # Full class path
    ├── metaParent (message)  # Parent connection
    ├── metaChildren (message[]) # Children connections
    └── tag (string)          # Custom tag
```

**Class Hierarchy**:
```
MetaNode
└── Component
    └── AnimComponent
        ├── FKComponent
        ├── IKComponent
        ├── FKIKComponent
        └── ... (all body components)
    └── Character
```

### core/component.py - Component & AnimComponent

Base classes for rig components.

**Component** extends MetaNode with:
- Settings storage
- Utility node management
- Required plugins handling

**AnimComponent** extends Component with:
- Standard rig hierarchy creation
- Control management
- Joint chain management
- Hook system for connections
- Skeleton attachment/detachment

**Standard AnimComponent Hierarchy**:
```
{name}_comp/           # Component root
├── {name}_ctls/       # Controls group
├── {name}_jnts/       # Joint group
├── {name}_parts/      # Parts group
│   └── {name}_noscale/  # No-scale group
└── {name}_out/        # Output/hooks group
```

### core/control.py - Control

Represents animator-facing rig controls.

**Control Hierarchy**:
```
{name}_grp                    # Group
└── {name}_ofs (optional)     # Offset
    └── {name}_ctl            # Transform
        └── {name}_cjnt       # Joint (optional)
```

**Key Features**:
- Automatic hierarchy creation
- Shape management via ShapeManager
- Space switching support
- Bind pose storage
- Mirror support

### core/shape_manager.py - ShapeManager

Manages control shapes from the shape library.

**Capabilities**:
- Load shapes from library
- Save custom shapes
- Copy/paste shapes between controls
- Color management
- Line width control

### components/

Pre-built rig components:

| Component | Purpose |
|-----------|---------|
| `Character` | Root rig component |
| `FKComponent` | FK chain |
| `IKComponent` | IK chain |
| `FKIKComponent` | FK/IK blend |
| `FKIKSpineComponent` | Spine with FK/IK |
| `BipedLegComponent` | Leg with foot |
| `HandComponent` | Hand with fingers |
| `EyeComponent` | Eye aim setup |
| `TwistComponent` | Twist distribution |
| And more... | |

### functions/

Utility function modules:

| Module | Purpose |
|--------|---------|
| `nameFn.py` | Naming operations |
| `jointFn.py` | Joint operations |
| `attrFn.py` | Attribute operations |
| `nodeFn.py` | Node creation |
| `curveFn.py` | Curve operations |
| `transformFn.py` | Transform operations |
| `animFn.py` | Animation operations |
| `rigFn.py` | Rig utility functions |
| `deformerFn.py` | Deformer operations |
| And more... | |

### importexport/

Data import/export:

| Module | Purpose |
|--------|---------|
| `skin.py` | Skin weights I/O |
| `blendshape.py` | Blendshape I/O |
| `control_shapes.py` | Control shape I/O |
| `driven_pose.py` | Driven pose I/O |
| `deltamush.py` | Delta mush I/O |

## Visual Builder (luna_builder)

The `luna_builder` package provides the visual node editor.

### main_window.py - BuilderMainWindow

Main application window with:
- MDI (Multiple Document Interface) for tabs
- Dock widgets for panels
- Menu bar

**Layout**:
```
┌──────────────────────────────────────────────────────────┐
│ Menu Bar                                            [⟳] │
├──────────┬────────────────────────────────┬─────────────┤
│ Nodes    │                                │ Workspace   │
│ Palette  │        Node Canvas             │ ----------- │
│          │        (MDI Area)              │ Attributes  │
│ -------- │                                │ ----------- │
│ Variables│                                │ History     │
└──────────┴────────────────────────────────┴─────────────┘
```

### editor/

Node editor core:

| Module | Purpose |
|--------|---------|
| `node_editor.py` | Editor widget |
| `node_scene.py` | Scene management |
| `node_node.py` | Base node class |
| `node_edge.py` | Edge connections |
| `node_socket.py` | Input/output sockets |
| `graph_executor.py` | Build execution |
| `graphics_*.py` | Qt graphics items |
| `node_scene_history.py` | Undo/redo |
| `node_scene_vars.py` | Scene variables |

### rig_nodes/

Visual node representations:

| Module | Purpose |
|--------|---------|
| `luna_node.py` | Base Luna node |
| `base_component.py` | Base component node |
| `node_character.py` | Character node |
| `node_fk_component.py` | FK component node |
| `node_fkik_component.py` | FKIK component node |
| `node_biped_leg.py` | Biped leg node |
| `node_control.py` | Control creation node |
| `node_for_each.py` | Loop node |
| `node_branch.py` | Conditional node |
| And more... | |

### menus/

Builder menus:

| Module | Purpose |
|--------|---------|
| `file_menu.py` | File operations |
| `edit_menu.py` | Edit operations |
| `graph_menu.py` | Graph operations |
| `controls_menu.py` | Control utilities |
| `joints_menu.py` | Joint utilities |
| `skin_menu.py` | Skin utilities |
| `deformers_menu.py` | Deformer utilities |
| `rig_menu.py` | Rig operations |

## Design Patterns

### Meta-Node Pattern

Based on GDC 2015 talk by David Hunt and Forrest Soderlind.

**Principle**: Use Maya network nodes to store rig metadata with a `metaType` attribute containing the full Python class path.

**Benefits**:
- Query rigs programmatically
- Reconstruct Python objects from scene data
- Maintain relationships between components

```python
# Example: Creating and retrieving a MetaNode
meta = MetaNode.create(meta_parent=None)
print(meta.pynode.metaType.get())
# Output: "luna_rig.core.meta.MetaNode"

# Later, reconstruct from node
same_meta = MetaNode(meta.pynode)
# Returns correct class instance
```

### Component Factory Pattern

Components use a class method factory pattern for creation:

```python
class MyComponent(AnimComponent):
    @classmethod
    def create(cls, meta_parent=None, side="c", name="component", **kwargs):
        instance = super(MyComponent, cls).create(meta_parent, side, name)
        # Component-specific setup
        return instance
```

### Hook System Pattern

Components define output hooks for child connections:

```python
class SpineComponent(AnimComponent):
    class Hooks(enumFn.Enum):
        ROOT = 0
        HIPS = 1
        CHEST = 2

# Usage
spine = SpineComponent.create(...)
arm = FKIKComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.CHEST
)
```

### Visitor Pattern (Builder Execution)

The GraphExecutor visits each node in the build graph:

```python
class GraphExecutor:
    def execute_graph(self):
        for node in self.build_queue:
            node._exec()
```

## Data Flow

### Rig Creation Flow

```
1. Character.create()
   └── Creates root hierarchy and control

2. Component.create()
   ├── Creates component hierarchy
   ├── Duplicates joint chain
   ├── Creates controls
   ├── Sets up constraints
   └── Connects to character

3. attach_to_skeleton()
   └── Creates constraints between ctl_chain and bind_joints

4. save_bind_pose()
   └── Stores control default values
```

### Builder Execution Flow

```
1. User creates/connects nodes

2. Execute Graph
   ├── Verify all nodes valid
   ├── Build execution queue from Input node
   └── For each node:
       ├── Get input values from connected nodes
       ├── Execute node._exec()
       └── Store output for downstream nodes

3. Post-execution
   └── Character.attach_to_skeleton()
```

### Import/Export Flow

```
Export:
1. Manager.export_single(geometry)
   ├── Find deformer
   ├── Collect weights
   └── Save to versioned file

Import:
1. Manager.import_single(geometry)
   ├── Find latest file
   ├── Create deformer if needed
   └── Apply weights
```

## Module Dependencies

```
luna (core)
    │
    ├── luna_rig (depends on luna)
    │   ├── core
    │   ├── components
    │   ├── functions
    │   └── importexport
    │
    └── luna_builder (depends on luna, luna_rig)
        ├── editor
        ├── rig_nodes
        └── menus
```

This architecture ensures:
- Clean separation of concerns
- Reusable components
- Extensibility for custom components
- Both programmatic and visual workflows
