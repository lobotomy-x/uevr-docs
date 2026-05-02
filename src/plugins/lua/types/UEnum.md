# UEVR_UEnum

## UEnum

UEnums in Unreal engine are uint8 enums with UObject reflection supported, allowing developers to retrieve names of fields at runtime. With UEVR you will use UEnums when calling functions or setting properties by passing an integer corresponding to the correct index.

Some script authors prefer to define tables with enum values taken from an SDK dump. This allows for calling enums by `enum_name.field_name` 
```lua
local EAttachmentRule = {
    KeepRelative                             = 0,
    KeepWorld                                = 1,
    SnapToTarget                             = 2,
    EAttachmentRule_MAX                      = 3,
}

comp:K2_AttachToComponent(actor, socket, EAttachmentRule.KeepRelative, EAttachmentRule.KeepWorld, EAttachmentRule.SnapToTarget, true)
comp:K2_AttachToComponent(actor, socket, 0, 1, 2, true)
```

### Obtaining UEnum names dynamically

```lua
local NodeHelper = uevr.api:find_uobject("Class /Script/Engine.KismetNodeHelperLibrary"):get_first_object_matching(true)

function dump_enums()
    local enums = {}
    for i, v in ipairs(api:find_uobject("Class /Script/CoreUObject.Enum"):get_objects_matching(false)) do
        local enum_name = v:get_short_name()
        local package_name = v:get_outer():get_short_name()
        enums[package_name] = enums[package_name] or {}
        enums[package_name][enum_name] = {}
        for j = 0, 255 do
            local valid_value =  NodeHelper:GetValidValue(v, j)
            local f =  NodeHelper:GetEnumeratorName(v, valid_value)
            f = f:to_string()
            local idx = f:find("::")
            f = f:sub((idx and idx + 2) or 1)
            if f ~= "None" then
                enums[package_name][enum_name][f] = valid_value
            else
                break
            end
        end
    end
    json.dump_file("Enums.json", enums, 4)
end
```
example output
```json
    "/Script/HeadMountedDisplay": {
        "EHMDTrackingOrigin": {
            "EHMDTrackingOrigin_MAX": 3,
            "Eye": 1,
            "Floor": 0,
            "Stage": 2
        },
        "EHMDWornState": {
            "EHMDWornState_MAX": 3,
            "NotWorn": 2,
            "Unknown": 0,
            "Worn": 1
        },

```

