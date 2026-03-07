# Description of electric motors

> Why [`Arc<Material>`]? 
- Reuse of the same material (e.g. copper) over multiple wires, even when
deserializing -> serde_mosaic

> Why Box<MotorComponent>?
- Generics: Generics bloat: Motor<StatorWinding<Wire>, StatorCore, RotorWinding<Wire>, RotorCore, RotorMagnet> ... -> Every function which takes a motor would need to be completely generic
- Enum: a) Not user-extensible. b) Adding a new wire type would be a breaking change

Performance issue compensated by caching in SimMotor -> No need to call methods
of wires, slots etc. in a tight loop of a simulation."
"