# introduction

When programming a multiplayer game in Godot, there are two nodes ther are useful: MultiplayerSyncronizer and MultiplayerSpawner. The former will sync properties to every other clients and the latter will sync nodes that are spawned during runtime to every other clients.

These two nodes rely on their multiplayer authority. They will only sync from their authority client to other clients. For example, if there are two characters in the game, A and B, both with their multiplayer authority correctly set, a MultiplayerSyncronizer will only sync from A to B if it has A authority.

(This works in Godot 4. I am not sure about Godot 3.)

# the problem

However, Godot does not help you to sync the authority between clients. Meaning that the whole scheme would break if two clients have the same nodes but with different authority. Thus, we Godot programmers need to find a way to sync them ourselves.

It's not too big of a problem for MultiplayerSyncronizer. But it's a huge problem for MultiplayerSpawner because MultiplayerSpawner would always spawn a node with authority of `1` (server) and we need to find a way to tell every clients "hey, this node's authority is A's" somehow.

# common solution

A common solution is to encode authority into the node's name. After setting up MultiplayerSpawner, before adding the new node to the tree, you would do something like this:

```
var id := multiplayer.get_remote_sender_id() # This is a placeholder. Replace it with however you get the player's ID
var node := NODE.instantiate()
node.name = "node%d" % id
# adding "node.set_multiplayer_authority" wouldn't work because this ID is not synced between clients. You have to do it yourself.
add_child(node)
```

And then in the `_ready()` function of the node:

```
func _ready():
    var id := name.substr(4).to_int()
    set_multiplayer_authority(id)
```

Well, this is all fine and dandy if you only have one node to spawn. What if you have multiple? You'd start encountering name collision or you'd need to start a more complex node naming scheme.

# my solution

There is a less known feature of MultiplayerSpawner: `spawn_function` and `spawn`. When you call `spawn`, it would tell every client to do the same thing with the same argument. And the function it called is set in `spawn_function`. Thus, an easy way to ensure everyone has the same authority for spawned nodes is to use these. When the game is loading up, do this:

```
$MultiplayerSpawner.spawn_function = func(id):
    var node := NODE.instantiate()
    node.set_multiplayer_authority(id)
    return node
```

Then, when you need to spawn a node for every clients, do:

```
var id := multiplayer.get_remote_sender_id() # This is a placeholder. Replace it with however you get the player's ID
$MultiplayerSpawner.spawn(id)
```

And that's it! If you have multiple things you want to sync during spawning phase, you can even pass dictionary in it!
