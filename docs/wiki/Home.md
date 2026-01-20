# Luna Rigging Framework

Welcome to the Luna Rigging Framework documentation. Luna is a professional, modular rigging framework for Autodesk Maya, designed to streamline character rigging workflows in VFX and animation production environments.

## Overview

Luna provides a comprehensive toolkit for character rigging with support for:

- **Modular Component-Based Architecture**: Build complex rigs from reusable, self-contained components
- **Dual Workflow Support**: Python-based procedural rigging for technical artists and a visual node-based editor (Luna Builder) for artists
- **Meta-Node System**: Intelligent rig metadata storage using Maya network nodes
- **Project/Asset Management**: Organized workspace structure for production pipelines
- **Automatic Control Shapes**: Built-in shape library with copy/paste functionality
- **FKIK Switching**: Seamless blending with matching capabilities
- **Space Switching**: Matrix-based or constraint-based space switching
- **Import/Export System**: Skin weights, blendshapes, control shapes, and more

## Quick Links

| Section | Description |
|---------|-------------|
| [Getting Started](Getting-Started.md) | Installation, setup, and quick start guide |
| **[Quick Start: Builder](Quick-Start-Builder.md)** | **Step-by-step tutorial for Luna Builder** |
| [Architecture](Architecture.md) | High-level architecture and design patterns |
| [Core Concepts](Core-Concepts.md) | Key concepts: MetaNode, Component, Control, Hook |
| [Components](Components.md) | All rig components reference |
| [Functions](Functions.md) | Utility functions reference |
| [Builder](Builder.md) | Luna Builder visual editor documentation |
| [Tools](Tools.md) | Animation tools, skin tools, etc. |
| [Naming Convention](Naming-Convention.md) | Luna naming system and templates |
| [API Reference](API-Reference.md) | Detailed API reference for developers |

## Version Information

- **Current Version**: 0.2.14
- **Maya Compatibility**: Maya 2019+, 2020+, 2022+, 2023+
- **License**: MIT
- **Repository**: https://github.com/Aiacos/luna.git

## Package Structure

```
luna/
├── luna/              # Core framework (logging, config, UI, workspace)
├── luna_rig/          # Rigging core (components, controls, functions)
├── luna_builder/      # Visual node-based editor
├── luna_configer/     # Configuration UI
├── configs/           # Default configuration files
├── plug-ins/          # Maya binary plugins by version
└── res/               # Resources (icons, shapes)
```

## Key Features

### Component-Based Rigging

Luna uses a modular component system where each body part is a self-contained component that can be created, modified, and connected to other components. This approach enables:

- Reusable rig modules across different characters
- Easy maintenance and updates
- Consistent rig structure across projects

### Meta-Node System

Based on the GDC 2015 talk by David Hunt and Forrest Soderlind, Luna uses Maya network nodes to store rig metadata. This enables:

- Querying and manipulating rigs programmatically
- Maintaining parent-child relationships between components
- Storing custom data on rig elements

### Visual Builder

Luna Builder provides a node-based visual interface for building rigs without writing code, perfect for artists who prefer a visual workflow.

### Production Pipeline Integration

Luna includes project and asset management tools designed for studio pipelines, with support for:

- Organized directory structures
- Version control for weights and builds
- Selection sets and publish modes

## Sample Usage

```python
import luna_rig
from luna_rig.components import Character, FKIKSpineComponent, FKIKComponent

# Create character
char = Character.create(name="hero")

# Create spine
spine = FKIKSpineComponent.create(
    character=char,
    start_joint="spine_01_jnt",
    end_joint="spine_04_jnt",
    tag="body"
)

# Create arm
arm = FKIKComponent.create(
    meta_parent=spine,
    hook=spine.Hooks.CHEST,
    character=char,
    side="l",
    name="arm",
    start_joint="l_arm_01_jnt",
    end_joint="l_arm_03_jnt"
)

# Attach and save bind poses
char.attach_to_skeleton()
char.save_bind_pose()
```

## Support and Resources

- **GitHub Issues**: Report bugs and request features
- **Sample Files**: https://github.com/S0nic014/luna_sample_files
- **Demo Video**: https://www.youtube.com/watch?v=0FLPQ91r0To

## Credits

Luna was created with inspiration from:
- GDC 2015 Meta-Node Talk by David Hunt and Forrest Soderlind
- Various open-source rigging frameworks in the Maya community
