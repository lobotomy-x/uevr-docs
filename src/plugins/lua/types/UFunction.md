# UEVR_UFunction

## What is UFunction?

UFunction is a type that represents a function in Unreal Engine. Functions are usually associated with a UClass and can be called on instances of that class.

UFunction inherits all the functions from [UEVR_UStruct](UStruct.md).

## Functions

### `UEVR_UFunction.static_class()`

Returns the [UEVR_UClass*](UClass.md) descriptor for UFunction.

### `function:call(obj, args_ptr)`

Not implemented correctly. Do not use. Use `obj:FuncName(args...)` instead.

### `function:get_native_function()`

Returns the native function pointer of the UFunction as a `void*`.

### `function:hook_ptr(pre, post)`

Hooks the UFunction with the specified pre and post function pointers. Functions can be `nil` if you only want to use one but not the other. Functions are only hooked per class, i.e. hooking a function does not affect child classes with the same function. You can set multiple hooks per function.

Pre Signature: `bool pre(fn: UFunction*, obj: UObject*, locals: StructObject*, result: void*)`

Return `false` to skip the original function. Not returning at all is equivalent to returning `true`.

Post Signature: `void post(fn: UFunction*, obj: UObject*, locals: StructObject*, result: void*)`

## Getting parameter information

### `function:get_child_properties()`

Inherited from [UEVR_UStruct](UStruct.md). Used to retrieve a list of parameters as a linked list of [UEVR_FField](FField.md) objects.

### `function:find_property(name)`

Inherited from [UEVR_UStruct](UStruct.md). Use this with functions to retrieve parameters as [UEVR_FProperty](FProperty.md) objects. The main use is to identify out parameters.

## Function flags

### `function:get_function_flags()`

Returns the bitmask of the [UFunction Specifiers](https://dev.epicgames.com/documentation/en-us/unreal-engine/ufunctions?application_version=4.27) applied to the function.
Individual values can be checked for by testing `function:get_function_flags() & flag_value ~= 0`

### `function:set_function_flags()`

Allows for manipulation of the bitmask. 

**Set a flag:**
```lua
ufunc:set_function_flags(ufunc:get_function_flags() | flag_value)
```
**Unset a flag:**
```lua
ufunc:set_function_flags(ufunc:get_function_flags() &~ flag_value)
```

See the table below for a list of valid flags. `is_native` is commonly used to make some functions callable. `is_static` is another useful flag as it tells you that the function can be called from the default instance of its class obtained with `get_class_default_object`.

| Name         |         Value |
| :---         |          ---: |
|is_final    |   0x1|
|is_required_api |   0x2|
|is_blueprint_authority_only |   0x4|
|is_blueprint_cosmetic   |   0x8|
|is_net  |   0x40|
|is_net_reliable |   0x80|
|is_net_request  |   0x100|
|is_exec |   0x200|
|is_native   |   0x400|
|is_event    |   0x800|
|is_net_response |   0x1000|
|is_static   |   0x2000|
|is_net_multicast    |   0x4000|
|is_ubergraph_function   |   0x8000|
|is_multicast_delegate   |   0x10000|
|is_public   |   0x20000|
|is_private  |   0x40000|
|is_protected    |   0x80000|
|is_delegate |   0x100000|
|is_net_server   |   0x200000|
|has_out_params  |   0x400000|
|has_defaults    |   0x800000|
|is_net_client   |   0x1000000|
|is_dll_import   |   0x2000000|
|is_blueprint_callable   |   0x4000000|
|is_blueprint_event  |   0x8000000|
|is_blueprint_pure   |   0x10000000|
|is_editor_only  |   0x20000000|
|is_const    |   0x40000000|
|is_net_validate |   0x80000000|
