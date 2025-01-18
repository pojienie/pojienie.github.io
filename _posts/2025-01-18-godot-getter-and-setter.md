# introduction

Godot getter and setter are really useful. Here are the things that I use them for.

# clamping

This is pretty self-explainary - A value might have a minimum or maximum values and it would become really tedious to check every time you set the value. In Godot, you can do something like this:

```
const MAX_HEALTH := 100
var health := 100 :
    set(v):
        health = clamp(v, 0, MAX_HEALTH)
```

# serial number

Sometimes, you might want something that would never repeat. In my case, this is used when syncing nodes between clients in a multiplayer game.

```
const MONSTER = preload("res://monster.tscn")

var serial_number := -1 :
    get():
        serial_number += 1
        return serial_number

# somewhere else in the code
@rpc("authority", "call_local", "reliable")
func spawn_monster(n:String):
    var monster := MONSTER.instantiate()
    monster.name = n
    # other stuff to initialize the monster such as its position or health
    add_child(monster)

func _on_timer_timeout() -> void:
    var n := "monster_%d" % serial_number
    spawn_monster.rpc(n)
```

## warning

Usually, we should follow "the principle of least surprise". This use case can be surprising to people who aren't familiar with the codebase (or yourself after 6 months). Thus, perhaps this is better done using a function:

```
var serial_number := -1

func get_next_serial_number() -> int:
    serial_number += 1
    return serial_number
```

The code may look the same but it's way less surprising than using the variable directly.

# sync UI elements

Sometimes, you want a particular UI element to always represent a particular variable in the game such as player health, player mana, etc.

```
@onready var gold_label: Label = $"GoldLabel"

var gold := 0 :
    set(v):
        gold = v
        gold_label.text = "gold:%d" % v
```

## exported variable

Note that, however, if the variable is exported and you set a value there, the `setter` would be called **before** the node is ready which might cause problem. In that case, just `await` for the ready signal:

```
@onready var gold_label: Label = $GoldLabel

var is_ready := false

@export var gold := 0 :
    set(v):
        if !is_ready:
            await ready
        gold = v
        gold_label.text = "gold:%d" % v

func _ready() -> void:
    is_ready = true
```
