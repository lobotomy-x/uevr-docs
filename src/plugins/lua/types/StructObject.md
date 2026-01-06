# StructObject

StructObject is a custom UEVR Lua type that represents a struct in Unreal Engine. It's for non-UObject based structs which support reflection.

## Functions

### `StructObject.new(struct: UEVR_UStruct*)`

Constructs a new StructObject from a [UEVR_UStruct](UStruct.md) pointer.

### `StructObject.new(object: UEVR_UObject*)`

Constructs a new StructObject from a [UEVR_UObject](UObject.md) pointer, if the object inherits from UStruct.

## Constructing ScriptStructs from Lua Tables

Alternatively you can create a table with named keys matching the expected struct properties. 
ex:
```lua
-- Simple inline style ScriptStruct creation
object.capsule_component.ShapeColor = {R = 255, G = 0, B = 0, A = 255}

-- Factory style functions to create Enums and ScriptStructs
local EViewTargetBlendFunction = {
   VTBlend_Linear    = 0,
   VTBlend_Cubic     = 1,
   VTBlend_EaseIn    = 2,
   VTBlend_EaseOut   = 3,
   VTBlend_EaseInOut = 4,
}

local function FViewTargetTransitionParams(blendTime, blendFunc, blendExponent, bLockOutGoing)
   return {
      -- use "or" to provide default values
      BlendTime = blendTime or 1.0,
      BlendFunction = blendFunc or 1,
      BlendExp = blendExponent or 1.1,
      bLockOutgoing = bLockOutGoing or false
   }
end

player_controller:ClientSetViewTarget(temp, FViewTargetTransitionParams(1.0, EViewTargetBlendFunction["VTBlend_EaseOut"], 1.1, false))
```
This allows you to handle most data directly in lua and only pass it to the game when calling functions or setting properties, giving you more control over your data and reducing the need for synchronization. There are also a few a known games, usually on custom engine versions, where StructObject creation does not work but tables are fine.


## Transformations
All relevant struct types for transformations support table-based creation. However in the case of Vectors and Rotators you should opt for the corresponding [Vector](Vector.md) or [Quaternion](Quaternion.md) types. Also keep in mind that out arguments can be filled with empty tables and optionally retrieved after calling if an indexed variable set to an empty table is passed. FHitResult is a common ScriptStruct object present in many function signatures, but its almost always an out parameter.
ex:
```lua
local sweep = true
local sweep_hits = {}
component:K2_SetWorldLocation(new_location, sweep, sweep_hits, false)
-- do something with the result 
local sweep_results = sweep.result

-- Not using sweep so the results can be ignored 
component:K2_SetWorldLocation(new_location, false, {}, false)
```

