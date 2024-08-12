# KHR_geometry

## Contributors

* Eoin Mcloughlin, Microsoft, <mailto:eomcl@microsoft.com>
* Rory Mullane, Microsoft, <mailto:romul@microsoft.com>
* George Tian, Microsoft, <mailto:geotian@microsoft.com>
* Aaron Franke, Godot Engine, <mailto:arnfranke@yahoo.com>
* Eric Griffith, Meta, <mailto:ericgriffith@meta.com>

## Status

Draft

## Dependencies

Written against glTF 2.0 spec.

## Overview

This extension adds the ability to specify additional geometry types inside a glTF asset. This extension does not mandate any particular behaviour for those objects aside from providing data definitions. These types are to be used in combination with other extensions that reference these objects.

## glTF Schema Updates

### Geometries

This extension provides a set of document-level objects, which can be referenced by objects in the scene. The precise usage of these primitives should be specified by the extensions which utilize the geometries.

Geometries are defined within a dictionary property in the glTF scene description file, by adding an `extensions` property to the top-level glTF 2.0 object and defining a `KHR_geometry` property with a `geometries` array inside it.

Each geometry defines a mandatory `type` property which designates the type of geometry, as well as an additional structure which provides parameterizations specific to that type. The following example defines a sphere.

```javascript
"extensions": {
    "KHR_geometry" : {
        "geometries": [
            {
                "sphere": { "radius": 0.1 },
                "type": "sphere"
            }
        ]
    }
}
```

An extension wishing to leverage these geometries may add fields to an extension object which index into this array of geometries. In the typical scenario where this extension object is associated with an object, geometries are parameterized in local space of that object. If a geometry is required to have an offset from the local space of the node the geometry is associated with (for example a sphere _not_ centered at local origin or a rotated box,) a child node should be added with the desired offset applied, and the geometry properties added to that child. TODO.eoin Reword all of this!

Geometries must define at most one of the following properties:

| |Type|Description|
|-|-|-|
|**sphere**|`object`|A sphere centered at the origin in local space.|
|**box**|`object`|An axis-aligned box centered at the origin in local space.|
|**cylinder**|`object`|A cylinder centered at the origin and aligned along the Y axis in local space, with potentially different radii at each end.|
|**capsule**|`object`|A capsule (cylinder with hemispherical ends) centered at the origin and defined by two "capping" spheres with potentially different radii, aligned along the Y axis in local space.|
|**mesh**|`object`|A geometry generated from a `mesh` object.|

The sphere, box, capsule, and cylinder all represent convex objects with a volume, while the mesh represents the surface of a referenced mesh object.

As the mesh geometry references a `mesh`, it additionally allows for optional `skin` and `weights` properties, which have the same semantics and requirements enforced by the properties of the same name associated with a `node`. Within the core glTF specification, there are two methods to specify morph target weights - the mesh object itself may have a `weights` parameter, while a node may specify per-instance weights, which take priority over the weights specified by the mesh. Analogously, a geometry may reference a mesh which has a set of weights, while the geometry itself may specify weights which take priority over those specified in the mesh; finally, when a mesh geometry specifies `useNodeWeights` as `true`, the weights used by that node referencing that geometry take priority. This construct allows for a node's instance weights to animate a mesh without having to additionally propagate the weights to the geometry. When `useNodeWeights` is `true`, the mesh referenced by the geometry must have the same number of morph targets as the mesh referenced by the node.

Degenerate geometries are prohibited. A sphere must have a positive, non-zero radius. A box must have positive non-zero values for each component of `size`. The cylinder and capsule must have a positive, non-zero `height` and both `radiusTop` and `radiusBottom` should be positive; at least one of the radii should be non-zero. For meshes, the referenced mesh should contain at least one non-degenerate triangle primitive.

A nodes scale presents some special cases that must be handled. In accordance with the glTF specification on [https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#transformations](transformations), a geometry referenced by a node whose scale is zero on all three axes should be considered disabled. When an analytical geometry is referenced by a node whose whose scale is negative on one or more axes, the resulting geometry size should be the absolute value of the nodes scale applied to the input geometry parameters. i.e. a box with `size` of `[1.0, 1.0, 1.0]` associated with a node whose scale is `[1.0, -2.0, 3.0]` should result in a box of size `[1.0, 2.0, 3.0]` in world space.

### JSON Schema

* **JSON schema**: [glTF.KHR_geometry.schema.json](schema/glTF.KHR_geometry.schema.json)

### Object Model

With consideration to the glTF 2.0 Asset Object Model Specification document, the following pointer templates represent mutable properties defined in this extension.

| Pointer | Type|
|-|-|
| `/extensions/KHR_geometry/geometries/{}/box/size` | `float3`|
| `/extensions/KHR_geometry/geometries/{}/capsule/height` | `float`|
| `/extensions/KHR_geometry/geometries/{}/capsule/radiusBottom` | `float`|
| `/extensions/KHR_geometry/geometries/{}/capsule/radiusTop` | `float`|
| `/extensions/KHR_geometry/geometries/{}/cylinder/height` | `float`|
| `/extensions/KHR_geometry/geometries/{}/cylinder/radiusBottom` | `float`|
| `/extensions/KHR_geometry/geometries/{}/cylinder/radiusTop` | `float`|
| `/extensions/KHR_geometry/geometries/{}/sphere/radius` | `float`|
| `/extensions/KHR_geometry/geometries/{}/mesh/weights/{}` | `float`|
| `/extensions/KHR_geometry/geometries/{}/mesh/weights` | `float[]`|

Additional read-only properties

| Pointer | Type|
|-|-|
| `/extensions/KHR_geometry/geometries.length` | `int`|
| `/extensions/KHR_geometry/geometries/{}/mesh/weights.length` | `int`|

## Known Implementations

[Blender importer/exporter](https://github.com/eoineoineoin/glTF_Physics_Blender_Exporter)

[Babylon.js importer](https://github.com/eoineoineoin/glTF_Physics_Babylon)

[Godot importer](https://github.com/eoineoineoin/glTF_Physics_Godot_Importer)

## Appendix: Full Khronos Copyright Statement

Copyright 2021-2023 The Khronos Group Inc.

This specification is protected by copyright laws and contains material proprietary
to Khronos. Except as described by these terms, it or any components
may not be reproduced, republished, distributed, transmitted, displayed, broadcast,
or otherwise exploited in any manner without the express prior written permission
of Khronos.

This specification has been created under the Khronos Intellectual Property Rights
Policy, which is Attachment A of the Khronos Group Membership Agreement available at
https://www.khronos.org/files/member_agreement.pdf. Khronos grants a conditional
copyright license to use and reproduce the unmodified specification for any purpose,
without fee or royalty, EXCEPT no licenses to any patent, trademark or other
intellectual property rights are granted under these terms. Parties desiring to
implement the specification and make use of Khronos trademarks in relation to that
implementation, and receive reciprocal patent license protection under the Khronos
IP Policy must become Adopters under the process defined by Khronos for this specification;
see https://www.khronos.org/conformance/adopters/file-format-adopter-program.

Some parts of this Specification are purely informative and do not define requirements
necessary for compliance and so are outside the Scope of this Specification. These
parts of the Specification are marked as being non-normative, or identified as
**Implementation Notes**.

Where this Specification includes normative references to external documents, only the
specifically identified sections and functionality of those external documents are in
Scope. Requirements defined by external documents not created by Khronos may contain
contributions from non-members of Khronos not covered by the Khronos Intellectual
Property Rights Policy.

Khronos makes no, and expressly disclaims any, representations or warranties,
express or implied, regarding this specification, including, without limitation:
merchantability, fitness for a particular purpose, non-infringement of any
intellectual property, correctness, accuracy, completeness, timeliness, and
reliability. Under no circumstances will Khronos, or any of its Promoters,
Contributors or Members, or their respective partners, officers, directors,
employees, agents or representatives be liable for any damages, whether direct,
indirect, special or consequential damages for lost revenues, lost profits, or
otherwise, arising from or in connection with these materials.

Khronos® and Vulkan® are registered trademarks, and ANARI™, WebGL™, glTF™, NNEF™, OpenVX™,
SPIR™, SPIR&#8209;V™, SYCL™, OpenVG™ and 3D Commerce™ are trademarks of The Khronos Group Inc.
OpenXR™ is a trademark owned by The Khronos Group Inc. and is registered as a trademark in
China, the European Union, Japan and the United Kingdom. OpenCL™ is a trademark of Apple Inc.
and OpenGL® is a registered trademark and the OpenGL ES™ and OpenGL SC™ logos are trademarks
of Hewlett Packard Enterprise used under license by Khronos. ASTC is a trademark of
ARM Holdings PLC. All other product names, trademarks, and/or company names are used solely
for identification and belong to their respective owners.
