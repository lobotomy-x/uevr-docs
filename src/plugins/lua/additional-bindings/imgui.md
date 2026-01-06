# imgui

Bindings for ImGui. Can be used in the `uevr.sdk.callbacks.on_draw_ui` and `uevr.sdk.callbacks.on_frame` callbacks or within a [Script Panel](../lua.md).
Calling ImGui functions outside of these situations will likely cause a crash but it is safe to call game functions from ImGui. Calls made during `on_frame` without creating a new window manually will automatically draw to a new window labeled "Debug". 

Example:
```
local thing = 1
local things = { "hi", "hello", "howdy", "hola", "aloha" }

uevr.sdk.callbacks.on_draw_ui(function()
    if imgui.button("Whats up") then 
        thing = 1
    end

    local changed, new_thing = imgui.combo("Greeting", thing, things) 
    if changed then thing = new_thing end
end)
```

## Enums

### `imgui.TableSortSpecs`

### `imgui.TableColumnSortSpecs`

### `imgui.TableFlags`

### `imgui.ColumnFlags`

### `imgui.ImGuiKey`

### `imgui.ImGuiStyleVar`


### ID Methods
Correct usage of ImGui's ID system is crucial to success and can alleviate many common errors. IDs are automatically generated based on window context but this is generally not sufficient when multiple widgets of the same type are present and it prevents use of features like appending to windows. Context for any widget can manually be set by preceding the calls with `push_id` and following with `pop_id`. Many widgets with labels will use the provided label as their ID. In those cases it is possible to hide the label and still specify an ID without using `push_id` by prefixing your desired ID with `"###"`. The actual effect is that anything before the `"###"` is ignored by the ID system therefore allowing different display names without affecting ID.
ex:
```lua
-- this will create a window without any visible label
imgui.begin_window("###test_window")
imgui.text("test1")
imgui.end_window()

-- this will append to the above window. Since its called afterwards the title cannot be changed 
imgui.begin_window("TEST###test_window")
imgui.text("test2")
imgui.end_window()

-- this will create the window with the visible name TEST
imgui.begin_window("TEST###test_window2")
imgui.text("test3")
imgui.end_window()

-- this will append to the second window without changing the name 
imgui.begin_window("DIFFERENT NAME###test_window2")
imgui.text("test2")
imgui.end_window()


```

### `imgui.get_id(id)`
`id` - can be a `string`, `number`, or `userdata` e.g. a UObject.
Calculates a valid numeric ID from the provided input. While other lua types are not supported directly, any non-nil value can be passed with `tostring`, e.g `local mytable_id = imgui.get_id(tostring(mytable))`

### `imgui.push_id(id)`
Assign the ID or context for all imgui calls until `pop_id` is called

### `imgui.pop_id()`


## Methods
### `imgui.begin_window(name, open, flags)`
Creates a new window with the title of `name`.

`open` is a bool. Can be `nil`. If not `nil`, a close button will be shown in the top right of the window.

`flags` - ImGuiWindowFlags. Can be `nil` or 0. 

`begin_window` must have a corresponding `end_window` call.

Returns a bool. Returns `false` if the user wants to close the window. The return value is optional, 
ex:
```lua
imgui.begin_window("My Window")
imgui.text("text in a window")
imgui.end_window()
```
Windows can be appended to by calling `begin_window` again with the same label.

### `imgui.end_window()`
Ends the last `begin_window` call. Required.

### `imgui.begin_child_window(name, size, border, flags)`
`name` - string

`size` - Vector2f

`border` - bool

`flags` - ImGuiWindowFlags

Child windows are ideal for organizing widgets within a window or script panel. Child windows can also be appended to but only in the context of the same window.

### `imgui.end_child_window()`

### `imgui.begin_group()`
### `imgui.end_group()`

Organize a set of widgets together. You can but do not have to call `push_id` before calling this. Allows for `is_item_active`, `is_item_hovered` and `is_item_focused` to work on the entire group.

### `imgui.begin_rect()`
### `imgui.end_rect(additional_size, rounding)`
These two methods draw a rectangle around the elements between `begin_rect` and `end_rect`

### `imgui.begin_disabled(disabled=true)`
### `imgui.end_disabled()`

These two methods will disable and darken elements in between it.

### `imgui.button(label, size)`
Draws a button with `label` text, and optional `size`.

`size` is a Vector2f or a size 2 array.

Returns `true` when the user presses the button.

### `imgui.small_button(label)`

### `imgui.invisible_button(id, size, flags)`
`size` is a Vector2f or a size 2 array.
`flags` is an int representing a set of ImGuiButtonFlags_ enum values. 

Can be used to create some advanced custom widgets with the right flags, ex:
```lua
-- use `4096 + 1024` to specify the `AllowsOverlap` and `Repeat` flags
 local text = "You can click and hold me"
 imgui.begin_rect()
 local pos = imgui.get_cursor_screen_pos()
 local clicked = imgui.invisible_button(imgui.get_id(label), imgui.calc_text_size(text), 4096 + 1024)
 imgui.set_cursor_screen_pos(pos)
 imgui.text(text)
 imgui.end_rect() 
```
### `imgui.arrow_button(id, dir)`
`dir` is an `ImguiDir`

### `imgui.text(text)`
Draws text.

### `imgui.text_colored(text, color)`
Draws text with color.

`color` is an integer color in the form ARGB.

### `imgui.checkbox(label, value)`
Returns a tuple of `changed`, `value`

### `imgui.drag_float(label, value, speed, min, max, display_format (optional))`
Returns a tuple of `changed`, `value`

### `imgui.drag_float2(label, value (Vector2f), speed, min, max, display_format (optional))`
Returns a tuple of `changed`, `value`

### `imgui.drag_float3(label, value (Vector3f), speed, min, max, display_format (optional))`
Returns a tuple of `changed`, `value`

### `imgui.drag_float4(label, value (Vector4f), speed, min, max, display_format (optional))`
Returns a tuple of `changed`, `value`

### `imgui.drag_int(label, value, speed, min, max, display_format (optional))`
Returns a tuple of `changed`, `value`

### `imgui.slider_float(label, value, min, max, display_format (optional))`
Returns a tuple of `changed`, `value`

### `imgui.slider_int(label, value, min, max, display_format (optional))`
Returns a tuple of `changed`, `value`

### `imgui.input_text(label, value, flags (optional))`
Returns a tuple of `changed`, `value`, `selection_start`, `selection_end`

### `imgui.input_text_multiline(label, value, size, flags (optional))`
Returns a tuple of `changed`, `value`, `selection_start`, `selection_end`

### `imgui.combo(label, selection, values)`
Returns a tuple of `changed, value`. 

`changed` = true when selection changes. 

`value` is the selection index within `values` (a table)

`values` can be a table with any type of keys, as long as the values are strings.

### `imgui.color_picker(label, color, flags)`

Returns a tuple of `changed`, `value`. `color` is an integer color in the form ABGR which `imgui` and `draw` APIs expect.

### `imgui.color_picker_argb(label, color, flags)`

Returns a tuple of `changed`, `value`. `color` is an integer color in the form ARGB.

### `imgui.color_picker3(label, color (Vector3f), flags)`

Returns a tuple of `changed`, `value`

### `imgui.color_picker4(label, color (Vector4f), flags)`

Returns a tuple of `changed`, `value`

### `imgui.color_edit(label, color, flags)`

Returns a tuple of `changed`, `value`. `color` is an integer color in the form ABGR which `imgui` and `draw` APIs expect.

### `imgui.color_edit_argb(label, color, flags)`

Returns a tuple of `changed`, `value`. `color` is an integer color in the form ARGB.

### `imgui.color_edit3(label, color (Vector3f), flags)`

Returns a tuple of `changed`, `value`

### `imgui.color_edit4(label, color (Vector4f), flags)`

Returns a tuple of `changed`, `value`

`flags` for `color_picker/edit` APIs: `ImGuiColorEditFlags`

### `imgui.tree_node(label)`
### `imgui.tree_node_ptr_id(id, label)`
### `imgui.tree_node_str_id(id, label)`
### `imgui.tree_pop()`
All of the above `tree` functions must have a corresponding `tree_pop`!
Note that tree_nodes are especially prone to ID conflicts, but these are easily resolvable especially when building from a table. In many cases using the `ptr_id` version with the iterator variable will suffice.

### `imgui.same_line()`
### `imgui.spacing()`
### `imgui.new_line()`
### `imgui.is_item_hovered(flags)`
### `imgui.is_item_active()`
### `imgui.is_item_focused()`

### `imgui.collapsing_header(name)`

### `imgui.load_font(filepath, size, [ranges])`
Loads a font file from the `UnrealVRMod/[GAMENAME]/fonts` subdirectory at the specified `size` with optional Unicode `ranges` (an array of start, end pairs ending with 0). Returns a handle for use with `imgui.push_font()`. If `filepath` is nil, it will load the default font at the specified size. Most default Windows fonts should be compatible.

### `imgui.push_font(font)`
Sets the font to be used for the next set of ImGui widgets/draw commands until `imgui.pop_font` is called.

### `imgui.pop_font()`
Unsets the previously pushed font.

### `imgui.get_default_font_size()`
Returns size of the default font for REFramework's UI.

### `imgui.set_next_window_pos(pos (Vector2f or table), condition, pivot (Vector2f or table))`
`condition` is the `ImGuiCond` enum.

```cpp
enum ImGuiCond_
{
    ImGuiCond_None          = 0,        // No condition (always set the variable), same as _Always
    ImGuiCond_Always        = 1 << 0,   // No condition (always set the variable)
    ImGuiCond_Once          = 1 << 1,   // Set the variable once per runtime session (only the first call will succeed)
    ImGuiCond_FirstUseEver  = 1 << 2,   // Set the variable if the object/window has no persistently saved data (no entry in .ini file)
    ImGuiCond_Appearing     = 1 << 3    // Set the variable if the object/window is appearing after being hidden/inactive (or the first time)
};
```

### `imgui.set_next_window_size(size (Vector2f or table), condition)`
`condition` is the `ImGuiCond` enum.



### `imgui.get_mouse()`
Returns a `Vector2f` corresponding to the user's actual mouse position.

### `imgui.progress_bar(progress, size, overlay)`
`progress` is a float between 0 and 1.

`size` is a `Vector2f` or a size 2 array.

`overlay` is a string on top of the progress bar.

```lua
local progress = 0.0

uevr.sdk.callbacks.on_frame(function()
    progress = progress + 0.001
    if progress > 1.0 then 
        progress = 0.0
    end
end)

uevr.sdk.callbacks.on_draw_ui(function()
    imgui.progress_bar(progress, Vector2f.new(200, 20), string.format("Progress: %.1f%%", progress * 100))
end)
```

### `imgui.item_size(pos, size)`

### `imgui.item_add(pos, size)`

Adds an item with the specified position and size to the current window.

### `imgui.draw_list_path_clear()`

Clears the current window's draw list path.

### `imgui.draw_list_path_line_to(pos)`

Adds a line to the current window's draw list path given the specified `pos`

- `pos` is a `Vector2f` or a size 2 array.

To draw on the main screen you can create a new window scaled to cover the entire screen and set the background style color to have 0 alpha and use appropriate Window flags to prevent stealing inputs or moving the window. This will not align correctly in VR.

### `imgui.draw_list_path_stroke(color, closed, thickness)`

Strokes the current window's background draw list path with the specified `color`, `closed` state, and `thickness`. This can be called to add thickness to the lines and outlines of calls from the [draw](draw.md) API.

- `color` is an integer color in the form ARGB.
- `closed` is a bool.
- `thickness` is a float.

### `imgui.get_key_index(imgui_key)`

Returns the index of the specified `imgui_key`.
### `imgui.is_key_down(key)`

Returns true if the specified `key` is currently being held down.
### `imgui.is_key_pressed(key)`

Returns true if the specified `key` was pressed during the current frame.
### `imgui.is_key_released(key)`

Returns true if the specified `key` was released during the current frame.
### `imgui.is_mouse_down(button)`

Returns true if the specified mouse `button` is currently being held down.
### `imgui.is_mouse_clicked(button)`

Returns true if the specified mouse `button` was clicked during the current frame.
### `imgui.is_mouse_released(button)`

Returns true if the specified mouse `button` was released during the current frame.
### `imgui.is_mouse_double_clicked(button)`

Returns true if the specified mouse `button` was double-clicked during the current frame.
### `imgui.indent(indent_width)`

Indents the current line by `indent_width` pixels.
### `imgui.unindent(indent_width)`

Unindents the current line by `indent_width` pixels.
### `imgui.begin_tooltip()`

Starts a tooltip window that will be drawn at the current cursor position.
### `imgui.end_tooltip()`

Ends the current tooltip window.
### `imgui.set_tooltip(text)`

Sets the text for the current tooltip window.
### `imgui.open_popup(str_id, flags)`

Opens a popup with the specified str_id and flags.
### `imgui.begin_popup(str_id, flags)`

Begins a new popup with the specified str_id and flags.
### `imgui.begin_popup_context_item(str_id, flags)`
Begins a new popup with the specified str_id and flags, anchored to the last item or group. Automatically handles right click activation.
ex:
```lua
local text = "blah blah blah"
local text2 = "copy me too"
imgui.begin_rect()
imgui.text(text)
imgui.text(text2)
imgui.end_rect()
if imgui.begin_popup_context_item("Context") then
    if imgui.menu_item("Copy text") then
        imgui.set_clipboard(text..text2)
        imgui.close_current_popup()
    end
    imgui.end_popup()
end
```

### `imgui.end_popup()`

Ends the current popup window.
### `imgui.close_current_popup()`

Closes the current popup window.
### `imgui.is_popup_open(str_id)`

Returns true if the popup with the specified str_id is open.
### `imgui.calc_text_size(text)`

Calculates and returns the size of the specified text as a Vector2f.
### `imgui.get_window_size()`

Returns the size of the current window as a Vector2f.
### `imgui.get_window_pos()`

Returns the position of the current window as a Vector2f.
### `imgui.set_next_item_open(is_open, condition)`

Sets the open state of the next collapsing header or tree node to `is_open` based on the specified `condition`.
### `imgui.begin_list_box(label, size)`

Begins a new list box with the specified label and size.
### `imgui.end_list_box()`

Ends the current list box.
### `imgui.begin_menu_bar()`

Begins a new menu bar. Windows and child windows can be created with a flag to draw a space for menu bars. Unlike window and child window begin and end calls this must be handled with an `if then end` block or you will crash
### `imgui.end_menu_bar()`

Ends the current menu bar.
### `imgui.begin_main_menu_bar()`

Begins a menu bar at the top of the main viewport. Does not require a new window to draw on, however it is best to have an invisible window flagged to ignore inputs and focus so that menus don't lose focus as easily.

### `imgui.end_main_menu_bar()`

Ends the main menu bar.
### `imgui.begin_menu(label, enabled)`

Begins a new menu with the specified label. The menu will be disabled if enabled is false but enabling a menu does not force it to open. Unlike window and child window begin and end calls this must be handled with an `if then end` block or you will crash. 

Menus do not require a menu bar and can be added to normal windows. When doing so they will stack vertically instead of horizontally and will open immediately outside the window on whichever side is available. To account for this you can draw the menu inside a child window.

Menus can be nested with additional menus, have great support for controllers, and can host any ImGui widget, not just menu items. They are probably the ideal structure for UEVR.

ex:
```lua
 imgui.begin_window("TEST###test")
 imgui.begin_child_window("child",  imgui.get_window_size() * 0.5, true)
 if imgui.begin_menu("menu", true) then
    if imgui.menu_item("I do nothing") then donothing() end
    local c, nv = imgui.slider_float("Float Slider", float_value, 0.01, 3.0)
    if c and float_value ~= tempscalar then float_value = nv end
    local c, nt, selection_start, selection_end = imgui.input_text_multiline("InputText", text_buffer, Vector2f.new(200, 200))
    if c then text_buffer = nt end
    imgui.begin_group()
    local c, nv = imgui.checkbox("Checkbox", checked)
    if c then checked = nv end
    imgui.same_line()
    imgui.text(checked and "Doing thing!" or "Do thing?")
    imgui.end_group()
    imgui.end_menu()

 end
 if imgui.begin_menu("menu2") then
    imgui.text("hi")
  imgui.end_menu()

 end
 imgui.end_child_window()
 imgui.end_window()
```


### `imgui.end_menu()`

Ends the current menu.
### `imgui.menu_item(label, shortcut, selected, enabled)`

Adds a menu item with the specified label, shortcut, selected state, and enabled state. Shortcuts do NOT work, they are only there for display purposes (this is an ImGui issue)
### `imgui.get_display_size()`

Returns the size of the display as a `Vector2f.`
### `imgui.push_item_width(item_width)`

Pushes the width of the next item to `item_width` pixels.
### `imgui.pop_item_width()`

Pops the last item width off the stack.
### `imgui.set_next_item_width(item_width)`

Sets the width of the next item to `item_width` pixels.
### `imgui.calc_item_width()`

Calculates and returns the current item width.
### `imgui.push_style_color(style_color, color)`

Pushes a new style color onto the style stack. Color should be in ARGB int format. Make sure to track your pushes correctly. Extra pops will cause crashes. Unpopped pushes will carry over to the next frame and if unhandled will eventually crash due to memory leakage.
### `imgui.pop_style_color(count)`

Pops count style colors off the style stack.
### `imgui.push_style_var(idx, value)`

Pushes a new style variable onto the style stack. Make sure to track your pushes correctly. Extra pops will cause crashes. Unpopped pushes will carry over to the next frame and if unhandled will eventually crash due to memory leakage.
### `imgui.pop_style_var(count)`

Pops count style variables off the style stack.
### `imgui.get_cursor_pos()`

Returns the current cursor position as a `Vector2f`.
### `imgui.set_cursor_pos(pos)`

Sets the current cursor position to `pos`. The cursor refers to the position of the drawing cursor to position elements, not the mouse cursor. If needed you can set the mouse position using the PlayerController class in Unreal Engine.
### `imgui.get_cursor_start_pos()`

Returns the initial cursor position as a `Vector2f`.
### `imgui.get_cursor_screen_pos()`

Returns the current cursor position in screen coordinates as a `Vector2f`. The cursor refers to the position of the drawing cursor to position elements, not the mouse cursor. If needed you can set the mouse position using the PlayerController class in Unreal Engine.
### `imgui.set_cursor_screen_pos(pos)`

Sets the current cursor position in screen coordinates to `pos`.
### `imgui.set_item_default_focus()`

Sets the default focus to the next widget.

## Scroll APIs

### `imgui.get_scroll_x()`

Returns the horizontal scroll position.
### `imgui.get_scroll_y()`

Returns the vertical scroll position.
### `imgui.set_scroll_x(scroll_x)`

Sets the horizontal scroll position to `scroll_x`.
### `imgui.set_scroll_y(scroll_y)`

Sets the vertical scroll position to `scroll_y`.
### `imgui.get_scroll_max_x()`

Returns the maximum horizontal scroll position.
### `imgui.get_scroll_max_y()`

Returns the maximum vertical scroll position.
### `imgui.set_scroll_here_x(center_x_ratio)`

Centers the horizontal scroll position.
### `imgui.set_scroll_here_y(center_y_ratio)`

Centers the vertical scroll position.
### `imgui.set_scroll_from_pos_x(local_x, center_x_ratio)`

Sets the horizontal scroll position from the specified `local_x` and `center_x_ratio`.
### `imgui.set_scroll_from_pos_y(local_y, center_y_ratio)`

Sets the vertical scroll position from the specified `local_y` and `center_y_ratio`.

## Table API
### `imgui.begin_table(str_id, column, flags, outer_size, inner_width)`

Begins a new table with the specified `str_id`, `column` count, `flags`, `outer_size`, and `inner_width`.

- `str_id` is a string.
- `column` is an integer.
- `flags` is an optional `ImGuiTableFlags` enum.
- `outer_size` is a `Vector2f` or a size 2 array.
- `inner_width` is an optional float.

Tables can be difficult to construct effectively and may cause crashes. Once mastered they are one of the most powerful structures in ImGui. Here's a minimal setup example:
```lua
  if imgui.begin_table(label, 2, flags) then
    imgui.table_setup_column("Name", column_flags)
    imgui.table_setup_column("Value", column_flags)
    imgui.indent()
    for k, v in pairs(data) do
        imgui.table_next_row()
        imgui.table_set_column_index(0)
        imgui.text(k)
        imgui.table_next_column()
        imgui.push_id(k)
        imgui.text(v)
        imgui.pop_id()
    end
    imgui.unindent()
    imgui.end_table()
    end
```



### `imgui.end_table()`

Ends the current table.
### `imgui.table_next_row(row_flags, min_row_height)`

Begins a new row in the current table with the specified `row_flags` and `min_row_height`.

- `row_flags` is an optional `ImGuiTableRowFlags` enum.
- `min_row_height` is an optional float.

### `imgui.table_next_column()`

Advances to the next column in the current table.
### `imgui.table_set_column_index(column_index)`

Sets the current column index to column_index.
### `imgui.table_setup_column(label, flags, init_width_or_weight, user_id)`

Sets up a column in the current table with the specified label, flags, init_width_or_weight, and user_id.
### `imgui.table_setup_scroll_freeze(cols, rows)`

Sets up a scrolling region in the current table with cols columns and rows rows frozen.
### `imgui.table_headers_row()`

Submits a header row in the current table.
### `imgui.table_header(label)`

Submits a header cell with the specified label in the current table.
### `imgui.table_get_sort_specs()`

Returns the sort specifications for the current table.
### `imgui.table_get_column_count()`

Returns the number of columns in the current table.
### `imgui.table_get_column_index()`

Returns the current column index.
### `imgui.table_get_row_index()`

Returns the current row index.
### `imgui.table_get_column_name(column)`

Returns the name of the specified `column` in the current table.
### `imgui.table_get_column_flags(column)`

Returns the flags of the specified `column` in the current table.
### `imgui.table_set_bg_color(target, color, column)`

Sets the background color of the specified `target` in the current table with the given `color` and `column` index.
