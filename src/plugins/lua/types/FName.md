# UEVR_FName

## What is FName?

FName is a struct that represents a name in Unreal Engine. Nearly all internal naming uses FName rather than strings. 

## Functions

### `name:to_string()`

Returns a Lua string representation of the name.

### `string` to FName conversion

This is handled automatically when calling functions with FName arguments or setting Name properties of structs. You should not need to directly work with FName objects in lua, just keep in mind that for any comparisons in lua and when displaying FName values you still need to convert them to strings. 

### `None`

Many functions require FName arguments, e.g. attaching components requires a Socket or Bone reference by name. For objects that don't have any sockets or bones you would need to provide an FName value of "None" which is a special zero value in the game engine. To pass a "None" value you can provide "None" as a string, provide an empty string `""` or pass `UEVR_FName.new()`. `nil` will not work although the use cases are similar. 




