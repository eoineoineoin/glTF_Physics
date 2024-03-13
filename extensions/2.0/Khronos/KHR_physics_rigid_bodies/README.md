# KHR_physics_rigid_bodies

## Contributors <!-- omit in toc -->

* Eoin Mcloughlin, Microsoft, <mailto:eomcl@microsoft.com>
* Rory Mullane, Microsoft, <mailto:romul@microsoft.com>
* George Tian, Microsoft, <mailto:geotian@microsoft.com>
* Aaron Franke, Godot Engine, <mailto:arnfranke@yahoo.com>

## Status <!-- omit in toc -->

Draft

## Dependencies <!-- omit in toc -->

Written against glTF 2.0 spec.

This specification depends on `KHR_collision_shapes` to describe geometries used for collision detection.

This specification will be updated to add support for the forthcoming glTF Interactivity and Animation Pointer (glTF Object Model) extensions. These are expected to be additional, non-breaking changes.

## Table of Contents <!-- omit in toc -->

- [Overview](#overview)
- [Units](#units)
- [glTF Schema Updates](#gltf-schema-updates)
  - [Motions](#motions)
  - [Colliders](#colliders)
  - [Physics Materials](#physics-materials)
  - [Collision Filtering](#collision-filtering)
  - [Triggers](#triggers)
  - [Joints](#joints)
- [JSON Schema](#json-schema)
- [Known Implementations](#known-implementations)
- [Validator](#validator)
- [Appendix: Constraint Limit Metrics](#appendix-constraint-limit-metrics)


## Overview

This extension defines a set of properties which may be added to glTF nodes, making them suitable for rigid body dynamics simulation. Such a simulation may update node transforms, effectively animating node transforms procedurally in a physically plausible manner.

Properties added by this extension include:

- Collision shapes, used to determine when nodes are physically overlapping.
  - Collision filters, which allow for control over which pairs of nodes should collide.
  - Physics materials, which describe how pairs of objects react to collisions.
- Joints, which describe physical connections between nodes.
- Mass and velocity, describing the movement of nodes.

The following diagram augments the overview diagram in the [main glTF repository](https://github.com/KhronosGroup/glTF) and shows the relationships between new properties added by this extension and existing objects in the glTF specification.

![Object relationship diagram](figures/Overview.png)

Note, amongst rigid body engines which exist today, there are a wide variety of approximations and solving strategies in use which result in differing behavior. As such, the same asset is very likely to behave differently in different simulation engines or with different settings applied to one simulation engine. An implementation should make a best effort to implement this specification within those limitations; this requires some discretion on the part of the implementer - for example, a video game is very likely willing to accept inaccuracies which would be unacceptable in a robotic training application.

### Units

Units used in this specification are the same as those in the [glTF specification](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units); some additional units are used by this extension:

| Property | Units|
|-|-|
|`motion.mass`|Kilograms (kg)|
|`motion.inertiaDiagonal`|Kilogram meter squared (kg·m<sup>2</sup>)|
|`motion.linearVelocity`|Meter per second (m·s<sup>-1</sup>)|
|`motion.angularVelocity`|Radian per second (rad·s<sup>-1</sup>)|
|`joint_limit.stiffness`, <br /> `joint_drive.stiffness`|Newton per meter (N·m<sup>-1</sup>) for linear limits <br /> Newton meter per radian (N·m·rad<sup>-1</sup>) for angular limits|
|`joint_limit.damping`, <br /> `joint_drive.damping`|Newton second per meter (N·s·m<sup>-1</sup>) for linear limits <br />Newton second meter per radian (N·s·m·rad<sup>-1</sup>) for angular limits|

## glTF Schema Updates

The `KHR_physics_rigid_bodies` extension may be added to any `node` to define one or more of the following properties:

| |Type|Description|
|-|-|-|
|**motion**|`object`|Allows the simulation to move this node, describing parameters for that motion.|
|**collider**|`object`|Describes the physical representation of a node's shape.|
|**trigger**|`object`|Describes a volume which can detect collisions, but not react to them.|
|**joint**|`object`|Constrains the motion of this node relative to another.|


### Motions

If a `node` has `motion` properties, that node should be represented by a rigid body in the simulation engine and the node's local transform should be updated by the physics engine after every simulation step.

As the simulation engine updates the local transform of a node, all descendant nodes should move with that node - i.e. the physics engine should treat them as part of a single rigid body. However, if a descendant node has its own `motion` properties, that node must be treated as an independent rigid body during simulation - there is no implicit requirement that it follows its 'parent' rigid body.

If a rigid body node's transform is animated by animations in the file, those animations should take priority over the physics simulation. Rigid bodies should follow the transforms provided by the animations. A simulation engine has several options for how this can be achieved at the discretion of the implementer - for example, the rigid body may be instantaneously teleported without traversing the intermediate space. Alternatively, a simulation engine may set velocities or joint motors such that the node reaches the target transform within the simulation step.

Rigid body motions have the following properties:

| |Type|Description|
|-|-|-|
|**isKinematic**|`boolean`|Treat the rigid body as having infinite mass. Its velocity will be constant during simulation.|
|**mass**|`number`|The mass of the rigid body. Larger values imply the rigid body is harder to move.|
|**inertiaOrientation**|`number[4]`|The quaternion rotating from inertia major axis space to body space.|
|**inertiaDiagonal**|`number[3]`|The principal moments of inertia. Larger values imply the rigid body is harder to rotate.|
|**centerOfMass**|`number[3]`|Center of mass of the rigid body in local space.|
|**linearVelocity**|`number[3]`|Initial linear velocity of the rigid body in local space.|
|**angularVelocity**|`number[3]`|Initial angular velocity of the rigid body in local space.|

If not provided, the mass and inertia properties should be calculated by the simulation engine. These values are typically derived from the volume of the collision geometry used by the rigid body. The 3x3 inertia tensor can be calculated as the product of the `inertiaOrientation` (in matrix form) and the matrix whose diagonal is `inertiaDiagonal`.

JSON is unable to represent infinite values; however, a value of infinity is useful for both `mass` and `inertiaDiagonal`. As zero is an *invalid* value for these properties, a value of zero in `mass` or any of the components of `inertiaDiagnal` should be understood to represent a value of infinity.

### Colliders

Pairs of triangulated meshes are typically unsuitable for collision detection. As such, this extension adds an additional `collider` property to a node. This property is used to describe the geometry and collision response of this node.


The `collider` property supplies three fields; the `shape` property indexes into the set of top level collision shapes (provided by `KHR_collision_shapes`) and describes the collision volume used by that node. The `physicsMaterial` indexes into the top level set of physics materials (see the "[Physics Materials](#physics-materials)" section of this document.) Finally, the `collisionFilter` indexes into the top level set of collision filters (see the "Collision Filtering" section of this document).

| |Type|Description|
|-|-|-|
|**shape**|`integer`|The index of a top level `KHR_collision_shapes.shape`, which provides the geometry of the collider.|
|**collisionFilter**|`integer`|Indexes into the top level `collisionFilters` and describes a filter which determines if this collider should perform collision detection against another collider.|
|**physicsMaterial**|`integer`|Indexes into the top level `physicsMaterials` and describes how the collider should respond to collisions.|

If the node is part of a dynamic rigid body (i.e. itself or an ascendant has `motion` properties) then the collider belongs to that rigid body and must move with it during simulation. Otherwise the collider exists as a static object in the physics simulation which can be collided with but must not be moved as a result of solving collisions or joints. A rigid body may have multiple descendent `collider` nodes.

Implementations of this extension should ensure that collider transforms are always kept in sync with node transforms - for example animated node transforms should be applied to the physics engine (even for static colliders).

Note, `convex` and `trimesh` colliders can impose a large computational cost when converting to native types if the source mesh contains many vertices. In addition, real-time engines generally recommend against allowing collisions between pairs of `trimesh` objects. For best performance and behavior, consult the manual for the physics simulation engine you are using.

### Physics Materials

When a pair of nodes collide with each other, additional properties are needed to determine the collision response. This response is partly controlled by the physics materials of each collider.

The top level array of `physicsMaterials` is provided by adding the `KHR_physics_rigid_bodies` extension to the root `glTF` object. If a collider has no physics material assigned, the simulation engine may choose any appropriate default values.

Physics materials offer the following properties:

| |Type|Description|
|-|-|-|
|**staticFriction**|`number`|The friction used when an object is laying still on a surface.<br/>Typical range from 0 to 1.|
|**dynamicFriction**|`number`|The friction used when already moving.<br/>Typical range from 0 to 1.|
|**restitution**|`number`|Coefficient of restitution.<br/>Typical range from 0 to 1.|
|**frictionCombine**|`string`|How to combine two friction values.<br/>"average", "minimum", "maximum", or "multiply".|
|**restitutionCombine**|`string`|How to combine two restitution values.<br/>"average", "minimum", "maximum", or "multiply".|

For handling friction parameters, a physics simulation should use a Coulomb friction model. The friction coefficient to use is be determined by the relative velocity of the surfaces perpendicular to the contact normal. When this velocity is sufficiently close to zero, the value of `staticFriction` should be used, otherwise, `dynamicFriction` should be used.

When a pair of physics materials interact during a simulation step, the applied friction and restitution values are based on their "combine" policies:

- If either uses "average" : The two values should be averaged.
- Else if either uses "minimum" : The smallest of the two values should be used.
- Else if either uses "maximum" : The largest of the two values should be used.
- Else if either uses "multiply" : The two values should be multiplied with each other.

### Collision Filtering

Colliders from distinct rigid bodies should generate a collision response when they are sufficiently close together to be considered in contact.

In some scenarios this is undesirable, so a collision filter allows control over which pairs of `collider` objects may interact. An array of `collisionFilters` are provided by the `KHR_physics_rigid_bodies` extension on the root `glTF` object. Each filter contains a subset of the fields:

| |Type|Description|
|-|-|-|
|**collisionSystems**|`string[1-*]`|An array of arbitrary strings indicating the "system" a node is a member of.|
|**notCollideWithSystems**|`string[1-*]`|An array of strings representing the systems which this node can _not_ collide with|
|**collideWithSystems**|`string[1-*]`|An array of strings representing the systems which this node can collide with|

Both `collideWithSystems` and `notCollideWithSystems` are provided so that users can override the default collision behavior with minimal configuration -- only one of these should be specified per object. Note, given knowledge of all the systems in a scene and one of the values `notCollideWithSystems`/`collideWithSystems` the unspecified field can be calculated as the inverse of the other.

`notCollideWithSystems` is useful for an object which should collide with everything except those listed in `notCollideWithSystems` (i.e., used to opt-out of collisions) while `collideWithSystems` is the inverse -- the collider should not collide with any other collider except those listed in `collideWithSystems`

A node `A` will collide with node `B` if `A.collisionSystem ⊆ B.collideWithSystems && A.collisionSystem ⊄ B.notCollideWithSystems`

This can generate asymmetric states - `A` might determine that it _does_ collide with `B`, but `B` may determine that it _does not_ collide with `A`. As the default behavior is that collision should be enabled, both `doesCollide(A, B)` and `doesCollide(B, A)` tests should be performed and collision should not occur if either returns false.


### Triggers

A useful construct in a physics engine is a collision volume which does not generate impulses when overlapping with other volumes but do generate events which execute application-specific logic. These objects are typically called "triggers", "sensors", "phantoms", or "overlap volumes" in physics simulation engines. Triggers allow specifying such volumes either as a single shape or combination of shapes.

A trigger is added to a node by specifying the `trigger` property.

A `trigger` may specify a `shape` property which references a geometric shape defined by the `KHR_collision_shapes` extension as well as an optional `collisionFilter` parameter, with the same semantics as a `collider`.

Alternatively, a `trigger` may have a `nodes` property, which is an array of glTF nodes which make up a compound trigger on this glTF node. The nodes in this array must be descendent nodes which must have `trigger` properties.

| |Type|Description|
| - | - | -|
|**shape**|`integer`| The index of a top level `KHR_collision_shapes.shape`, which provides the geometry of the trigger.|
|**nodes**|`integer[1-*]`|For compound triggers, the set of descendant glTF nodes with a trigger property that make up this compound trigger.|
|**collisionFilter**|`integer`|Indexes into the top level `collisionFilters` and describes a filter which determines if this collider should perform collision detection against another collider.|

Describing the precise mechanism by which overlap events are generated and what occurs as a result is beyond the scope of this specification; simulation software will typically output overlap begin/end events as an output from the simulation step, which is hooked into application-specific business logic.

### Joints

A `node` may have a `joint` property, which describes a physical connection to another object, constraining the relative motion of those two objects in some manner. Rather than defining explicit types of connections, joints are composed of simple primitive limits, which can be composed to build complex joints.

A `joint` is composed of:

- Two attachment frames, defining the position and orientation of the joint pivots in each object.
- A set of limits, which restrict the relative motion of the attachment frames.
- An optional set of drives, which can apply forces to the attachment frames.

The attachment frames are specified using the transforms of `node` objects; the first of which is the `node` of the `joint` object itself. A `joint` must have a `connectedNode` property, which is the index of the `node` which supplies the second attachment frame. For each of these frames, the relative transform between the `node` and the first parent `motion` (or the simulation's fixed reference frame, if no such `motion` exists) define the constraint space in that rigid body. In order for the joint to have any effect on the simulation, at least one of the pair of nodes or its ancestors should have `motion` properties.

A node's `joint` must specify a `joint` property, which indexes into the top level array of `physicsJoints` inside the `KHR_physics_rigid_bodies` extension object. This object describes the limits and drives utilized by the joint in a shareable manner.

A joint description must contain one of more `joint_limit` objects and zero or more `joint_drive` objects. Each of the limit objects remove some of the relative movement permitted between the two attachment frames, while the drive objects apply forces to achieve a relative transform or velocity between the attachment frames.

If a joint were to eliminate all degrees of freedom, the physics simulation should attempt to move the `motion` nodes such that the transforms of the constrained child nodes (i.e. the `joint` node and the node at index `connectedNode`) become aligned with each other in world space. <!--TODO: remove?-->

Each `joint_limit` contains the following properties:

| |Type|Description|
|-|-|-|
|**linearAxes**|`integer[1..3]`|The linear axes to constrain (0=X, 1=Y, 2=Z).|
|**angularAxes**|`integer[1..3]`|The angular axes to constrain (0=X, 1=Y, 2=Z).|
|**min**|`number`|The minimum allowed relative distance/angle.|
|**max**|`number`|The maximum allowed relative distance/angle.|
|**stiffness**|`number`|Optional softness of the limits when beyond the limits.|
|**damping**|`number`|Optional spring damping applied when beyond the limits.|

Each limit must provide either `linearAxes` or `angularAxes`, declaring which are restricted. The indices in these arrays refer to the columns of the basis defined by the attachment frame of the joint and as such, must be in the range 0 to 2. The number of axes determines whether the limit should be a 1, 2, or 3 dimensional constraint as follows:

* A 1D linear constraint should keep the world-space translation of the attachment frames within the signed distance from the infinite plane spanned by the other two axes.
* A 2D linear constraint should keep the attachment frame translations within a distance from the infinite line along the remaining axis.
* A 3D linear constraint should keep the attachment frame translations within a distance from each other.
* A 1D angular constraint should restrict the attachment frame rotation about that axis, as in a universal joint.
* A 2D angular constraint should restrict the attachment frame rotations to a cone oriented along the remaining axis.

Each limit contains a `min` and `max` parameter, describing the range of allowed difference between the two node transforms - within this range, the constraint is considered non-violating and no corrective forces are applied. These values represent a _distance_ in meters for linear constraints, or an _angle_ in radians for angular constraints.

Additionally, each `joint_limit` has an optional `stiffness` and `damping` which specify the proportion of the recovery applied to the limit. By default, an infinite spring constant is assumed, implying hard limits. Specifying a finite stiffness will cause the constraint to become soft at the limits.

This approach of building joints from a set of individual constraints is flexible enough to allow for many types of bilateral joints. For example, a hinged door can be constructed by locating the attachment frames at the point where the physical hinge would be on each body, adding a 3D linear constraint with zero maximum distance, a 1D angular constraint with `min`/`max` describing the swing of the door around it's vertical axis, and a 2D angular constraint with zero limits about the remaining two axes.

Addition of drive objects to a joint allows the joint to apply additional forces to modify the relative transform between the joint object and the connected node. A `joint_drive` object models a forced, damped spring and contains the following properties:

| |Type|Description|
|-|-|-|
|**type**|`string`|Determines if the drive affects is a `linear` or `angular` drive|
|**mode**|`string`|Determines if the drive is operating in `force` or `acceleration` mode|
|**axis**|`integer`|The index of the axis which this drive affects|
|**maxForce**|`number`|The maximum force that the drive can apply|
|**positionTarget**|`number`|The desired relative target between the pivot axes|
|**velocityTarget**|`number`|The desired relative velocity of the pivot axes|
|**stiffness**|`number`|The drive's stiffness, used to achieve the position target|
|**damping**|`number`|The damping factor applied to reach the velocity target|

Each `joint_drive` describes a force applied to one degree of freedom in constraint space, specified with a combination of the `type` and `axis` parameters and drives to either a target position, velocity, or both. The drive force is proportional to `stiffness * (positionTarget - positionCurrent) + damping * (velocityTarget - velocityCurrent)` where `positionCurrent` and `velocityCurrent` are the signed values of the position and velocity of the connected node in joint space. To assist with tuning the drive parameters, a drive can be configured to be in an `acceleration` `mode` which scales the force by the effective mass of the driven degree of freedom. This mode is typically easier to tune to achieve the desired behaviour, particularly in scenarios where the masses of the connected nodes are not known in advance.


### JSON Schema

* **JSON schema**: [glTF.KHR_physics_rigid_bodies.schema.json](schema/glTF.KHR_physics_rigid_bodies.schema.json)

## Known Implementations

[Blender importer/exporter](https://github.com/eoineoineoin/glTF_Physics_Blender_Exporter)

[Babylon.js importer](https://github.com/eoineoineoin/glTF_Physics_Babylon)

[Godot importer](https://github.com/eoineoineoin/glTF_Physics_Godot_Importer)

## Validator

[glTF validator](https://github.com/eoineoineoin/glTF-Validator)


## Appendix: Constraint Limit Metrics

*This section is non-normative.*

To determine if a particular constraint limit is violated, it is useful to determine a metric for each type of limit. For each of these limits, the constraint limit is considered non-violating if the metric evaluates to a value in the range (`min`, `max`).

|Type|Metric|
|-|-|
|linearLimits=[i]|$e_i \cdot (t_b - t_a)$|
|linearLimits=[i,j]|$\lvert(t_b - t_a) - (e_k \cdot (t_b - t_a))e_k\rvert$|
|linearLimits=[i,j,k]|$\lvert(t_b - t_a)\rvert$|
|angularLimits=[i]|$2\arccos(\mathrm{Tw_i(q_a^* q_b)})$|
|angularLimits=[i,j]|$\arccos(q_a e_k \cdot q_b e_k)$|
|angularLimits=[i,j,k]|$2\arccos(\mathrm{Re}(q_a^* q_b))$

Where:
- $e_i$ is the $i$-th basis vector.
- $t_a$ is the translation of the joint node in world space.
- $t_b$ is the translation of the attached node in world space.
- $q_a$ is the orientation of the joint node in world space.
- $q_b$ is the orientation of the attached node in world space.
- $\mathrm{Tw_i}$ is the function which returns the twist component of the twist-swing decomposition of a quaternion.
- $\mathrm{Re}$ is the function which extracts the real component of a quaternion.
