# Serialization and deserialization

In stem, motors, their components and whole self-contained simulation models can
be serialized and deserialized using the [serde](https://serde.rs/) ecosystem.
For example, this allows defining a material programmatically, storing its
serialized representation somewhere and using it at a later point in time
for a motor. Similarily, it is also possible to write a file (e.g. a .yaml-file)
describing a motor or simulation model and then use it in a program
based on stem. The flexibility granted by serde allows using a huge variety of
different formats such as JSON, YAML, TOML, Pickle, CSV, Postcard, BSON and so
on. Within this book and the crates of stem, the .yaml-format is used throughout
the examples. 

At the end of this page, there is a complete [yaml-example](#motor-yaml-example)
for a motor showcasing the features described in the following sections. The
[stem test database](https://github.com/StefanMathis/stem_test_database)
contains various examples for materials and motors as well.

## Distributed storage in a database using serde_mosaic

## Units

The SI-quantities defined by the [uom](https://crates.io/crates/uom) crate are
used extensively throughout stem to provide type-safe dimensional analysis. When
serializing an uom quantity, only the raw value in base units is stored. In
practice, this means that the serialized representation (in .yaml) of this
struct:

```rust
my_struct = MyLength {length: uom::si::f64::Length::new::<uom::si::length::millimeter>(1.0)};
```
... looks like this:

```yaml
length: 0.001
```

When writing e.g. a motor file by hand, it might be preferable to write the full
quantity instead to increase maintainability. For this reasons, everytime a
quantity gets deserialized in stem, the
[dyn_quantity](https://crates.io/crates/dyn_quantity) crate is involved. This
crate provides a fully-fledged SI unit parser for strings, which allows to
write the .yaml-file of the example above as:

```yaml
length: 1 mm
```

[dyn_quantity](https://crates.io/crates/dyn_quantity) also allows for simple
mathematical operations during parsing, which can be useful to cleanly express a
physical quantity as a term:

```yaml
vacuum_permeability: 4 pi 10^(-7) H/m
```

When deserializing such a field, it is checked if the dimensions given in the
string match that of the quantity. If they don't, the deserialization fails and
an error pointing out the issue is returned.

Of course, it is still possible to use the "raw" representation from the initial
example when manually writing a file. In this case, no dimensional checks are
applied.

<a id="motor-yaml-example"></a>
## Example of a motor described in .yaml

```yaml
MotorRot:
  name: FLS-364 V1
  stator: 
    core:
      air_gap_radius: 55 mm
      yoke_radius: 85 mm
      axial_length: 165 mm
      axial_coil_overhang: 0 mm
      iron_fill_factor: 0.95
      material:
        name: M270-50A
      pole_pairs: 2
      skew_angle: 0.0
      air_gap: 
        AirGapSlotted:
          slots: 36
          starts_in_slot_middle: false
          carter_factor_model: Bin12
          slot:
            SlotTrapezoidSemi:
              bottom_width: 9.21 mm
              opening_width: 2 mm
              height: 17.75 mm
              opening_height: 0.75 mm
              angle_slot: 10 deg
              bottom_radius: 0.5 mm
              top_radius: 1 mm
              opening_radius: 0.0 mm
              consider_tooth_tip_leakage: true
    winding:
      WindingDistributed:
        slots: 36
        pole_pairs: 2
        phases: 3
        layers: 1
        coil_span_reduction: 0
        zone_span_variation: 0
        turns_per_coil: 30
        parallel_paths: 1
        connection: Star
        end_winding_leakage_coefficient: 0.25
        wire:
          WireStranded:
            strand_list:
              - wire:
                  WireRound:
                    conductor_material:
                      name: "Copper"
                    outer_diameter: 0.67 mm
                    inner_diameter: 0.0 mm
                    insulation_thickness: 0.05025 mm
                number_wires: 2
              - wire:
                  WireRound:
                    conductor_material:
                      name: "Copper"
                    outer_diameter: 0.71 mm
                    inner_diameter: 0.0 mm
                    insulation_thickness: 0.05325 mm
                number_wires: 3
        zone_plan_method: Tingley
        concentric_coils: false
    surface_magnet: ~
  rotor: 
    core:
      air_gap_radius: 54.4 mm
      yoke_radius: 19 mm
      axial_length: 165 mm
      axial_coil_overhang: 0 mm
      iron_fill_factor: 0.95
      material:
        name: M270-50A
      pole_pairs: 2
      skew_angle: 0.0
      air_gap: 
        AirGapSlotted:
          slots: 28
          starts_in_slot_middle: false
          carter_factor_model: Bin12
          slot:
          SlotTrapezoidSemi:
              bottom_width: 6.76 mm
              top_width: 1.5 mm
              side_top_width: 8 mm
              opening_width: 1.5 mm
              height: 6.79 mm
              opening_height: 0.75 mm
              angle_slot: -360/28 deg
              angle_bottom: 
                bottom_width: 6.76 mm
                side_bottom_width: 6.76 mm
                bottom_height: 0.0 mm
                angle_slot: -360/28 deg
              angle_top:
                top_width: 1.5 mm
                side_top_width: 8 mm
                top_height: 0.5 mm
                angle_slot: -360/28 deg
              bottom_radius: 0.0 mm
              slope_bottom_radius: 0.0 mm
              top_radius: 0.0 mm
              slope_top_radius: 0.0 mm
              opening_radius: 0.0 mm
              consider_tooth_tip_leakage: true
      flux_barrier: 
        FluxBarrierV1Def:
          distance_yoke: 4.5 mm
          air_gap_relief_path: 3.5 mm
          relief_path_length: 1.33 mm
          relief_path_width: 4 mm
          opening_angle: 90 deg
          magnet_space_width: 10 mm
          magnet_space_height: 20 mm
          glue_gap: 0.2 mm
          width_leakage_path: 1.0 mm
          magnet_material: 
            name: NMF-12J 430mT
          fillet_q_side_leakage_space:
    winding:
      WindingCage:
        slots: 28
        pole_pairs: 2
        end_winding_leakage_coefficient: 0.35
        wire:
          WireRectangular:
            conductor_material:
              name: Copper
            height: 5.5 mm
            width: 6.7 mm
            insulation_thickness: 0.0 mm
        end_ring_width: 11 mm
        end_ring_height: 11.2 mm
        consider_current_displacement: true
    surface_magnet: ~
```