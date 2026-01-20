# Luna Builder

Luna Builder is a visual node-based editor for building rigs without writing Python code. This document covers the Builder interface, workflow, and all available nodes.

## Table of Contents

1. [Overview](#overview)
2. [Launching the Builder](#launching-the-builder)
3. [Interface](#interface)
4. [Basic Workflow](#basic-workflow)
5. [Node Types](#node-types)
6. [Executing Builds](#executing-builds)
7. [Saving and Loading](#saving-and-loading)
8. [Advanced Features](#advanced-features)

## Overview

Luna Builder provides:

- Visual node-based interface for rig creation
- Drag-and-drop node placement
- Real-time node connections
- Graph execution with logging
- Undo/redo support
- Scene variables
- Multiple document interface (tabs)

## Launching the Builder

### From Luna Menu

**Luna > Builder**

### From Python

```python
from luna_builder.main_window import BuilderMainWindow

BuilderMainWindow.display()
```

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Ctrl+N | New build |
| Ctrl+O | Open build |
| Ctrl+S | Save build |
| Ctrl+Shift+S | Save build as |
| Ctrl+E | Execute graph |
| Ctrl+Z | Undo |
| Ctrl+Y | Redo |
| Delete | Delete selected nodes |
| Ctrl+C | Copy nodes |
| Ctrl+V | Paste nodes |

## Interface

```
+------------------------------------------------------------------+
| Menu Bar                                                    [R]  |
+----------+--------------------------------------+----------------+
| Nodes    |                                      | Workspace      |
| Palette  |                                      | -----------    |
|          |        Node Canvas                   | Attributes     |
| -------- |        (MDI Tabs)                    | -----------    |
| Variables|                                      | History        |
+----------+--------------------------------------+----------------+
```

### Menu Bar

| Menu | Contents |
|------|----------|
| **File** | New, Open, Save, Save As, Recent files |
| **Edit** | Undo, Redo, Cut, Copy, Paste, Delete |
| **Graph** | Execute, Step Execute, Verify |
| **Window** | Toggle dock widgets |
| **Controls** | Control shape operations |
| **Joints** | Joint utilities |
| **Skin** | Skin weight operations |
| **Deformers** | Deformer utilities |
| **Rig** | Rig operations |
| **Help** | Documentation, About |

### Nodes Palette

Left dock widget containing all available nodes organized by category:

- **Graph**: Input, Output, For Each, Branch, Sequence
- **Components**: Character, FK, IK, FKIK, Spine, etc.
- **Utility**: Control, Logger, Script, GetSet, List
- **Maya**: PyMEL operations

Drag nodes from the palette to the canvas to add them.

### Node Canvas

Central area where you build your graph:

- **Zoom**: Mouse wheel
- **Pan**: Middle mouse button drag
- **Select**: Left click or box select
- **Multi-select**: Shift+click
- **Connect**: Drag from output socket to input socket
- **Disconnect**: Right-click on edge

### Workspace Widget

Shows project/asset information when a project is set.

### Attributes Editor

Displays properties of the selected node for editing:

- Node name/title
- Input values
- Settings
- Help text

### Variables Widget

Manage scene variables:

- Add new variables
- Edit variable names and values
- Delete variables
- Variables persist with the build file

### History Widget

Shows scene history for undo/redo:

- Click on history entries to jump to that state
- Visual indication of current state

## Basic Workflow

### Step 1: Create New Build

1. **File > New** or press **Ctrl+N**
2. A new empty canvas appears

### Step 2: Add Input Node

1. From Nodes Palette, drag **Input** node to canvas
2. This is required as the starting point of execution

### Step 3: Add Character Node

1. Drag **Character** node to canvas
2. Connect Input's output to Character's input
3. Select Character node and set properties in Attributes panel:
   - Name
   - Tag

### Step 4: Add Component Nodes

1. Drag component nodes (Spine, FK, FKIK, etc.)
2. Connect them to Character or parent components
3. Configure each node's properties:
   - Start joint
   - End joint
   - Side
   - Name
   - Hook index

### Step 5: Configure Node Properties

For each node:
1. Select the node
2. Use the Attributes panel to set:
   - Joint assignments
   - Side (l/r/c)
   - Name
   - Hook connections
   - Component-specific settings

### Step 6: Execute Graph

1. **Graph > Execute** or press **Ctrl+E**
2. Watch the Maya scene for rig creation
3. Check Script Editor for logs

### Step 7: Save Build

1. **File > Save As** or press **Ctrl+Shift+S**
2. Choose location and filename (.rig extension)
3. Build files can be reopened and modified

## Node Types

### Graph Nodes

#### Input Node

Starting point for graph execution.

- **Output**: exec (execution flow)

#### Output Node

End point for graph execution.

- **Input**: exec (execution flow)

#### For Each Node

Iterates over a list.

- **Inputs**:
  - exec (execution flow)
  - list (items to iterate)
- **Outputs**:
  - loop_body (execute for each item)
  - item (current item)
  - index (current index)
  - completed (after iteration)

#### Branch Node

Conditional execution.

- **Inputs**:
  - exec (execution flow)
  - condition (boolean)
- **Outputs**:
  - true (if condition is true)
  - false (if condition is false)

#### Sequence Node

Sequential execution of multiple outputs.

- **Inputs**: exec
- **Outputs**: Multiple execution outputs

### Component Nodes

#### Character Node

Creates the root character component.

- **Inputs**:
  - exec
- **Outputs**:
  - exec
  - character (Character instance)
- **Properties**:
  - name
  - tag

#### FK Component Node

Creates an FK chain.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - start_joint
  - end_joint
  - add_end_ctl
  - lock_translate
  - tag

#### IK Component Node

Creates an IK chain.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - start_joint
  - end_joint
  - tag

#### FKIK Component Node

Creates an FK/IK blend chain.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - start_joint
  - end_joint
  - ik_world_orient
  - default_state
  - tag

#### Spine (FKIK) Node

Creates a spine with FK/IK.

- **Inputs**:
  - exec
  - character
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - start_joint
  - end_joint
  - up_axis
  - forward_axis
  - tag

#### Biped Leg Node

Creates a biped leg with foot.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - start_joint
  - end_joint
  - ik_world_orient
  - default_fkik_state
  - tag

#### Hand Component Node

Creates a hand with fingers.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - finger joints
  - tag

#### Eye Component Node

Creates an eye aim setup.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - aim_locator
  - eye_joint
  - aim_vector
  - up_vector
  - target_wire
  - tag

#### Ribbon Node

Creates a ribbon setup.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - start_joint
  - end_joint
  - tag

#### Corrective Node

Creates a corrective driver.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - tag

#### Simple Component Node

Creates a minimal component.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - joint
  - tag

#### Twist Node

Creates twist joint distribution.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - start_joint
  - end_joint
  - num_joints
  - tag

#### Stretch Node

Creates stretchy setup.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - tag

#### Wire Node

Creates wire deformer setup.

- **Inputs**:
  - exec
  - character
  - meta_parent
  - hook
- **Outputs**:
  - exec
  - component
- **Properties**:
  - side
  - name
  - tag

### Utility Nodes

#### Control Node

Creates a standalone control.

- **Inputs**:
  - exec
  - guide
  - parent
- **Outputs**:
  - exec
  - control
- **Properties**:
  - name
  - side
  - shape
  - color
  - attributes
  - offset_grp
  - joint

#### Logger Node

Logs messages during execution.

- **Inputs**:
  - exec
  - message
- **Outputs**:
  - exec
- **Properties**:
  - level (info/warning/error/debug)

#### Script Node

Executes custom Python code.

- **Inputs**:
  - exec
  - (custom inputs)
- **Outputs**:
  - exec
  - (custom outputs)
- **Properties**:
  - code

#### GetSet Node

Gets or sets scene variables.

- **Inputs**:
  - exec
  - variable_name
  - value (for set)
- **Outputs**:
  - exec
  - value (for get)
- **Properties**:
  - mode (get/set)

#### Constant Node

Provides constant values.

- **Outputs**:
  - value
- **Properties**:
  - value_type
  - value

#### List Node

Creates or manipulates lists.

- **Inputs**:
  - items
- **Outputs**:
  - list
- **Properties**:
  - operation

#### Function Node

Calls Python functions.

- **Inputs**:
  - exec
  - function
  - arguments
- **Outputs**:
  - exec
  - result

### Maya Nodes

#### PyMEL Node

Executes PyMEL operations.

- **Inputs**:
  - exec
  - node
  - operation
- **Outputs**:
  - exec
  - result

## Executing Builds

### Full Execution

Execute the entire graph from Input to Output:

1. **Graph > Execute** or **Ctrl+E**
2. Execution starts from Input node
3. Follows connections through the graph
4. Creates rig in Maya scene

### Step Execution

Execute one node at a time:

1. **Graph > Step Execute**
2. Executes next node in queue
3. Useful for debugging

### Verify Graph

Check graph validity without executing:

1. **Graph > Verify**
2. Reports errors if any nodes are invalid
3. Checks for missing connections

### Execution Flow

The `GraphExecutor` handles build execution:

1. **Verify**: Check all nodes are valid
2. **Build Queue**: Traverse from Input node following connections
3. **Execute**: Run each node's `_exec()` method in order
4. **Report**: Log execution time and any errors

## Saving and Loading

### File Format

Build files use the `.rig` extension and store:

- Node types and positions
- Node connections
- Node attribute values
- Scene variables
- Metadata

### Save Build

```
File > Save As (Ctrl+Shift+S)
```

### Open Build

```
File > Open (Ctrl+O)
```

### Recent Files

```
File > Recent Files
```

### Asset Integration

When a project/asset is set, builds can be saved to:

```
{asset_path}/build/{asset_name}.{version}.rig
```

## Advanced Features

### Scene Variables

Variables allow sharing data between nodes:

1. **Add Variable**: Click + in Variables panel
2. **Set Name**: Double-click to rename
3. **Set Value**: Use GetSet node with "set" mode
4. **Get Value**: Use GetSet node with "get" mode

Variables persist with the build file.

### Script Node

For custom Python code:

```python
# Access inputs
start_joint = self.get_input("start_joint")
end_joint = self.get_input("end_joint")

# Your custom code
import pymel.core as pm
chain = pm.ls(start_joint, end_joint, type="joint")

# Set outputs
self.set_output("result", chain)
```

### For Each Loops

Process lists of items:

1. Add **For Each** node
2. Connect list to iterate over
3. Connect loop_body to nodes that process each item
4. Use item output to access current item
5. Connect completed to continue after loop

### Conditional Execution

Use **Branch** node for if/else logic:

1. Add **Branch** node
2. Connect condition (boolean value)
3. Connect true output to nodes for true case
4. Connect false output to nodes for false case

### Programmatic Execution

Execute builds from Python:

```python
from luna_builder.editor.graph_executor import GraphExecutor
from luna_builder.editor.node_scene import NodeScene

# Load scene from file
scene = NodeScene()
scene.load_from_file("/path/to/build.rig")

# Create executor
executor = GraphExecutor(scene)

# Execute
executor.execute_graph()

# Or step-by-step
while executor.has_next():
    executor.execute_step()
```

### Custom Node Creation

To create custom nodes for Luna Builder:

```python
from luna_builder.rig_nodes.luna_node import LunaNode
from luna_builder.rig_nodes.base_component import BaseComponentNode

class MyCustomNode(BaseComponentNode):
    """Node for MyCustomComponent."""

    ID = 200  # Unique ID
    IS_EXEC = True
    TITLE_EDITABLE = True
    COMPONENT_CLASS = MyCustomComponent

    def init_sockets(self, reset=True):
        super(MyCustomNode, self).init_sockets(reset)
        # Add custom sockets
        self.add_input("start_joint", "Start Joint", socket_type="joint")
        self.add_input("end_joint", "End Joint", socket_type="joint")

    def _exec(self):
        # Get input values
        start_joint = self.get_input_value("start_joint")
        end_joint = self.get_input_value("end_joint")

        # Create component
        self.component = self.COMPONENT_CLASS.create(
            meta_parent=self.get_meta_parent(),
            hook=self.get_hook_index(),
            character=self.get_character(),
            side=self.get_side(),
            name=self.get_name(),
            start_joint=start_joint,
            end_joint=end_joint,
            tag=self.get_tag()
        )

        return self.component
```

### Best Practices

1. **Start with Input**: Always begin your graph with an Input node
2. **Character First**: Create Character node before components
3. **Connect Hooks**: Use correct hook indices for parent-child relationships
4. **Save Often**: Save builds frequently during development
5. **Use Variables**: Store shared data in variables rather than duplicating
6. **Test Incrementally**: Execute and verify parts of the graph before completing
7. **Log Messages**: Use Logger nodes to debug complex graphs
