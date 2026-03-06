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

### Serialization

The SI-quantities defined by the [uom](https://crates.io/crates/uom) crate are
used extensively throughout stem to provide type-safe dimensional analysis. When
serializing an uom `Quantity` using the `Serialize` implementation provided by
uom, only the raw value in base units is stored. In practice, this means that
the serialized representation (in .yaml) of this struct:

```
my_length = MyLength {length: uom::si::f64::Length::new::<uom::si::length::millimeter>(1.0)};
```
... looks like this:

```yaml
length: 0.001
```

Sometimes, it might be preferable to serialize the `length` field together with
its SI units as a string. For this purpose, the
[dyn_quantity](https://crates.io/crates/dyn_quantity) crate offers the
[`serialize_quantity`](https://docs.rs/dyn_quantity/latest/dyn_quantity/quantity/serde_impl/fn.serialize_quantity.html)
function, which is used for all `Quantity` fields throughout stem:

```rust
#[derive(Serialize)]
struct LengthWrapper {
    #[serde(deserialize_with = "serialize_quantity")]
    length: Length,
}
```

When used in conjunction with the
[`serialize_with_units`](https://docs.rs/dyn_quantity/latest/dyn_quantity/quantity/serde_impl/fn.serialize_with_units.html) wrapper (also from [dyn_quantity](https://crates.io/crates/dyn_quantity)),
serializing `LengthWrapper` adds the units to the serialized representation:

```rust
let length = LengthWrapper {length: Length::new::<millimeter>(1.0)};
serialize_with_units(||{serde_yaml::to_string(&length)}).expect("serialization succeeds");
```

results in

```yaml
length: 0.001 m
```

If this wrapper is not used, a `Quantity` gets serialized as its raw value
(matching default uom behaviour). Due to the reduced memory footprint, this
behaviour is preferable if the serialized representation doesn't need to be
human-readable.

```rust
let length = LengthWrapper {length: Length::new::<millimeter>(1.0)};
serde_yaml::to_string(&length);
```

results in

```yaml
length: 0.001
```

### Deserialization

After serializing a quantity together with its units using the aforementioned
mechanism, it must of course be possible to deserialize the resulting string
back into a `Quantity`. Similarily, when e.g. writing the serialized
representation of a motor by hand, it might be preferable to write the full
quantity instead to increase maintainability.

For this reasons, everytime a quantity gets deserialized in stem, the
[`deserialize_quantity`](https://docs.rs/dyn_quantity/latest/dyn_quantity/quantity/serde_impl/fn.deserialize_quantity.html)
function is involved. It is a fully-fledged SI unit parser for strings, which
allows to write the .yaml-file of the example above as:

```yaml
length: 1 mm
```

The corresponding struct definition looks like this:
```rust
#[derive(Deserialize)]
struct LengthWrapper {
    #[serde(deserialize_with = "deserialize_quantity")]
    length: Length,
}
```

`deserialize_quantity` also allows for simple mathematical operations during
parsing, which can be useful to cleanly express a physical quantity as a term:

```yaml
vacuum_permeability: 4 pi 10^(-7) H/m
```

When deserializing a field annotated with `deserialize_quantity`, it is checked
if the dimensions given in the string match that of the quantity. If they don't,
the deserialization fails and an error pointing out the issue is returned.

Of course, it is still possible to use the "raw" representation from the initial
example when manually writing a file. In this case, no dimensional checks are
applied.

<a id="motor-yaml-example"></a>
## Example of a motor described in .yaml

TODO: Copy-paste content of https://github.com/StefanMathis/stem_test_database/blob/main/src/RotMotor/FLS-364%20V1.yaml once "stem_motor" has been published.
