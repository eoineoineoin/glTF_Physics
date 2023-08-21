# KHR_collision_shapes

## Status

Draft

## Dependencies

Written against glTF 2.0 spec.

## Overview

This extension adds the ability to specify collision primitives inside a glTF asset. This extension does not mandate any particular behaviour for those objects aside from their collision geometry. These types may be used in combination with additional extensions or application-specific business logic. The KHR\_rigid\_bodies extension uses primitives defined in this spec to provide geometric primitives for collision detection in a rigid body simulation.

## glTF Schema Updates

### Shapes

This extension provides a set of document-level objects, which can be referenced by objects in the scene. The precise usage of these primitives should be specified by the extensions which utilize the shapes. The intended purpose of these these objects are to specify geometry which can be used for collision detection, which has informed the set of shapes which we have defined.. Typically, the geometry specified by the shape will be simpler than any render meshes used by the node or it's children, enabling real-time applications to perform queries such as intersection tests.

Each shape defines a mandatory `type` property that designates the type of shape, as well as an additional structure which provides parameterizations specific to that type.

To describe the geometry which represents the object, shapes must define at most one of the following properties:

| |Type|Description|
|-|-|-|
|**sphere**|`object`|A sphere centered at the origin in local space.|
|**box**|`object`|An axis-aligned box centered at the origin in local space.|
|**capsule**|`object`|A capsule centered at the origin and defined by two "capping" spheres with potentially different radii, aligned along the Y axis in local space.|
|**cylinder**|`object`|A cylinder centered at the origin and aligned along the Y axis in local space, with potentially different radii at each end.|
|**convex**|`object`|A convex hull wrapping a `mesh` object.|
|**trimesh**|`object`|A triangulated representation of a `mesh` object.|

The sphere, box, capsule, cylinder and convex types all represent convex objects with a volume, however, the trimesh type always represents an infinitely thin shell or sheet - for example a trimesh created from a `mesh` object in the shape of a box will be represented as a hollow box.

If you want your shape to have an offset from the local space of the node the shape is associated with (for example a sphere _not_ centered at local origin, or a rotated box), you should add an extra node to the hierarchy and apply your transform and your shape properties to that.

### JSON Schema

* **JSON schema**: [glTF.KHR_collision_shapes.schema.json](schema/glTF.KHR_collision_shapes.schema.json)

## Known Implementations

[Blender importer/exporter](https://github.com/eoineoineoin/glTF_Physics_Blender_Exporter)

[Babylon.js importer](https://github.com/eoineoineoin/glTF_Physics_Babylon)

[Godot importer](https://github.com/eoineoineoin/glTF_Physics_Godot_Importer)

## Validator

[glTF validator](https://github.com/eoineoineoin/glTF-Validator)
