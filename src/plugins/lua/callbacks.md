# uevr.sdk.callbacks 
These functions all take an inner function as a parameter and may pass a specific set of variables to that function. Each use of a callback function the inner function to a global array to be batch executed. Callbacks can occur multiple times in one script and can be called from within an outer function defining conditional logic. Be careful when calling `require` to import script modules as doing so will both import the script and run it. Since UEVR shares a singular state you should be able to retrieve modules from the global table as long as they return the functions you need. If you prefer to require then scripts that are meant to be required should avoid using sdk callbacks in their main scope to prevent doubling calls, especially with script panels or imgui calls. You can avoid the issue by using a main script with centralized callbacks or you can instead embed the callbacks within your functions to prevent immediate usage. When done this way you can even declare local variables within a local function and create an upvalue for the callback function.
ex:
```lua
function post_tick(interval, fn)
    local pt = 0
    uevr.sdk.callbacks.on_post_engine_tick(function(engine, delta)
       pt = pt + 1
       if pt == interval or pt % interval == 0 then
            if type(fn) == "function" then
                fn(engine, delta)
             end
        end
        if pt > math.max(2000, interval) then pt = 0 end
    end)
end
-- Prints the engine address every 50 ticks
post_tick(50, function(engine, delta)
    print("50 ticks")
end)

-- If pt was defined outside of post_tick this would make both calls fire at 20 ticks
post_tick(20, function(engine, delta)
    print("20 ticks")
end)

```

## Functions

### `uevr.sdk.callbacks.on_xinput_get_state(fn)`
Registers a callback to be called when XInput state is requested.

Prototype: `function(retval: uint32, user_index: uint32, state: XINPUT_STATE*)`

[XINPUT_STATE](thirdparty/XINPUT_STATE.md)

### `uevr.sdk.callbacks.on_xinput_set_state(fn)`

Registers a callback to be called when XInput state is set.

Prototype: `function(retval: uint32, user_index: uint32, state: XINPUT_VIBRATION*)`

[XINPUT_VIBRATION](thirdparty/XINPUT_VIBRATION.md)

### `uevr.sdk.callbacks.on_pre_engine_tick(fn)`

Registers a callback to be called before the engine tick.

Prototype: `function(engine: void*, delta_time: float)`

### `uevr.sdk.callbacks.on_post_engine_tick(fn)`

Registers a callback to be called after the engine tick.

Prototype: `function(engine: void*, delta_time: float)`

### `uevr.sdk.callbacks.on_pre_slate_draw_window_render_thread(fn)`

Registers a callback to be called before the slate window is drawn on the render thread.

Prototype: `function(renderer: UEVR_FSlateRHIRendererHandle, viewport_info: UEVR_FViewportInfoHandle)`

### `uevr.sdk.callbacks.on_post_slate_draw_window_render_thread(fn)`

Registers a callback to be called after the slate window is drawn on the render thread.

Prototype: `function(renderer: UEVR_FSlateRHIRendererHandle, viewport_info: UEVR_FViewportInfoHandle)`

### `uevr.sdk.callbacks.on_pre_calculate_stereo_view_offset(fn)`

Registers a callback to be called before VR transformations are applied to the view.

Prototype: `function(device: UEVR_StereoRenderingDeviceHandle, view_index: int, world_to_meters: float, position: Vector3f* or Vector3d*, rotation: Quaternionf* or Quaterniond*, is_double: bool)`

### `uevr.sdk.callbacks.on_post_calculate_stereo_view_offset(fn)`

Registers a callback to be called after VR transformations are applied to the view.

Prototype: `function(device: UEVR_StereoRenderingDeviceHandle, view_index: int, world_to_meters: float, position: Vector3f* or Vector3d*, rotation: Quaternionf* or Quaterniond*, is_double: bool)`

### `uevr.sdk.callbacks.on_pre_viewport_client_draw(fn)`  

Registers a callback to be called before the viewport is drawn on the game thread.

Prototype: `function(viewport: UEVR_FViewportClientHandle, viewport: UEVR_FViewportHandle canvas: UEVR_FCanvasHandle)`

### `uevr.sdk.callbacks.on_post_viewport_client_draw(fn)`

Registers a callback to be called after the viewport is drawn on the game thread.

Prototype: `function(viewport: UEVR_FViewportClientHandle, viewport: UEVR_FViewportHandle canvas: UEVR_FCanvasHandle)`

### `uevr.sdk.callbacks.on_frame(fn)`

Registers a callback to be called every frame. For ImGui functionality.  
Prototype: `function()`

### `uevr.sdk.callbacks.on_draw_ui(fn)`

Registers a callback to be called when the UI is drawn. For ImGui functionality that should be constrained to the UEVR window.

### `uevr.sdk.callbacks.on_script_reset(fn)`

Registers a callback to be called before the Lua script is reset. Script cleanup should be performed here.

### `uevr.sdk.callbacks.on_lua_event(fn)`

Registers a callback that can be triggered when a C plugin dispatches an event for Lua.

Prototype: `function(event_name: string, event_data: string)`
