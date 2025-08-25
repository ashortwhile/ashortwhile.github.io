## 一、 StaseMachine
```
extends Node
class_name StateMachine

@export var initial_state : NodePath
@export var auto_start = true

var current_state : StateBase = null

@onready var states = get_children()
func _ready() -> void:
	if auto_start:
		launch()


func launch():
		assert(initial_state != null,"错误")
		current_state = get_node(initial_state)
		current_state.enter()


func _physics_process(delta: float) -> void:
	current_state.physics_update(delta)


func _process(delta: float) -> void:
	current_state.update(delta)


func has_state(state_name):
	for s in states:
		if "state_name" in s and s.state_name == state_name:
			return true
	return false


func get_state(state_name):
	for s in states:
		if "state_name" in s and s.state_name == state_name:
			return s
	return null


func transition_to(state_name : String,msg : Dictionary = {}):
	if has_state(state_name):
		var state = get_state(state_name)
		if state:
			current_state = state
			current_state.enter(msg)
```
## 二、 StateBase
```
extends Node
class_name StateBase
var state_name = "StateBase"
func enter(_msg : Dictionary = {}):
	pass
	
func exit():
	pass
	
func update(_delta : float):
	pass

func physics_update(_delta : float):
	pass
```
## 三、player脚本
```
extends CharacterBody2D
class_name Player
@export var player : Player
var current_direction = "down"
```
## 四、角色空闲脚本
```
extends StateBase
@export var player: Player
@export var animatedsprite2d: AnimatedSprite2D
@export var statemachine: StateMachine
func _ready():
	state_name = "idle"
		
func enter(_msg : Dictionary = {}):
	pass
	
func update(_delta : float):
	animatedsprite2d.play(player.current_direction + "_idle")
	if Input.get_vector("left","right","up","down"):
		statemachine.transition_to("move")

func physics_update(_delta : float):
	pass
	
func exit():
	animatedsprite2d.stop()
```
## 五、角色移动脚本
```
extends StateBase
@export var player: Player
@export var animatedsprite2d: AnimatedSprite2D
@export var statemachine: StateMachine
@export var SPEED = 100
func _ready():
	state_name = "move"

func enter(_msg : Dictionary = {}):
	pass
	
func update(_delta : float):
	animatedsprite2d.play(player.current_direction + "_move")
	if !Input.get_vector("left","right","up","down"):
		statemachine.transition_to("idle")

func physics_update(_delta : float):
	var input = Input.get_vector("left", "right", "up", "down")
	if input != Vector2.ZERO:
		if abs(input.x) > abs(input.y):
			player.current_direction = "right" if input.x > 0 else "left"
		else:
			player.current_direction = "down" if input.y > 0 else "up"
		player.velocity = input.normalized() * SPEED
		player.move_and_slide()

func exit():
	animatedsprite2d.stop()
```
状态机能够很好的对角色的状态进行调整，使制作者能够迅速有效的制作多状态的角色。