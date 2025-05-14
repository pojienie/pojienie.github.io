# introduction

Do you know about Godot's `await` keyword? For someone coming from a C background, it sounded like a magic. The keyword basically allowed you to hold a function until a signal comes.

## basic syntax

Consider the following code snippet:

```
func foo() -> void:
    print("this will print upon calling this function")
    await get_tree().create_timer(1.0).timeout
    print("this will print after 1 second")
```

The `foo` function here will print two lines, one will print immediately and the other will print after a second. All this is done without you creating a timer node which could be useful when you don't know how many timers you would need.

## usage

### auto repeating functions

I personally used this to create a self-loop function. For example, in some games, when you press the "save" button, it would pop out a "saved!" floating text to indicate that you have saved the game. This is how I do it:

```
func _label_animation(label:Label) -> void:
	if label == null:
		label = Label.new()
		label.text = "saved!"
		label.position = get_rect().size / 2.0
		label.position.x -= label.get_rect().size.x / 2
		add_child(label)
		await get_tree().process_frame
		_label_animation(label)
		return

	label.position.y += 1
	label.self_modulate.a -= 0.05
	if label.self_modulate.a <= 0.05:
		label.queue_free()
		return
	await get_tree().process_frame
	_label_animation(label)
```

With this, you can just call `_label_animation(null)` and it would automatically create a floating text. This function may seem recursive but, in fact, it wouldn't cause stack overflow because, once the `await` is encountered, it basically "escaped" stack.
