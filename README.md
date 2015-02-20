Commutative rendering
=====================

**Problem**:  Writing efficient and modular graphics code is hard.

# Survey

## Immediate mode

Every object is responsible for drawing itself

#### Pros:

* state is stored in application and graphics system just draws it directly
* Each module is responsible for drawing each object, just loop over everything and draw it
* Can grow democratically, anyone and everyone can add their own creative features; easy to compose functions

#### Cons:

* Inefficient, potentially lots of state changes
* Multiple passes are hard, requires strong coupling across modules

### Examples

Examples include gl-axes, gl-simplicial-complex, gl-surface-plot and gl-scatter-plot.  This approach works fine for small scenes and debug data, but has issues including strong coupling across interactions.

## Retained mode

Have a "3d engine" with some representation of 3D objects, draw by mutating state of 3D engine.

#### Pros:

* Allows for advanced optimizations, batching/mimizing state changes etc.
* Easier to coordinate multipass rendering (shadows, picking, etc.)

#### Cons:

* scene graph/DOM-like API, state stored in application *and* in the engine.  User responsible for keeping multiple states in sync
* Non-extensible, hard to add new features.  Shaders need to coordinate tightly within the retained mode API
* tends toward massive feature bloat ("frameworkitis")
* Splits users into 2 classes: owners & consumers; big power differential

### Examples

Most 3D engines on the market.  Unreal, Unity, THREE.js, babylon.js etc.

# Commutative rendering

Immediate mode code is easier to write and reason about, but performance considerations and multipass rendering push us toward retained mode.

The idea behind commutative rendering is to create an immediate mode API that is fast on top of a lower level graphics API (like WebGL) by buffering and roordering draw calls.  For example, a user might write code that issues draw calls like this:

```
bind(shader1)
draw(objectA)

bind(shader2)
draw(objectB)

bind(shader1)
draw(objectC)

bind(shader2)
draw(objectD)
```

And the commutative rendering layer would translate the code into something like this:

```
bind(shader1)
draw(objectA)
draw(objectC)

bind(shader2)
draw(objectB)
draw(objectD)
```

In order to support this sort of transformation, we would require that each draw operation is **commutative**, that is the order of calling draw does not matter.

This is certainly true for depth-buffered rasterization without blending, but it isn't necessarily true in general.

### Multipass rendering

