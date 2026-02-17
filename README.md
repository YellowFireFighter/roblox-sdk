# Roblox sdk
> Lua Perception SDK for Roblox · `version-bd08027bb04e4045`

---

## Getting Started

```lua
local rbx = require("[LUA] Roblox SDK")
rbx.init(proc)
```

---

## Example

Iterate all players and display their health and position:

```lua
local base = proc:get_base()
local dm = rbx.resolve_datamodel(base)
local ve = rbx.resolve_visual_engine(base)

local players = rbx.get_players_service(dm)
local lp = rbx.get_local_player(players)

local dims = ve:get_dimensions()
local view = ve:get_viewmatrix()

for _, player in ipairs(rbx.get_all_players(players)) do
    if player.address == lp.address then goto continue end

    local char = player:get_character()
    local hum  = rbx.get_humanoid(char)
    local hrp  = rbx.get_root_part(char)
    local prim = hrp and hrp:get_primitive()
    local pos  = prim and prim:get_position()

    if pos and hum and hum:get_health() > 0 then
        local screen, onscreen = ve:world_to_screen(pos, dims, view)
        if onscreen then
            print(player:get_name(), screen.x, screen.y, hum:get_health())
        end
    end

    ::continue::
end
```

---

## API Reference

### Initialization

#### `rbx.init(proc)`
Sets the process handle used for all memory reads. Call this before anything else.

#### `rbx.get_proc()`
Returns the currently set process handle.

---

### Resolving the Engine

#### `rbx.resolve_datamodel(base)` → `Instance | nil`
Resolves the Roblox `DataModel` from the process base address. Tries the VisualEngine chain first, then falls back to the FakeDataModel path.

#### `rbx.resolve_visual_engine(base)` → `VisualEngine | nil`
Returns the `VisualEngine` object from the process base address.

---

### Players

#### `rbx.get_players_service(datamodel)` → `Instance | nil`
Finds the `Players` service inside the DataModel.

#### `rbx.get_local_player(players_svc)` → `Player | nil`
Returns the local player from the Players service.

#### `rbx.get_all_players(players_svc)` → `Player[]`
Returns every player in the server as a list of `Player` objects.

---

### Characters & Parts

#### `rbx.get_humanoid(char)` → `Humanoid | nil`
Finds the `Humanoid` instance inside a character model.

#### `rbx.get_root_part(char)` → `Part | nil`
Finds `HumanoidRootPart` inside a character and returns it as a `Part`.

#### `rbx.get_game_id(datamodel)` → `number | nil`
Returns the game ID.

---

### Utilities

#### `rbx.distance(a, b)` → `number`
Distance between two `{x, y, z}` tables.

```lua
local d = rbx.distance(pos_a, pos_b)
```

---

## Classes

### `Instance`
The base class for anything in the Roblox tree. Most objects you deal with are instances.

```lua
instance:get_name()                          -- string | nil
instance:get_class_name()                    -- string | nil
instance:get_children([ctor])                -- Instance[]
instance:find_first_child(name)              -- Instance | nil
instance:find_first_child_by_class(class)    -- Instance | nil
instance:isvalid()                           -- bool
```

`get_children` accepts an optional constructor so you can get typed children back:
```lua
local kids = some_instance:get_children(Player.new)
```

---

### `Player`
Extends `Instance`. Represents a connected player.

```lua
player:get_character()    -- Instance | nil  (the character model)
player:get_team()         -- number | nil    (raw team pointer)
player:get_name()         -- inherited from Instance
```

---

### `Humanoid`
For reading health and rig data from a character's Humanoid.

```lua
hum:get_health()        -- number | nil  (current HP)
hum:get_max_health()    -- number | nil  (max HP)
hum:get_rig_type()      -- number | nil  (0 = R6, 1 = R15)
```

---

### `Part`
An Instance that has a physics `Primitive` attached to it.

```lua
local prim = part:get_primitive()   -- Primitive | nil
```

---

### `Primitive`
Reads raw physics data from a BasePart. This is where you get world position, size, and rotation.

```lua
prim:get_position()    -- {x, y, z} | nil
prim:get_size()        -- {x, y, z} | nil
prim:get_rotation()    -- 3x3 nested table | nil
```

Getting a part's world position looks like this in practice:
```lua
local hrp  = rbx.get_root_part(character)
local prim = hrp:get_primitive()
local pos  = prim:get_position()
-- pos.x, pos.y, pos.z
```

---

### `VisualEngine`
The rendering engine. Use this for screen dimensions and world-to-screen projection.

```lua
ve:get_dimensions()                       -- {x, y} | nil  (screen size in px)
ve:get_viewmatrix()                       -- number[16] | nil  (row-major 4x4 matrix)
ve:world_to_screen(world, dims, view)     -- {x, y}, bool
```

`world_to_screen` returns the screen position and a boolean indicating whether the point is in front of the camera. Always check the bool before drawing anything:

```lua
local screen, onscreen = ve:world_to_screen(pos, dims, view)
if onscreen then
    -- safe to use screen.x and screen.y
end
```

---

## Low-Level Memory Helpers

If you need to do your own reads, `rbx.mem` exposes the same safe wrappers the SDK uses internally:

```lua
rbx.mem.valid(addr)       -- sanity check an address before reading
rbx.mem.ru64(addr)        -- read u64, returns nil on bad address/read
rbx.mem.ru64_raw(addr)    -- read u64 without value validation (can return 0)
rbx.mem.ru8(addr)         -- read u8
rbx.mem.rf32(addr)        -- read f32
rbx.mem.rs(addr, len)     -- read string
```
