# Quaternion

## What are Quaternions?

Quaternions represent rotations in 4 axes. All rotations in Unreal are internally calculated using quaternions as they avoid issues inherent to Euler angles. Look up "Gimbal Lock" for more information.

Quaternions in UEVR are represented in three different ways, `UEVR_Quaternionf` represents rotations managed by the UEVR API, e.g. for motion controller states, while `Quaternionf`, `Quaterniond` provide bindings for GLM quaternions depending on game engine version. Unreal Engine 5 uses `double` precision floating point numbers so you must use `Quaterniond`  or you will lose precision. UE4 uses `float` numbers so you use `Quaternionf` and `UEVR_Quaternionf`. For the sake of simplicity `f` will be used for the remaining text. If you were to only use one type it needs to be the `float` variant as this will only lose precision while `double` versions can cause crashes.

## Components
All quaternions are made up of a Vector3 with `x`, `y`, `z` and a scalar `w`
### `UEVR_Quaternionf`
- `x`, `y`, `z`,  `w`
  
### `Quaternionf`
- `x` or `X`, `y` or `Y`, `z` or `Z`, `w` or `W`

## Functions

### UEVR_MotionControllerState:set_rotation_offset

This function can take rotation input as `Vector4f`/`Vector3d` , `UEVR_Quaternionf`, or `Vector3f`/`Vector3d`. If a `Vector4` is provided it will be treated as a Quaternion. `Vector3` inputs are treated as Euler angle rotations. In all cases the internal calculation will use `UEVR_Quaternionf`.

### `UEVR_Quaternion.new()`

Constructs a new `UEVR_Quaternionf`. Do not directly set the components, instead create the quaternion and then set them individually if needed. You rarely need to do so as the main usage of `UEVR_Quaternion` is to receive VR data and it wil automatically be created with 0 assigned to each component.

### `UEVR_Quaternion:as_full_binding()`

Casts the struct to a `Quaternionf` GLM variant. You should always cast for any non-trivial work as this is the only function available with UEVR variants. 
ex:
`local right_controller_rotation = UEVR_Quaternionf.new()
 params.vr.get_pose(1, right_controller_position, right_controller_rotation)
 local r_quat = right_controller_rotation:as_full_binding()
 `

### `Quaternionf.new(x, y, z, w)`

Constructs a new `Quaternionf`. You do need to directly set the components.
ex:
`local quat = Quaternionf.new(1.0, 0.0, 0.0, 1.0)`

### `Quaternionf:length()`

Returns the magnitude calculated as the sum of its squared components.

### `Quaternionf:normalize()`

Normalizes components so that length is 1.0 making it a unit quaternion. Quaternions representing rotations should always be unit quaternions. This modifies the original variable.

### `Quaternionf:normalized()`

Normalizes components so that length is 1.0 making it a unit quaternion. This returns a new variable

### `Quaternionf:conjugate()`

Negates the vector (x, y, z) part of the quaternion by flipping the signs of x, y, z. Equivalent to `inverse` for unit quaternions.

### `Quaternionf:inverse()`

Inverts the rotation by giving the conjugate divided by the magnitude squared. Since a unit quaternion has a magnitude of 1 this is equivalent to the conjugate.

### `Quaternionf:dot(quat2)`

Returns the dot product of two quaternions.

### `Quaternionf.slerp(q1, q2, t)`

Performs SLERP (spherical linear interpolation) on two quaternions by the factor of `t`.

## Meta Functions

### `Quaternionf.__add(lhs_quat, rhs_quat)`

Adds two `Quaternionf` values together. Note that order of operations matters with Quaternion and Vector mathematics meaning `lhs + rhs ~= rhs + lhs` 

This can be performed with the `+` operator.
ex:
`local quat_sum = q1 + q2
 quat_sum:normalize()
`
Note that you should almost never need to sum two quaternions as that is not a valid way to work with rotations. If you do, make sure to normalize to make a valid rotation.

### `Quaternionf.__sub(lhs_quat, rhs_quat)`

Subtracts a `Quaternionf` from another `Quaternionf`. Note that order of operations matters with Quaternion and Vector mathematics meaning `lhs - rhs ~= rhs - lhs` 

This can be performed with the `-` operator.
ex:
`local quat_diff = q1 - q2
 quat_sum:normalize()
`
Note that you should almost never need to subtract two quaternions as that is not a valid way to work with rotations. If you do, make sure to normalize to make a valid rotation.


### `Quaternionf.__mul(lhs_quat, scalar)`

Multiplies a `Quaternionf` value by a scalar. 

This can be performed with the `*` operator.
ex:
`local quat_mul = q1 * scalar`
Note that while Quaternion multiplication is considered useful for rotation you cannot multiply two quaternions like this and will need to perform calculations by accessing the components or build a `Vector4` from the components and use KismetMathLibrary Quat functions.



