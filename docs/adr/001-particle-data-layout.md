# ADR-001: Store particles as a structure of arrays

- Date: 2026-09-03

## Context

The simulator repeatedly updates a large collection of particles during each
time step. The hot paths—force calculation, integration, and collision 
resolution—usually operate on only a subset of each particle's attributes. 
For example, integration needs position and velocity, while a collision test 
commonly needs position, radius, and mass but not a particle's display state 
or identifier.

The particle representation must support predictable traversal, good cache use,
and future SIMD or parallel implementations without making the particle model
hard to work with.

## Decision

Store simulation particle state in a **structure of arrays (SoA)**. Each
attribute is held in a contiguous array indexed by particle index. Arrays are
kept the same length, and the value at an index in every array describes the
same particle.

Conceptually:

```cpp
struct Particles {
    std::vector<float> position_x;
    std::vector<float> position_y;
    std::vector<float> position_z;
    std::vector<float> velocity_x;
    std::vector<float> velocity_y;
    std::vector<float> velocity_z;
    std::vector<float> mass;
    std::vector<float> radius;
};
```

The initial implementation may use separate `x`, `y`, and `z` arrays as shown.
Particle indices are the internal handle used by simulation systems.

## Rationale

SoA lets a pass load only the fields it needs in contiguous memory. This reduces
cache traffic compared with loading full particle objects, improves hardware
prefetching, and maps naturally to vectorized and data-parallel loops. These
properties matter particularly for the O(n²) all-pairs force or collision paths
that an initial n-body simulator is likely to use.

## Alternatives considered

### Array of structures (AoS)

An AoS representation would store one complete `Particle` object per element.
It is straightforward to model, debug, and pass around, and is appropriate when
most operations need every field of every particle. It is not selected because
the simulator's hot passes are field-oriented and would repeatedly fetch unused
attributes.

### AoS with packed vector and scalar fields

Packing positions and velocities into vector types improves the ergonomics of
AoS but does not avoid loading unrelated fields. Alignment and padding can also
increase each particle's stride.

## Consequences

Positive:

- Hot loops read less memory, have more predictable access patterns, and are
  cache friendly.
- Force, integration, and collision systems can operate directly on the arrays
  they require.
- The design is ready for SIMD, multithreading, and GPU-friendly data transfer.

Negative:

- Adding, removing, or reordering a particle must update every particle array
  atomically.
- Per-particle code is less ergonomic than accessing one `Particle` object.
- Invariants on matching array lengths and stable index handling must be tested.

## Implementation notes

- Provide `size()`, `reserve()`, `add_particle()`, and `remove_particle()` on
  the owning `Particles` type so callers cannot mutate individual arrays in a
  way that breaks alignment.
- Use swap-and-pop removal unless simulation semantics require stable ordering.
- Keep a separate index-to-external-ID mapping if stable public identifiers are
  required.
- Revisit the layout after representative benchmarks exist; the decision is
  based on expected n-body access patterns and should be verified with profiles.
