# introduction

In Godot, You have to decide what kind of data to save and how to replicate the scenes when you load the game data. In the official [tutorial](https://docs.godotengine.org/en/stable/tutorials/io/saving_games.html), it used `persistent` group in order to know what nodes needed to save data. However, when you load the data, those nodes might not exist so you couldn't find them when you iterate through the nodes inside the group!

# my solution

Inspired by `multiplayerSpawner`, I realized that I could create a `Node` that was specifically used for loading game data and replicating the scene. This way, I can do `get_tree().get_nodes_in_group("loader")`, get all loaders, and feed them the save data.

For example, I was currently writing a Supermarket game and I needed to save each shelves' locations. For saving, it's pretty easy, just add `Shelf` nodes to `Saver` group and call them when the player saves the game. The save format was in JSON and its type was `Array[Dictionary]`. Thus, the save data would look like this: `[{"type":"shelf","x":-2.5,"y":0.1,"z":-1.5},{"type":"shelf","x":-2.5,"y":0.1,"z":-0.5}]`.

When loading the game, I feed each `Dictionary` to all loaders. The loaders would check `type` and see if it's for them. If yes, it would spawn the node accordingly and return true. It would return false otherwise, like this:

```
extends Node
class_name ShelfLoader

const SHELF = preload("res://scenes/game/furniture/shelf/shelf.tscn")

@export var add_to_path : Node

func load(v:Dictionary) -> bool:
	if v["type"] != "shelf":
		return false

	var n := SHELF.instantiate()
	n.position = Vector3(v["x"], v["y"], v["z"])
	add_to_path.add_child(n)

	return true
```

On the main node (the one who reads the data from the file and feed them into loaders), it would iterate through all `Dictionary` and feed them to each loader until all loaders was tried or a loader said that it was for them (by returning true), like this:

```
const SAVE_PATH = "user://game"

func _ready() -> void:
    if FileAccess.file_exists(SAVE_PATH):
		var file := FileAccess.open(SAVE_PATH, FileAccess.READ)
		var s := file.get_line()
		var d : Array = JSON.parse_string(s)
		load_all(d)

func load_all(d:Array) -> void:
	for i:Dictionary in d:
		for node in get_tree().get_nodes_in_group("loader"):
			if node.load(i):
				break
```

Of course, this isn't very efficient. But my games were all small games and this pattern served me really well so far!
