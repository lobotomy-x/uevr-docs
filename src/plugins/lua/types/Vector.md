# Vectors

## What are Vectors?

In mathematics vectors are objects with both magnitude and direction that can have any number of dimensions. Vectors in Unreal Engine function as geometric vectors but are also concrete data structs holding 2, 3, or 4 values typically representing locations. The most common form, `Vector` contains the `X`, `Y`, and `Z` components representing positions. Unless otherwise specified, `Vector` will refer to this version. Additional types, `Vector2D` and `Vector4`, are used for screen-space transformations and material functions but are less likely to be encountered.

Vectors in UE4 use `float` values while in UE5 they use `double` precision floats. UEVR bindings have `float` and `double` variants denoted by their suffix. For the sake of simplicity `f` will be used for any examples. If you are ever in doubt use the `float` version. You will lose some precision as opposed to throwing an error. As a sidenote, lua numbers are `double` precision by default but will be converted to the appropriate type when calling game functions. However, tables have higher overhead and less functionality so they should generally be avoided.

*Note that while UEVR and UE itself streamline most of the difficult math, its useful to have some baseline understanding of linear algebra concepts. Its less important that you know how to compute something as it is to know when or why you would want to. The examples here will show some basic usage patterns but if you came in knowing nothing about linear algebra you will probably still know very little. Learning through trial and error is not a bad idea and will certainly be more enjoyable than sitting in a classroom.*


# UEVR Vectors

UEVR offers an assortment of Vector types with various use cases. Versions prefixed with UEVR are custom struct types used for getting VR transformation data. The other types are bindings for the [GLM math library](https://github.com/g-truc/glm/). These GLM versions offer more functionality and should be used as much as possible.

## Vector3
**`Vector3f`** / **`Vector3d`**

These are the most important variants as they provide automatic bindings to Unreal Engine's `Vector` and `Rotator` structs and can be used instead of [StructObject](StructObject.md) or tables when calling functions or setting properties. This is also handled automatically when getting values. Any location, rotation, or scale values can use `Vector3f`

### Components
Internally `Vector3f` has `x`, `y`, and `z` components but to make things easier you can access each component with the following aliases
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
```

## Constructor Functions

### `Vector3f.new(x, y, z)`

Constructs a new `Vector3f`. You must provide some value for each component. Note that while components can be accessed similar to table keys, `Vector3f` is not a lua table so the values must be entered directly.

ex:
```lua
local vec = Vector3f.new(1.0, 0.0, 0.0)
```

### `Vector3f:clone()`

Copies data into a new `Vector3f` variable. Can use the colon operator to refer to `self`.

ex:
```lua
local vec = Vector3f.new(1.0, 0.0, 0.0)
local vec2 = vec:clone()
-- or 
local vec2 = Vector3f.clone(vec)
```


### `Vector3f:set(x, y, z)`

Takes an existing `Vector3f` and sets the values directly. Similar usage as `new` but should use the colon operator. 
 
ex:
```lua
local vec = Vector3f.new(1.0, 0.0, 0.0)
vec:set(2.0, 0.5, 1.0)
-- or
Vector3f.set(vec, 1.0, 1.0, 1.0)
```

Creating a new object does incur some overhead so it is preferable to reuse your objects when possible, especially inside of loops. Technically it is still faster to set each component individually but this is a negligible difference. 

There is no particular harm in storing `Vector` objects in a table but its worth noting that they will not function like tables when using [json](additional-bindings/json.md) to dump or load.
If you do need to pass data from a table frequently, e.g. if loading constants from a json, you could setup a shortcut with `table.unpack` for numerically keyed tables. This is probably not very efficient but it should not a be a problem if used sparingly.
ex:
```lua
function v3f(t)
    assert(t[1] ~= nil, "Table must be an array")
    return Vector3f.new(table.unpack(t))
end


local v = v3f{1.0, 0.3, 0.6}
```

This may be a bit more useful as it takes an optional vector to allow reusing it.
```lua
function v3f(t, v)
    return v and v:set(table.unpack(t)) or Vector3f.new(table.unpack(t))
end

v = v3({20, 20, 20}, v)

```

Here's another alternative constructor. This one might actually be useful but is still trading speed for ease of use.
```lua
-- pass any number of args
function vector(...)
    -- gets the size of the variadic args
    local vector_size = select('#', ...)
    local vector_alias = vector_size > 3 and Vector4f or vector_size > 2 and Vector3f or Vector2f
    return vector_alias.new(table.unpack{...})
end

local v2 = vector(1.0, 2.0)
print(tostring(v))
print(v.x)
print(v.y)

local v4 = vector(1.0, 0.0, 1.0, 2.0)


```

As a small note on speed, consider that lua does actually compile before it runs. If you have a lot of constructors out in the main body or in a callback you are adding some overhead during script load. Well optimized lua scripts should only add a few seconds to startup time. Your scripts could still run fast enough but if the startup is over 10 seconds longer than uevr on its own there is a serious issue and debugging will be hell

# MetaMethods

### `Vector2f`, `Vector2d`, `Vector3f`, `Vector3d`, `Vector4f`, `Vector4d`

### `Vector3f.__add(lhs_vec, rhs_vec)`

Adds two `Vector3f` values together. Note that order of operations matters with `Vector` mathematics meaning `lhs + rhs ~= rhs + lhs`.  

This can be performed with the `+` operator.

ex:
`local vec_sum = v1 + v2
 `


### `Vector3f.__sub(lhs_vec, rhs_vec)`

Subtracts a `Vector3f` from another `Vector3f`. Note that order of operations matters with Quaternion and Vector mathematics meaning `lhs - rhs ~= rhs - lhs` 

This can be performed with the `-` operator.
ex:
```lua
local vec_diff = v1 - v2
 vec_sum:normalize()
```

### `Vector3f.__mul(lhs_vec, scalar)`

Multiplies a `Vector3f` value by a scalar. 

This can be performed with the `*` operator.
ex:
`local vec_mul = v1 * scalar`

## Functions

### `Vector2f`, `Vector2d`, `Vector3f`, `Vector3d`, `Vector4f`, `Vector4d`


### `Vector3f:length()`

Returns the `float` or `double` magnitude calculated as the sum of its squared components. If magnitude is equal to 1.0 then you can assume this is a unit vector representing a rotation.

This can be used to get the distance between two positions.
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

Normalizes components so that length is 1.0 making it a unit vector representing a rotation. This modifies the original variable.

### `Vector3f:normalized()`

Normalizes components so that length is 1.0 making it a unit vector. This returns a new variable


### `Vector3f:dot(vec2)`

Returns the dot product of two vectors


## `Vector3f`, `Vector3d`, `Vector4f`, `Vector4d`


### `Vector3f:cross(vec2)`

Returns the cross product of two vectors

### `Vector3f.lerp(v1, v2, t)`

Performs LERP (linear interpolation) on two vectors by the factor of `t`.


### `Vector3f:reflect(vec2)`


### `Vector3f:refract(vec2, eta)`



### `Vector2f`, `Vector2d`,

## `Vector2f:to_vec3()`
## `Vector2f:to_vec4()`



`Vector4f`

Used for ImGui colors. No UE bindings but can be useful for calculations

## Vector2
**`Vector2f`**  
The most common `Vector2` variant. [ImGui](additional-bindings/imgui.md) size and position values are expected as `Vector2f`. 

**`Vector2d`** 
Note that the d here is for `double` unlike `Vector2D` in Unreal that means "2-Dimensions". This version is not used anywhere at this time.

**`UEVR_Vector2f`**
Used for reading analogue stick data from VR controllers.

**UEVR_Vector3f**
Position data

**UEVR_Rotatorf**
Some rotator data


USELESS:
UEVR_Vector4f, UEVR_Vector3d, UEVR_Rotatord




`Vector4d`

Useless



### Rotator Gotchas

Note that while rotations are internally calculated using [Quaternions](Quaternion.md) the engine already converts and caches `Rotators` into  for easy access, this is not a UEVR specific feature. The one notable exception to this is with `Transform` structs which always have a [Quaternion](Quaternion.md) for Rotation. You can use `Kismet Math Library` to handle this if need be. On the other hand when accessing the `RelativeRotation` or `RelativeTransform.Rotation` property of `SceneComponent` objects you will get a 3 component `Rotator`.

While you should always stick to the GLM bindings, if for some reason you are trying to handle things as tables or by accessing components directly you may occasionally run into cases where capitalization is inconsistent, e.g. "roll" instead of "Roll". If you look in the UEVR GUI or an SDK dump you will see the actual naming, but because rotators are recieved as `Vector3f` objects you will never actually be able to test for this result. However when submitting a table this will be directly converted to the expected struct so case does need to match. While this is rare, its impossible to handle automatically with tables. 

Another important thing to consider is that UE uses degress for rotation but glm and UEVR typically use radians. You can use math.deg() or math.rad() to convert as needed. There is no specific handling in UEVR for rotations looping around so consider it when lerping



`ScriptStruct /Script/CoreUObject.Vector`
There is zero reason to ever actually fetch this uobject. Don't do it

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

