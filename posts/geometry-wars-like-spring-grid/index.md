---
title: Geometry Wars like Spring Grid
date: 2024-05-31
cover: 'cover.png'
cover_alt: 'Spring grid reacting to a directional force'
preview: 'Recreating the background of Geometry Wars for my circle pusher game.'
preview_image: 'preview.png'
---

## Inspiration

Somehow, I found myself watching Geometry Wars gameplay and was mesmerized by the fancy background grid.

![Gameplay video Geometry Wars](./spring_grid_1.mp4)

It's amazing how much the background enhances the impact / weight of certain attacks.

I felt like I could use something similar for my Circle Pushing game, or at the very least, I wanted to know how to create something similar even if I can't find a use for it 😅

## Research

In my research, I found this [great tutorial](https://code.tutsplus.com/make-a-neon-vector-shooter-in-xna-the-warping-grid--gamedev-9904t). I recommend reading through it to understand the setup and perhaps delve deeper into some of the physics concepts.

TLDR: The grid is composed of points of mass connected by springs.

## Implementation

I followed the tutorial and translated it into GDScript. In the process, I created three new scripts:

### grid.gd

- Initializes the grid:
  - Creates and stores all `Springs` and `PointMasses`.
  - Creates `Line2D` objects to connect each `PointMass`.
- Calls `spring.update()` and `point.update()` in `_process()` to calculate the physics.
- Updates the position of the sprite representing the `PointMass` and the `Line2D` that connect everything.
- Provides methods to interact with the grid:
  - `apply_directed_force()`
  - `apply_implosive_force()`
  - `apply_explosive_force()`
- Translates grid position to screen position with `to_vec_2()`.

  - By default, the grid space is per cell, so a 40 by 40 grid is `Vector2(40, 40)`. `to_vec_2()` translates that to screen space.

<details>
	<summary>
	Code
	</summary>

```php
  class_name Grid
  extends Node

  var size: Vector2
  var spacing: Vector2
  var springs: Array[Spring] = []
  var points := []
  var dots := []
  var lines_vertical := []
  var lines_horizontal := []

  var dot := preload("res://stages/background/dot.tscn")

  func _init(_size: Vector2, _spacing: Vector2) -> void:
  	size = _size
  	spacing = _spacing

  	var number_columns: int = size.x / spacing.x + 1
  	var number_rows: int = size.y / spacing.y + 1
  	var points_fixed := []
  	var counter_point_mass := 0

  	# Create the point masses
  	for row_i in size.x:
  		var new_line := Line2D.new()
  		new_line.width = 3.0
  		var new_horizontal_line := Line2D.new()
  		new_horizontal_line.width = 3.0
  		lines_vertical.push_back(new_line)
  		lines_horizontal.push_back(new_horizontal_line)
  		add_child(new_line)
  		for column_i in size.y:
  			add_child(new_horizontal_line)
  			if points.size() - 1 < row_i:
  				points.push_back([])
  			points[row_i].push_back(PointMass.new(Vector3(row_i, column_i, 0), 1, counter_point_mass))
  			if points_fixed.size() - 1 < row_i:
  				points_fixed.push_back([])
  			points_fixed[row_i].push_back(PointMass.new(Vector3(row_i, column_i, 0), 0, counter_point_mass))
  			if dots.size() - 1 < row_i:
  				dots.push_back([])
  			var new_dot := dot.instantiate()
  			new_dot.debug_text = "%s-%s" % [row_i, column_i]
  			dots[row_i].push_back(new_dot)
  			add_child(new_dot)
  			lines_vertical[row_i].add_point(Vector2.ZERO, column_i)
  			lines_horizontal[row_i].add_point(Vector2.ZERO, row_i)
  			counter_point_mass = counter_point_mass + 1

  	# Link the point masses with springs
  	for row_i in number_rows - 1:
  		for column_i in number_columns - 1:
  			const stiffness: float = 0.28
  			const damping: float = 0.06

  			if row_i == 0 or column_i == 0 or row_i == number_rows -1 or column_i == number_columns - 1: # anchor the border of the grid
  				springs.push_back(Spring.new(points_fixed[row_i][column_i], points[row_i][column_i], 0.1, 0.1))
  			elif row_i % 3 == 0 and column_i % 3 == 0:
  				springs.push_back(Spring.new(points_fixed[row_i][column_i], points[row_i][column_i], 0.002, 0.02))
  			if row_i > 0:
  				springs.push_back(Spring.new(points[row_i - 1][column_i], points[row_i][column_i], stiffness, damping))
  			if column_i > 0:
  				springs.push_back(Spring.new(points[row_i][column_i - 1], points[row_i][column_i], stiffness, damping))

  	handle_shader_data()

  func _process(delta: float) -> void:
  	for spring in springs:
  		spring.update()
  	for row_i in points.size():
  		var row = points[row_i]
  		for column_i in row.size():
  			var mass = row[column_i]
  			mass.update()
  			dots[row_i][column_i].position = to_vec_2(mass.position)
  			lines_vertical[row_i].set_point_position(column_i, to_vec_2(mass.position))
  			lines_horizontal[column_i].set_point_position(row_i, to_vec_2(mass.position))

  func to_vec_2(vec_3: Vector3) -> Vector2:
  	var factor: float = (vec_3.z + 2000) / 2000
  	var screen_size_half := get_viewport().get_visible_rect().size / 2.0
  	return ((Vector2(vec_3.x, vec_3.y) - screen_size_half) * factor + screen_size_half) * 60

  func apply_directed_force(force: Vector3, position: Vector3, radius: float) -> void:
  	for row in points:
  		for mass in row:
  			if position.distance_squared_to(mass.position) < radius * radius:
  				mass.apply_force(10 * force / (10 + position.distance_to(mass.position)))

  func apply_implosive_force(force: Vector3, position: Vector3, radius: float) -> void:
  	for row in points:
  		for mass in row:
  			var distance_squared := position.distance_squared_to(mass.position)
  			if distance_squared < radius * radius:
  				mass.apply_force(10 * force * (position - mass.position) / (100 + distance_squared))
  				mass.increase_damping(0.6)

  func apply_explosive_force(force: Vector3, position: Vector3, radius: float) -> void:
  	for row in points:
  		for mass in row:
  			var distance_squared := position.distance_squared_to(mass.position)
  			if distance_squared < radius * radius:
  				mass.apply_force(100 * force * (mass.position - position) / (10000 + distance_squared))
  				mass.increase_damping(0.6)
```

</details>

### point_mass.gd

- Represents a point of mass in the grid (🤯)

<details>
	<summary>
	Code
	</summary>

```php
  class_name PointMass
  extends RefCounted

  var id: int
  var position: Vector3
  var velocity: Vector3
  var inverse_mass: float
  var acceleration: Vector3
  var damping := 0.98

  func _init(_position: Vector3, _inverse_mass: float, _id: int) -> void:
  	id = _id
  	position = _position
  	inverse_mass = _inverse_mass

  func apply_force(force: Vector3) -> void:
  	acceleration = acceleration + force * inverse_mass

  func increase_damping(factor: float) -> void:
  	damping = damping * factor

  func update() -> void:
  	velocity = velocity + acceleration
  	position = position + velocity
  	acceleration = Vector3.ZERO

  	if is_zero_approx(velocity.length_squared()):
  		velocity = Vector3.ZERO

  	velocity = velocity * damping
  	damping = 0.98
```

</details>

### spring.gd

- One `Spring` connects two `PointMass` objects.
- In the `Spring`'s `update()`, [Hooke's law](https://en.wikipedia.org/wiki/Hooke%27s_law) is applied, and the force is applied to each point.

<details>
	<summary>
	Code
	</summary>

    ```php
    class_name Spring
    extends RefCounted

    var end_1: PointMass
    var end_2: PointMass
    var target_length: float
    var stiffness: float
    var damping: float


    func _init(_end_1: PointMass, _end_2: PointMass, _stiffness: float, _damping: float) -> void:
    	end_1 = _end_1
    	end_2 = _end_2
    	stiffness = _stiffness
    	damping = _damping
    	# When we create a spring, we set the natural length of the spring
    	# to be just slightly less than the distance between the two end points.
    	# This keeps the grid taut even when at rest and improves the appearance somewhat.
    	target_length = _end_1.position.distance_to(end_2.position) * 0.95


    func update() -> void:
    	var x := end_1.position - end_2.position
    	var length := x.length()
    	var dv: Vector3
    	var force: Vector3

    	if length <= target_length:
    		return

    	# https://en.wikipedia.org/wiki/Hooke's_law
    	x = (x / length) * (length - target_length)
    	dv = end_2.velocity - end_1.velocity
    	force = stiffness * x - dv * damping

    	end_1.apply_force(-force)
    	end_2.apply_force(force)

````

</details>



Now, add a new grid to a scene:

```php
func _ready() -> void:
	grid = Grid.new(Vector2(40, 40), Vector2(1, 1))
	add_child(grid)
````

and apply some force to the grid to see the springs in action:

```php
func _on_timer_timeout() -> void:
	grid.apply_directed_force(Vector3(3.0, 2.0, 0.0), Vector3(10.0, 10.0, 0.0), 2.0)
```

![Shows the grid reacting to a directional force](./spring_grid_2.mp4)

## The Result

It's totally possible that I messed up something in the translation process. I didn’t spend too much time evaluating if the grid behaves as it should. The tutorial mentions additional steps to optimize the implementation. What's apparent pretty quickly is that it’s a costly backdrop. The 40 by 40 grid and its resulting 1600 points drop the FPS to around 90 from 350.

## Next Steps

This seems like a good point to start learning how to write compute shaders 😄 So, that's exactly what I’m currently working on. I will try my best to move the simulation over to the GPU. I have already made some good progress but need a topic to write about next month, so we will have to wait and see if I can pull this off 👀

## That's it!

If you have any questions or feel offended by my bad code, find me on the [Godot Modding Discord](https://discord.godotmodding.com/) and ping me in **#dev-general** or on Twitter [@KANAjetzt](https://twitter.com/KANAjetzt) 👍
