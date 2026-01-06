# UEVR_FProperty

## Functions

### `property:is_param()`

Whether the property represents a parameter for a [UFunction](types/UFunction.md).

### `property:is_out_param()`

Whether the property represents an out parameter for a [UFunction](types/UFunction.md). Refer to [UEVR_UObject](types/UObject.md) to understand out parameters.

### `property:is_reference_param()`

Whether the property represents a reference parameter for a [UFunction](types/UFunction.md). You may need to check the object passed by reference to see a result.

### `property:is_return_param()`

### `property:is_pod()`

Whether the property represents a Plain Old Data type. POD types can include classes or structs but should only have simple constructors with no virtual functions. Typically non-UObject derived types like FVector, FRotator, FHitResult can be POD types.

### `property:get_property_flags()`

### `property:get_offset()`
