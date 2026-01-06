# UEVR_FField

## What is FField?

FField is a struct that represents a field in Unreal Engine. UEVR_FField is an interface representing either an FField or a UObject-derived UField. Unreal Engine version 4.25 introduced FField but older versions can still use UEVR_FField methods thanks to this abstraction.

## Functions

### `field:get_next()`

Returns the next [UEVR_FField*](FField.md) object in the linked list of fields.

### `field:get_fname()`

Returns the [UEVR_FName](FName.md) object of the field name.

### `field:get_class()`

Returns the [UEVR_FFieldClass](FFieldClass.md) object of the field.
