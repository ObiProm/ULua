# ULua API Documentation

## Main ideas

Every node, which was created by lua code is just table wrapper for Godot.Node class with specific metatable. It means that you can still use native functions (not all), and add custom fields for this node. But sometimes nodes are not tables, like after using `PackedScene:Instantiate`, so you need to create wrapper for it by `game.CreateNodeWrapper`

Which functions you cant call for `Node`?
Here is list:
`Rpc`, `RpcId`, `Connect`, `Disconnect`, `EmitSignal`, `Multiplayer`

To use `Rpc` read [RpcNode](#rpcnode)<br>
To use some `Multiplayer` functions use `IsServer()` and `GetUniqueId()`

Navigation:
- [globals](#globals)
- [game](#game)
- [LoadingUI](#loadingui)
- [RpcNode](#rpcnode)
- Structs: 
    - [Room](#room)
    - [RoomInfo](#roominfo)
    - [PlayerLobby](#playerlobby)
    - [PlayerCamera](#playercamera)
- [Discord Activity](#discordactivity)
- [Mod structure](#mod-structure)
- [Recomendations](#recomendations)

<br><br>

# Globals

### `print(...)`
Prints a message to the console.

### `printError(...)`
Prints an error message (displayed in red).

### `printWarning(...)`
Prints a warning message (displayed in yellow).

---

### `Vector3(X, Y, Z)` → `Vector3`

Creates a 3D vector.

**Parameters:**
- `X`, `Y`, `Z` (`number`) — Coordinates.

**Returns:**
- `Vector3` — A new 3D vector.

---

### `Vector2(X, Y)` → `Vector2`

Creates a 2D vector.

**Parameters:**
- `X`, `Y` (`number`) — Coordinates.

**Returns:**
- `Vector2` — A new 2D vector.

---

### `IsServer()` → `boolean`

Checks if the current instance is running as a server.

**Returns:**
- `boolean` — `true` if the application is a server.

---

### `GetUniqueId()` → `boolean`

Returns unique ID

**Returns:**
- `number` — unique ID of current running instace (1 = server)

<br>

# game
The main class for interacting with the native functions

### `game.GetCurrentScene()` → `Node`

Returns currently active scene root node.

**Returns:**
- `Node` — scene node.

### `game.GetRoot()` → `Node`

Reference to the global scene root.

**Returns:**
- `Node` — root node of app.


### `game.GetUserdataName(object)` → `string`

Returns the type name of a userdata object (e.g., Godot node type).

**Parameters:**
- `object` (`userdata`) — The userdata object.

**Returns:**
- `string` — The type name (e.g., `"Node3D"`).

**Example:**
```lua
local node = game.GetPlayerPacked()
print(game.GetUserdataName(node)) -- "PackedScene"
```

---

### `game.CreateNode(classname)` → `T`

Creates a new node instance of the specified class.

**Parameters:**
- `classname` (`string`) — The name of the class to instantiate.

**Returns:**
- `T` — A new node instance, wrapped as a Lua object.

**Example:**
```lua
local entity = game.CreateNode("CharacterBody3D")
```

---

### `game.RegisterClass(className, table, baseType?)`

Registers a custom node class.

**Parameters:**
- `className` (`string`) — The name of the new class.
- `table` (`table`) — A Lua table containing methods and properties.
- `baseType` (`string?`) — Optional base class name. Defaults to `"Node"`.

**Example:**
```lua
game.RegisterClass("SuperNode", {
    Talk = function(self, amount)
        print("Hello!")
    end
}, "Node")
```

---

### `game.LoadScene(path)` → `PackedScene`

Loads a scene from the given path.

**Parameters:**
- `path` (`string`) — Resource path (e.g., `"Scenes/Game.tscn"`).

**Returns:**
- `PackedScene` — A packed scene ready for instantiation.

**Example:**
```lua
local scene = game.LoadScene("Scenes/Game.tscn")
game.CurrentScene:Instantiate(scene)
```

---

### `game.GetPlayerPacked()` → `PackedScene`

Returns the default player scene.

**Returns:**
- `PackedScene` — Player scene template. With See [PlayerCamera](#playercamera) for logging utilities. as root

**Example:**
```lua
local playerObject = playerScene:Instantiate()
playerObject.Player = playerLobby
```

---

### `game.GetUIPacked()` → `PackedScene`

Returns the default UI scene.

**Returns:**
- `PackedScene` — UI scene template.

---

### `game.CreateNodeWrapper(node)` → `Node`

Converts a raw Godot Node into a Lua-wrapped Node with methods and metatable.

**Parameters:**
- `node` (`Node`) — The raw Godot node.

**Returns:**
- `Node` — The wrapped node.

**Note:** Use this when working with nodes passed from Godot scripts.

<br>

# LoadingUI

Controls the loading screen interface.

### `LoadingUI.SetPercentage(number)`
Sets the progress bar value.

**Parameters:**
- `number` (`number`) — Progress value from 0 to 100.

**Example:**
```lua
LoadingUI.SetPercentage(75)
```

---

### `LoadingUI.SetStatus(text)`
Updates the status text on the loading screen.

**Parameters:**
- `text` (`string`) — The status message.

**Example:**
```lua
LoadingUI.SetStatus("Loading assets...")
```

---

### `LoadingUI.Show()`
Displays the loading UI by playing the forward animation.

---

### `LoadingUI.Hide()`
Hides the loading UI by playing the reverse animation.

---

## RpcNode

A specialized node for remote procedure calls (RPC). Regular nodes do not support RPC by default. It supports only next types: `number`, `string`, `boolean`, `Vector2`, `Vector3`, `table`

**Warning** to avoid code execution errors, you need to create functions using ":"(colon) not "."(dot) since the first argument will be self

### `RpcNode:RpcReliable(methodName, ...)`
Sends a reliable RPC call to all connected clients.

**Parameters:**
- `methodName` (`string`) — Name of the method to call.
- `...` — Arguments (supports `number`, `string`, `table`).

**Example:**
```lua
rpcNode:RpcReliable("SyncPosition", Vector3(100, 5, 200))
```

---

### `RpcNode:RpcUnreliable(methodName, ...)`
Sends an unreliable RPC call (faster, but may be dropped).

**Parameters:**
- `methodName` (`string`) — Name of the method.
- `...` — Arguments.

---

### `RpcNode:RpcIdReliable(reciever, methodName, ...)`
Sends a reliable RPC call to a specific client by ID.

**Parameters:**
- `reciever` (`number`) — Client ID.
- `methodName` (`string`) — Method name.
- `...` — Arguments.

---

### `RpcNode:RpcIdUnreliable(reciever, methodName, ...)`
Sends an unreliable RPC call to a specific client.

---

### `RpcNode:BindLuaTable(table)`
Binds a Lua table to the node to handle incoming RPC calls.

**Parameters:**
- `table` (`table`) — Table containing callable methods.

**Example:**
```lua
local handlers = {
    OnPlayerJoin = function(self, name)
        print(name .. " joined")
    end
}
rpcNode:BindLuaTable(handlers)
```

---

### `RpcNode:HasLuaTable()` → `boolean`
Checks whether a Lua table is bound to the node.

**Returns:**
- `boolean` — `true` if a table is bound.

---

### `RpcNode:GetLuaTable()` → `table`
Retrieves the bound Lua table.

**Returns:**
- `table` — The bound table, or `nil` if none.

<br>

# DiscordActivity

Controls Discord Rich Presence activity.

### `DiscordActivity.SetState(state)`

Sets the activity state (the lower line in Rich Presence).

**Parameters:**
- `state` (`string`) — The state text, e.g. `"In lobby"` or `"Playing Solo"`.

---

### `DiscordActivity.SetDetails(details)`

Sets the activity details (the upper line in Rich Presence).

**Parameters:**
- `details` (`string`) — The description of current activity, e.g. `"In menu"` or `"Survival Mode"`.

<br>

# PlayerCamera

**Fields:**
- `Player` (`PlayerLobby`) — Sets player to this player controller

<br>

# PlayerLobby

Represents a player in the lobby or a room.

**Fields:**
- `PeerID` (`number`) — Network ID of the player (64-bit integer, represented as number in Lua).
- `PlayerName` (`string`) — Display name of the player.
- `Inventory` (`table<string>`) — List of items in player's inventory (e.g. `{"Agent"}`).
- `Disconnected` (`fun()|nil`) — Callback invoked when player disconnects.
- `CurrentRoom` (`Room|nil`) — Reference to the room the player is currently in.
- `CurrentTeam` (`Team|nil`) — Reference to the team the player is currently assigned to.
- `IsRoomAdmin` (`boolean`) — Read-only: true if this player is the admin of the current room.

<br>

# RoomInfo

Metadata about a room, used for room listing.

**Fields:**
- `RoomId` (`string`) — Unique identifier for the room.
- `RoomName` (`string`) — Display name of the room.
- `MaxPlayers` (`integer`) — Maximum number of players allowed.
- `Settings` (`RoomSettings`) — Game settings (map, mode, etc.).
- `PlayersCount` (`integer`) — Current number of players in the room.

<br>

# Room

Represents an active room instance.

**Fields:**
- `Info` (`RoomInfo`) — Basic metadata about the room.
- `PlayersCount` (`integer`) — Read-only: current number of connected players.
- `Players` (`table<number, PlayerLobby>`) — Map of `PeerID` → `PlayerLobby` instances.
- `Teams` (`table<string, Team>`) — Map of team name → `Team` instance.
- `Password` (`string`) — Room password; empty string means no password.
- `IsVisible` (`boolean`) — Whether the room appears in the public room list.
- `GameStarted` (`boolean`) — True if the game has started and players can no longer join.
- `Admin` (`PlayerLobby`) — The player who owns/created the room.
- `BecomeEmpty` (`fun()|nil`) — Event callback triggered when the last player leaves the room.

**Example:**
```lua
-- Iterate all players
for peerId, player in pairs(room.Players) do
    print(player.PlayerName)
end

-- Check admin
if room.Admin.PeerID == myPeerId then
    print("I am the room owner")
end
```

---

## Mod structure

Mods have lua folder with `autorun` folder. Scripts in this folder will start after all systems loaded, but before menu started to render.

**Example structure:**
```
YourCoolMod/
└── Gamemodes.json
└── lua/
    └── autorun/
        └── lua_script1.lua
        └── lua_script2.lua
```

Gamemodes.json contains config with playerble gamemodes, example:
```json
{
    "Standard": {
        "Description": "Standard mode with a variety of modifiers",
        "LoadingManagerScript": "StandardLoadingManager",
        "Modifiers": {
            "SplitRewards": {
                "Type": "bool",
                "Name": "Split Rewards",
                "Description": "All players receive money for damage dealt to zombies, even if they didn't deal it personally. Rewards are shared across the team."
            }
        }
    }
}
```


## Recomendations
Place all scripts which registers classes into `lua/autorun/`
