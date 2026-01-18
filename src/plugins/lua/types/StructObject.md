# StructObject

StructObject is a custom UEVR Lua type that represents a struct in Unreal Engine. It's for non-UObject based structs which support reflection. However it is almost always better to use a table to represent ScriptStructs.

## Functions

### `StructObject.new(struct: UEVR_UStruct*)`

Constructs a new StructObject from a [UEVR_UStruct](UStruct.md) pointer.
ex:
```lua
local hitresult_c = uevr.api:find_uobject("ScriptStruct /Script/Engine.HitResult")
local hit = StructObject.new(hitresult_c)
```

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
This allows you to handle most data directly in lua and only pass it to the game when calling functions or setting properties, giving you more control over your data and reducing the need for synchronization. There are also a few a known games, usually on custom engine versions, where StructObject creation does not work but tables are fine. In fact even in games where ScriptStruct creation works for some cases there are others such as PostProcessSettings that the API cannot retrieve, even when a memory address is supplied, but using a table works. 

Table to struct conversion should work for almost all cases taking a StructProperty which includes setting properties on objects and calling functions with StructProperty parameters.

## Common Table to ScriptStruct Uses

**LinearColor**:
```lua
   local sphere = api:add_component_by_class(pawn, "Class /Script/Engine.DrawSphereComponent", false)
   sphere:SetShapeColor({R = 255, G = 0, B = 0, A = 255})

  
   
    -- take an imgui Vector4f e.g. an output from imgui.color_edit4 and create a LinearColor struct
   local function vec4_to_linear_color(color)
       return {
           R = math.floor(color.x * 255.0),
           G = math.floor(color.y * 255.0),
           B = math.floor(color.z * 255.0),
           A = math.floor(color.w * 255.0),
       }
   end

```
**FKey**
```lua
-- rest of key names omitted here
fkey = {"AnyKey","MouseX","MouseY","Mouse2D","MouseScrollUp","MouseScrollDown","MouseWheelAxis",
    "LeftMouseButton","RightMouseButton","MiddleMouseButton","ThumbMouseButton","ThumbMouseButton2", ...}


-- Poll all keys once per tick (can only be read in engine or on_frame callbacks, imgui also steals gamepad inputs)
local current_pressed_keys = {}   
uevr.sdk.callbacks.on_pre_engine_tick(function(engine, delta)
    for i, key in ipairs(fkey) do
        if pc:IsInputKeyDown({KeyName = key}) then
            current_pressed_keys[key] = true
        else
            current_pressed_keys[key] = false
        end
    end
end)

```

Note that you do not need to fill out all properties of a struct when supplying a table although certain properties may be required by the engine. 
ex:
**PostProcessSetitngs**
```lua
pc = api:get_player_controller(0)
local vt = pc:GetViewTarget()
camera_component_c = camera_component_c or api:find_uobject("Class /Script/Engine.CameraComponent")
local CameraComp = vt.GetComponentByClass and vt:GetComponentByClass(camera_component_c)
if UEVR_UObjectHook.exists(CameraComp) and CameraComp.bIsActive then
   CameraComp.PostProcessSettings = {
       bOverride_GrainJitter = 1,
       GrainJitter = 1,
       bOverride_GrainIntensity = 1,
       GrainIntensity = 1,
   }
end

```

## Transformations
All relevant struct types for transformations support table-based creation. However in the case of Vectors and Rotators you should opt for the corresponding [Vector](Vector.md). 

FHitResult is a common ScriptStruct object present in many function signatures, but its almost always an out parameter meaning you don't even need to fill in the properties in most cases.
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

## How to make structs yourself

You will not be able to view the definitions for many ScriptStructs in the UEVR GUI as you can with UObject Classes so you will be limited to the cases where ScriptStructs are properties of an object or function results if using the process event hook viewer. Generally you should be able to find the necessary data by dumping the SDK with UEVR or using an alternative dumper.

