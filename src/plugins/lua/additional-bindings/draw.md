# draw

A subset of [ImGui](imgui.md) accessed through a separate table `draw`. Invokable in the same conditions as `imgui` methods, i.e. within `uevr.sdk.callbacks.on_frame`, `uevr.sdk.callbacks.on_draw_ui` and within [Script Panels](../lua.md). Draw calls are added to current window's background draw list but do not modify the ImGui cursor position nor interact with any logical components of ImGui.

Note that as with ImGui, draw API is compatible with VR but is not well suited for in-game overlays, especially when projecting 3D world positions to the 2D screen. ImGui and draw API calls are projected in VR on the same plane as the UEVR UI and can be adjusted accordingly, but all scaling from the desktop screen is handled by UEVR.

Basic example of a draw API function projecting a world position to screen
```lua
local fill_color = 0xB1A3D9D8
local function scale_by_distance(world_location, min_radius, max_radius)
  -- use GetFocalLocation for quick camera world location 
  local distance = (world_location - player_controller:GetFocalLocation()):length()
  local UE_WORLD_MAX = 2097152.0
  -- normalize distance to 1.0
  local t = math.min(distance / UE_WORLD_MAX, 1.0)
  -- lerp specified radius scale by normalized distance
  local radius = max_radius + (min_radius - max_radius) * t

  return radius
end
end

local function get_screen_position(world_location)
  local screen_location = {}
  player_controller:ProjectWorldLocationToScreen(world_location, screen_location, true)
  return screen_location.result
end

local draw_calls = {}

local function object_draw_call(object)
  if UEVR_UObjectHook.exists(object) then
    local resolution = imgui.get_display_size()
    -- get location assuming object is either an actor or component
    local world_location = object.K2_GetActorLocation and object:K2_GetActorLocation() or object:K2_GetComponentLocation()
    -- project the world location to screen
    local screen_position = get_screen_position(world_location)
    -- check that the object is actually on screen (could limit to portion of the screen easily)
    if (screen_position.x < resolution.x and screen_position.x > 0) and
       (screen_position.y < resolution.y and screen_position.y > 0) then
      -- scale the size of the output circle
      local radius = scale_by_distance(world_location, 4.0, 32.0)
      -- use the radius to simplify the circle and reduce draw call complexity
      local segments = math.floor(radius)
      draw.filled_circle(screen_position.x, screen_position.y, radius, fill_color, segments)
      end
  end
end

-- Call this from somewhere else, e.g. a script panel or a window
local function add_object_draw_calls(object_list)
  for i, object in ipairs(object_list) do
    -- store the draw function as a lua object with the argument already provided
    local fn = object_draw_call(object)
    if fn then
      table.insert(draw_calls, fn)
    end
  end
end      



uevr.sdk.callbacks.on_frame(function()
  imgui.set_next_window_size(imgui.get_display_size())
  imgui.set_next_window_pos(Vector2f.new(0, 0))
  -- make a window with flags to hide the background, prevent focus, and ignore inputs
  imgui.begin_window("###Canvas", true, 209599)
  for i, fn in ipairs(draw_calls) do
    if type(fn) == "function" then
      fn()
    end
  end


end)

```
See [Lobotomy's script repository](github.com/lobotomy-x) for many more advanced examples of draw API.

# Methods

draw.text(text, x, y, color)
draw.filled_rect(x, y, w, h, color)
draw.outline_rect(x, y, w, h, color)
draw.line(x1, y1, x2, y2, color)
draw.outline_circle(x, y, radius, color, num_segments)
draw.filled_circle(x, y, radius, color, num_segments)
draw.outline_quad(x1, y1, x2, y2, x3, y3, x4, y4, color)
draw.filled_quad(x1, y1, x2, y2, x3, y3, x4, y4, color)
