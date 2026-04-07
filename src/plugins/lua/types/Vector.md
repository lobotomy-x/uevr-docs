# Vectors

## What are Vectors?

In mathematics vectors are objects with both magnitude and direction that can have any number of dimensions but most commonly are used for 2D and 3D spatial calculations. Vectors in Unreal Engine function as geometric vectors but are also concrete data structs holding 2, 3, or 4 values typically representing transformation data. The most common form, `Vector` contains the `X`, `Y`, and `Z` components representing positions. Unless otherwise specified, `Vector` will refer to this version. Additional types, `Vector2D` and `Vector4`, are used for screen-space transformations and material functions but are less likely to be encountered. Vectors in UE4 use `float` values while in UE5 they use `double` precision floats. UEVR bindings have `float` and `double` variants denoted by their suffix. Lua numbers are `double` precision by default but will be converted to the appropriate type when calling game functions. However, tables have higher overhead and less functionality so they should generally be avoided for any computation. 

## UEVR Vectors

UEVR offers an assortment of Vector types with various use cases. Versions prefixed with UEVR are custom struct types used for getting VR transformation data. The other types are bindings for the [GLM math library](https://github.com/g-truc/glm/). These GLM versions offer more functionality and should be used as much as possible. UEVR types can be converted to this type with `as_full_binding` except for Rotators which use `cast_as_vector`

# Vector2
**`Vector2f`**  
The most common `Vector2` variant. [ImGui](additional-bindings/imgui.md) size and position values are expected as `Vector2f`. 

**`Vector2d`** 
Note that the d here is for `double` unlike `Vector2D` in Unreal that means "2-Dimensions". This version is not used anywhere at this time.

**`UEVR_Vector2f`**
Used for reading analogue stick data from VR controllers.

# Vector3
**`Vector3f`** / **`Vector3d`**

These are the most important variants as they provide automatic bindings to Unreal Engine's `Vector` and `Rotator` structs and can be used instead of [StructObject](StructObject.md) or tables when calling functions or setting properties. This is also handled automatically when getting values meaning that when you call a game function or read a property returning a `Vector` or `Rotator` you can immediately use the `Vector3f` functions. Any location, rotation, or scale values can use `Vector3f` with the only exception being the `Rotation` member of `Transform` structs which is a `Quat` 

There is no reason to fetch the actual ScriptStruct. The UEVR already does so to create the correct game object for Vectors
`ScriptStruct /Script/CoreUObject.Vector`


**`UEVR_Vector3f`**
Position data for VR devices that should be read by providing an empty UEVR_Vector3f. 
ex:
```lua
local function get_hmd_pose()
	local hmd_pos = UEVR_Vector3.new()
	vr.get_standing_origin(hmd_pos)
	return hmd_pos:as_full_binding()
end

local function get_left_pose()
	local vr = uevr.params.vr
	local pos, rot = UEVR_Vector3f.new(), UEVR_Quaternionf.new()
	vr.get_pose(vr.get_left_controller_index(), pos, rot)
	return pos:as_full_binding(), rot:as_full_binding()
end
```

**`UEVR_Rotatorf`**
Internally provides HMD rotation to the "on_early_calculate_stereo_view_offset", "on_pre_calculate_stereo_view_offset", and "on_post_calculate_stereo_view_offset" callbacks. However this is cast to `Vector3f` before its made accessible to lua. No other usage. You *can* use it if you want to but it lacks functionality. 

```lua
local rot = UEVR_Rotatorf.new()
rot.pitch = 0.0
rot.yaw = 0.0
rot.roll = 0.0
rot:cast_to_vector()
```

# Vector4
**`Vector4f`**
Used for ImGui colors. No UE bindings but can be useful for calculations. If used for RGBA ImGui colors you can directly take the values as `LinearColor` structs in Unreal. These are less common than `Color` structs which range from 0 - 255. If converting ImGui colors to UE you can multiply each component by 255 for a close approximation and call `math.floor`

**`Vector4d`**
No particular usage. Can be used as an intermediate holder for UE `Vector4` or `Quat` if you want access to the glm functions. If you are not transforming the data you may prefer to simply use a table since those support `doubles`.

### Components

Internally `Vector3f` has `x`, `y`, and `z` components but to make things easier you can access each component with the following aliases. 

- x : `x` or `X` or `Pitch`
- y : `y` or `Y` or `Yaw`
- z : `z` or `Z` or `Roll `

ex:
```lua
local v = pawn:K2_GetActorLocation()
local x = v.x
local y = v.Y
local r = pawn:K2_GetActorRotation()
local yaw = r.Yaw
yaw == r.Y
```

`Vector4f` can take `w` or `W` for its 4th parameter. 

**`Vector2f` does not support `X` and `Y` and only takes `x` and `y` at this time. This is very important to know if using it in combination with Unreal Engine `Vector2D` which only supports `X` and `Y`. This especially important when working with ImGui.
ex:

```lua
 local out = {}
 pc = pc or api:get_player_controller(0)
 GameplayStatics:ProjectWorldToScreen(pc, vec3, out, true)
 local vec2 = Vector2f.new(out.result.X, out.result.Y)
```


As with any table or userdata object in lua it is possible to access a component either with dot notation `vec.x` or bracket notation `vec["x"]` however you cannot use `pairs`, `ipairs` or `next`. 
 ex:
 ```lua
 local function vector_unpack(v)
    return v.x, v.y, v.z or nil, v.w or nil
end

```
Note that the term *component-wise* will be used to mean performing an operation given two vectors on each set of components individually, e.g. 
```lua
	local vec3 = Vector3f.new(vec1.x * vec2.x, vec1.y * vec2.y, vec1.z * vec2.z) 
```
## Constructor Functions

### `Vector3f.new(x, y, z)`

Constructs a new `Vector3f`. You must provide some non-nil value for each component

ex:
```lua
local vec = Vector3f.new(1.0, 0.0, 0.0)
```

### `Vector3f:set(x, y, z)`

Takes an existing `Vector3f` and sets the values directly. Similar usage as `new` but should use the colon operator to refer to `self`.
```lua
local vec = Vector3f.new(1.0, 0.0, 0.0)
vec:set(2.0, 0.5, 1.0)
-- or
Vector3f.set(vec, 1.0, 1.0, 1.0)
```

This operation returns itself which means that it can be used in place or as a return value.
ex:
```lua
-- components of vec2 are capitalized which will allow a table or StructObject to be passed
local function component_mult(vec1, vec2)
    return vec1:set(vec1.x * vec2.X, vec1.y * vec2.Y, vec1.z * vec2.Z)
end
```

### `Vector3f:clone()`

Copies data into a new `Vector3f` variable. Can use the colon operator to refer to `self`. This is faster than copying data to a table although if you don't actually need both objects you may as well use `set`. If you are storing the data in a live table or you want to manually calculate an offset position without directly passing the result to your next step it may be useful as this will not affect the original data and is faster to access than reading from Unreal.

ex:
```lua
local vec = Vector3f.new(1.0, 0.0, 0.0)
local vec2 = vec:clone()
```

# MetaMethods

### `Vector2f`, `Vector2d`, `Vector3f`, `Vector3d`, `Vector4f`, `Vector4d`

## `Vector3f.__add(lhs_vec, rhs_vec)`

Adds two `Vector3f` values together. Order does not matter.

This can be performed with the `+` operator.

ex:
`local vec_sum = v1 + v2
 `

### `Vector3f.__sub(lhs_vec, rhs_vec)`

Subtracts a `Vector3f` from another `Vector3f`. Note that order of operations matters with some Quaternion and Vector mathematics. In this case, reversing the order of operations will return a vector in the opposite direction. 
`lhs - rhs = - (rhs - lhs) = -rhs + lhs` 

This can be performed with the `-` operator.
ex:
```lua
local vec_diff = v1 - v2
```

### `Vector3f.__mul(lhs_vec, scalar)`

Multiplies a `Vector3f` value by a scalar. Order does not matter from a math perspective 

This can be performed with the `*` operator. 
ex:
```lua
local vec_mul = v1 * scalar`
```
## Functions

### `Vector2f`, `Vector2d`, `Vector3f`, `Vector3d`, `Vector4f`, `Vector4d`


### `Vector3f:length()`

Returns the `float` or `double` magnitude calculated as the sum of its squared components. If magnitude is equal to 1.0 then you can assume this is a unit vector representing a rotation.

This can be used to get the distance between two positions. Note that while order matters for the sign of the vector produced during subtraction, the magnitude will be the same, therefore when used for distance the order of subtraction does not matter.
ex:
```lua
local v1 = pawn:K2_GetActorLocation()
local v2 = other_actor:K2_GetActorLocation()
local distance = (v1 - v2):length()
```
Or to extrapolate to a point past a known position along the same line of sight

```lua
local function extrapolate(v1, v2, distance)
    return v2 +  (v2 - v1) * distance
end
```

### `Vector3f:normalize()`

Normalizes components so that length is 1.0 making it a unit vector representing a rotation. This modifies the original variable meaning there is no return value. This is a useful step when working with rotators
```lua
-- set rotation to forward vector to avoid division by 0
-- GLM already actually handles this under the hood but you get the idea
local function safe_normalize(v)
    v:normalize()
     return v > 0.0001 and v
     or v:set(1, 0, 0) 
end
```

### `Vector3f:normalized()`

Normalizes components so that length is 1.0 making it a unit vector. This returns a new variable.
```lua
function Vector3f:lookat(other)
    -- direction vector -> magnitude / sqrt(dot(v, v)
    return (other - self):length():normalized()
end
```

### `Vector3f:dot(vec2)`

Returns the dot product of two vectors. Order does not matter as this returns a float. The dot product is equivalen to the following expression 
```
l.x*r.x + l.y*r.y + l.z*r.z
```
The dot product is intended for determining the angle between two direction unit vectors (normalized) along a single axis. 
The value returned will be between -1.0 and 1.0. This can be used to calculate to also calculate the shortest distance of rotation by taking the sign.
If the axis is not known you can use the cross product, but very likely you will only need to consider yaw rotations.

```lua
local function clamp(x, min, max)
  return x < min and min or (x > max and max or x)
end

local pawn_forward = pawn.RootComponent:GetForwardVector()
local target_dir = (target:K2_GetActorLocation() - pawn:K2_GetActorLocation()):normalized()
local speed = 2.0

local dot = pawn_forward:dot(target_dir)
local angle = math.rad(math.acos(clamp(dot, -1.0, 1.0)))
-- snap to target if the angle is short
if (angle > 0 and angle < 45) or angle > -45 then speed = 12.0
pawn:K2_SetActorRotation(pawn:K2_GetActorRotation():lerp(target_dir, speed))
```

## `Vector3f`, `Vector3d`, `Vector4f`, `Vector4d`


### `Vector3f:cross(vec2)`

Returns the cross product of two vectors. Order matters. Very useful for gamedev

### `Vector3f.lerp(v1, v2, t)`

Performs LERP (linear interpolation) on two vectors by the factor of `t`. Order matters


### `Vector3f:reflect(vec2)`



### `Vector3f:refract(vec2, eta)`



### `Vector2f`, `Vector2d`,

## `Vector2f:to_vec3()`
## `Vector2f:to_vec4()`



**Unused**
UEVR_Vector4f, UEVR_Vector3d, UEVR_Rotatord, Vector4d

### Rotator Gotchas

Note that while rotations are internally calculated using [Quaternions](Quaternion.md) the engine already converts and caches Scene component rotations as `Rotators` for easy access, this is not a UEVR specific feature. The one notable exception to this is with `Transform` structs which always have a [Quaternion](Quaternion.md) for Rotation.

While you should always stick to the GLM bindings, if for some reason you are trying to handle things as tables or by accessing components directly you may occasionally run into cases where capitalization is inconsistent, e.g. "roll" instead of "Roll". If you look in the UEVR GUI or an SDK dump you will see the actual naming, but because rotators are recieved as `Vector3f` objects you will never actually be able to test for this result. However when submitting a table this will be directly converted to the expected struct so case does need to match. While this is rare, its impossible to handle automatically with tables.
 
Another important thing to consider is that UE uses degrees for rotation but glm and UEVR typically use radians. You can use math.deg() or math.rad() to convert as needed. There is no specific handling in UEVR for rotations looping around so consider it when lerping


## Functions

### UEVR_MotionControllerState:set_position_offset

This function can take rotation input as  `UEVR_Vector3f`, or `Vector3f`/`Vector3d`.

### `UEVR_Vector3f.new()`

Constructs a new `UEVR_Vector3f`. Do not directly set the components, instead create the vecernion and then set them individually if needed. You rarely need to do so as the main usage of `UEVR_Quaternion` is to receive VR data and it wil automatically be created with 0 assigned to each component.

### `UEVR_Vector3f:as_full_binding()`

Casts the struct to a `Vector3f` GLM variant. You should always cast for any non-trivial work as this is the only function available with UEVR variants. 
ex:
`local right_controller_rotation = UEVR_Vector3f.new()
 params.vr.get_pose(1, right_controller_position, right_controller_rotation)
 local r_vec = right_controller_rotation:as_full_binding()
 `

