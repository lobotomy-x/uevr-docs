# draw

A subset of [ImGui](imgui.md) accessed through a separate table `draw`. Invokable in the same conditions as `imgui` methods, i.e. within `uevr.sdk.callbacks.on_frame`, `uevr.sdk.callbacks.on_draw_ui` and within [Script Panels](../lua.md). Draw calls are added to current window's background draw list but do not modify the ImGui cursor position nor interact with any logical components of ImGui.

Note that as with ImGui, draw API is compatible with VR but is not well suited for in-game overlays, especially when projecting 3D world positions to the 2D screen. ImGui and draw API calls are projected in VR on the same plane as the UEVR UI and can be adjusted accordingly, but all scaling from the desktop screen is handled by UEVR.
```lua
      imgui.draw_list_path_clear()
      for i,v in ipairs(path) do
            screen_location = v:world_to_screen()
            -- otherwise it will connect to the upper left corner
            if screen_location.x + screen_location.y == 0 then
                imgui.draw_list_path_clear()
            else
                imgui.draw_list_path_line_to(screen_location)
            end
        end
        imgui.draw_list_path_stroke(color, false, 12)
```

Complete example of using draw API with UEVR_UObjectHook to project a circle on screen space for every skeletal mesh
```lua
  -- draw api takes U32 hex integers for colors 
  local function VecToU32(vec)
      local r = math.floor(math.max(0, math.min(255, vec.x * 255 + 0.5)))
      local g = math.floor(math.max(0, math.min(255, vec.y * 255 + 0.5)))
      local b = math.floor(math.max(0, math.min(255, vec.z * 255 + 0.5)))
      local a = math.floor(math.max(0, math.min(255, vec.w * 255 + 0.5)))
      -- ImGui expects 0xAABBGGRR
      return (a << 24) | (b << 16) | (g << 8) | r
  end


  pc = api:get_player_controller(0)
  local fill_color = 0xB1A3D9D8
  -- some constants
  local UE_WORLD_MAX = 2097152.0
  local MIN_DISTANCE = 850.0
  local RADIUS_SCALE = 0.6
  local MAX_DISTANCE = 6000

  local function scale_by_distance(world_location, distance, min_radius, max_radius)
    if distance > MAX_DISTANCE then MAX_DISTANCE = distance * 1.1 end
    local d = math.min(math.max(distance, MIN_DISTANCE), MAX_DISTANCE)
    -- log scaling for more of a curve
    local t = (math.log(d) - math.log(MIN_DISTANCE)) / (math.log(MAX_DISTANCE) - math.log(MIN_DISTANCE))
    t = math.min(math.max(t, 0.0), 1.0)
    -- lerp specified radius scale by log scaled distance
    local radius = max_radius + (min_radius - max_radius) * t
  return radius
end

local function get_screen_position(world_location)
  local screen_location = {}
  GameplayStatics:ProjectWorldToScreen(pc, world_location, screen_location, true)
  return Vector2f.new(screen_location.result.X, screen_location.result.Y)
end


local resolution = imgui.get_display_size()

local function object_draw_call(object)
  if UEVR_UObjectHook.exists(object) then
    -- get location from the actor
    local world_location = object.K2_GetComponentLocation and object:K2_GetComponentLocation() or object:get_outer():K2_GetActorLocation()
    -- raise the height a bit since it will give us the location of the root bone which is between their feet
    world_location.z = world_location.z + 50
    -- project the world location to screen
    local screen_position = get_screen_position(world_location)
    if not screen_position then return end
    -- check that the object is actually on screen (could limit to portion of the screen easily)
    -- if you're doing anything more complex, e.g. checking for mouse over you should have a function to check if the point is in a rectangle
    if screen_position and screen_position.x and (screen_position.x < resolution.x and screen_position.x > 0) and
     screen_position.y and (screen_position.y < resolution.y and screen_position.y > 0) then

      -- scale the size of the output circle using glm Vec3 bindings
      -- GetFocalLocation gives the location of the camera itself, not the pawn or player controller
      local distance = (world_location - pc:GetFocalLocation()):length()
      local radius = scale_by_distance(world_location, distance, 4.0, 48.0) * RADIUS_SCALE

      -- use the radius scaling to simplify the circle and reduce draw call complexity, math.floor to get an integer
      local segments = math.floor(radius * 0.25)
      draw.filled_circle(screen_position.x, screen_position.y, radius, fill_color, segments)
      -- add a slight outline
      draw.outline_circle(screen_position.x, screen_position.y, radius * 1.01, VecToU32(Vector4f.new(1.0, 1.0, 1.0, 1.0)), segments)
      -- add text to the draw list
      draw.text("Distance: "..tostring(math.floor(distance)), screen_position.x, math.max(0,screen_position.y - radius * 2), fill_color)
      draw.text("Radius: "..tostring(math.floor(radius)), screen_position.x, math.max(0, screen_position.y - radius), fill_color)
      end
  end
end
  
  
  local mesh_comps
  local skinnedmesh = api:find_uobject("Class /Script/Engine.SkinnedMeshComponent")
  
  
  uevr.sdk.callbacks.on_frame(function()
      mesh_comps = mesh_comps or skinnedmesh:get_objects_matching(false)
      resolution = resolution or imgui.get_display_size()
      -- make a full sized window with flags to hide the background, prevent focus, and ignore inputs
      imgui.set_next_window_size(resolution)
      imgui.set_next_window_pos(Vector2f.new(0, 0))
      imgui.begin_window("###Canvas", true, 209599)
      if mesh_comps then
          for i, object in ipairs(mesh_comps) do
              object_draw_call(object)
          end
      end
      imgui.end_window()
  end)
```
Spinning circle
```lua
local function lerp(x0, x1, t)
    return (1.0 - t) * x0 + t * x1
end

local function interval(t0, t1, tween_func)
    return function(t)
        --return t < t0 and 0.0 or t > t1 and 1.0 or tween_func((t - t0) / (t1 - t0))
        if t < t0 then
            return 0.0
        elseif t > t1 then
            return 1.0
        end

        return tween_func((t - t0) / (t1 - t0))
    end
end

local function sawtooth(x, t)
    return math.fmod(x * t, 1.0)
end

local function cubic_bezier(t, p0, p1, p2, p3)
    local u = 1.0 - t
    return p0 * u*u*u + p1 * 3.0 * u*u*t + p2 * 3.0 * u*t*t + p3 * t*t*t
end

local function stroke_head_tween(d, t)
    t = sawtooth(d, t)
    return interval(0.0, 0.5, function(x) return cubic_bezier(x, 0.2, 0.0, 0.4, 1.0) end)(t)
end

local function stroke_tail_tween(d, t)
    t = sawtooth(d, t)
    return interval(0.5, 1.0, function(x) return cubic_bezier(x, 0.2, 0.0, 0.4, 1.0) end)(t)
end

local function step_tween(x, t)
    return math.floor(lerp(0.0, x, t))
end

-- this in turn is taken from reframework
-- https://github.com/ocornut/imgui/issues/1901
function draw.spinner(center, radius, color, thickness)
    local rect = {
        imgui.get_cursor_pos(),
        imgui.get_cursor_pos() + Vector2f.new(radius * 2, radius * 2) -- todo: frame padding
    }

    imgui.item_size(rect[1], rect[2])
    if not imgui.item_add(rect[1], rect[2], "circle") then
        --print("oh no")
        --return
    end

    local period = 5.0
    local t = math.fmod(os.clock(), period) / period

    imgui.draw_list_path_clear()

    local num_segments = 24

    local num_detents = 5
    local skip_detents = 3

    local head_value = stroke_head_tween(num_detents, t);
    local tail_value = stroke_tail_tween(num_detents, t);
    local step_value = step_tween(num_detents, t);
    local rotation_value = sawtooth(num_detents, t);

    local min_arc =  30.0 / 360.0 * 2.0 * math.pi
    local max_arc = 270.0 / 360.0 * 2.0 * math.pi
    local step_offset = skip_detents * 2.0 * math.pi / num_detents
    local rotation_compensation = math.fmod(4.0*math.pi - step_offset - max_arc, 2.0 * math.pi);

    local start_angle = -math.pi * 2.0
    local a_min = start_angle + tail_value * max_arc + rotation_value * rotation_compensation - step_value * step_offset;
    local a_max = a_min + (head_value - tail_value) * max_arc + min_arc;

    for i = 0, num_segments - 1 do
        local a = a_min + (i / num_segments) * (a_max - a_min)
        local x = center.x + math.cos(a) * radius
        local y = center.y + math.sin(a) * radius
        imgui.draw_list_path_line_to(Vector2f.new(x, y))
    end

    imgui.draw_list_path_stroke(color, false, thickness)
end
```
See [Lobotomy's script repository](https://github.com/lobotomy-x/Unreal-Lua-Library/blob/main/Docs/UEVR/DrawAPI.md) for some advanced examples of draw API.

# Methods

### draw.text(text, x, y, color)
Uses the default font and scale for UEVR. You should probably instead use a window with no background and set cursor screen pos.

## draw.line(x1, y1, x2, y2, color)
No way to set thickness, better to use a rect or use `path_line_to.` The only advantage over `path_line_to` is that each line segment here can be a different color/opacity while only one color can be used per frame with path stroke.

### draw.filled_rect(x, y, w, h, color) / draw.outline_rect(x, y, w, h, color) 
Use this if you don't need to apply rotation. Winding direction is handled automatically. x and y define the top left origin point.

## draw.filled_circle(x, y, radius, color, num_segments) / draw.outline_circle(x, y, radius, color, num_segments)
num_segments is optional and defaults to 12

## draw.filled_quad(x1, y1, x2, y2, x3, y3, x4, y4, color) / draw.outline_quad(x1, y1, x2, y2, x3, y3, x4, y4, color)
Use quads if you want to apply rotation. Quad corners must be submitted in clockwise winding order or there will be visible aliasing. 
