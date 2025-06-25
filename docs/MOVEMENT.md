# MOVEMENT

## General flow
Other than the DISPLAY module this one may be a bit confusing. Well maybe not as much as that one, but anyway. So this module is intended to accomodate the movement of the player according to the keys that are pressed. The initial function `move_player` is the first line of the `init_render` function of the rendering function from the display mod. This function is meant to catch the key press (or release) and connect it to the movement on screen.

### Player movement
```c
void	move_player(t_player *player, t_data *cub3d)
{
	cub3d->move.speed = MOVE_SPEED;
	cub3d->move.angle_speed = ANGLE_SPEED;
	cub3d->move.collision_buffer = BLOCK * COLBUF_FACTOR;
	move_rotate(player, cub3d->move.angle_speed);
	cub3d->move.cos_angle = cos(player->angle);
	cub3d->move.sin_angle = sin(player->angle);
	if (player->key_down)
		move_down(player, cub3d);
	if (player->key_up)
		move_up(player, cub3d);
	if (player->key_right)
		move_right(player, cub3d);
	if (player->key_left)
		move_left(player, cub3d);
}
```
The first part of the `move_player` function just sets-up how fast the movement & rotation is along with the size of the collision buffer.

then each key press calls a corresponding movement function that calculates the proper movement vector and *attempts* to execute the movement.

Prior to any lines that register linear movement, a function to register a rotation is set up. The function first checks the rotation state flags. If `left_rotate` is active (left key is pressed), the function decreases the player's viewing angle by the provided `angle_speed` parameter (and vice versa).

```c
void	move_rotate(t_player *player, float angle_speed)
{
	if (player->left_rotate)
		player->angle -= angle_speed;
	if (player->right_rotate)
		player->angle += angle_speed;
	if (player->angle > 2 * PI)
		player->angle = 0;
	if (player->angle < 0)
		player->angle = 2 * PI;
}
```

The second part of the function handles angle normalization, ensuring the angle always remains within the valid range of 0 to 2π radians (a complete circle). If the angle exceeds 2π after a right rotation, it wraps back to 0. Conversely, if the angle becomes negative after a left rotation, it wraps to 2π. 

In the linear movement part, using the `move_down` as an example, the function first determines the direction components based on the player's current orientation. It examines the cosine and sine of the player's angle to set direction indicators (dir_x and dir_y). These binary values (1 or -1) capture which quadrant the player is facing in. For the x-axis, a positive cosine means the player should move right when backing up, while a negative cosine indicates movement to the left. Similarly, for the y-axis, a positive sine means downward movement, while a negative sine indicates upward movement.

```c
void	move_down(t_player *player, t_data *cub3d)
{
	if (cub3d->move.cos_angle > 0)
		cub3d->move.dir_x = 1;
	else
		cub3d->move.dir_x = -1;
	if (cub3d->move.sin_angle > 0)
		cub3d->move.dir_y = 1;
	else
		cub3d->move.dir_y = -1;
	try_move(player, cub3d->move.cos_angle * cub3d->move.speed,
		cub3d->move.sin_angle * cub3d->move.speed, cub3d);
}
```

After establishing these direction indicators, the function calls `try_move`, which handles the actual displacement calculation and collision detection. 

### Collision detection
The `try_move` function basically determines if the (intended) movement is valid or not. Instead of directly proceeding with the movement, the function first checks if this movement would result in a collision.

```c
void	try_move(t_player *player, float dx, float dy, t_data *cub3d)
{
	float	new_x;
	float	new_y;

	new_x = player->x + dx;
	new_y = player->y + dy;
	if (!touch(new_x + cub3d->move.collision_buffer * cub3d->move.dir_x,
			new_y + cub3d->move.collision_buffer * cub3d->move.dir_y, cub3d))
	{
		player->x = new_x;
		player->y = new_y;
	}
}
```

The function begins by calculating potential new coordinates for the player by adding the requested displacement values (dx and dy) to the player's current position. 

Then the function calls `touch` with coordinates that include a small buffer in the direction of movement. This buffer is calculated by multiplying the collision_buffer value (which defines how far to check ahead) by the direction indicators (dir_x and dir_y, which are either 1 or -1 depending on movement direction). This approach creates a small probing area in front of the player to detect potential collisions before they occur.

If `touch` returns false (meaning no wall or obstacle was detected), the player's coordinates are updated to the new position. Otherwise, if a collision would occur, the player's position remains.

```c
bool	touch(float pos_x, float pos_y, t_data *cub3d)
{
	int	x;
	int	y;

	if (pos_x < 0 || pos_y < 0)
		return (true);
	x = (int)(pos_x / BLOCK);
	y = (int)(pos_y / BLOCK);
	if (y >= cub3d->map_info.map_rows || x >= cub3d->map_info.map_cols)
		return (true);
	if (cub3d->map_info.map[y] && x < (int)ft_strlen(cub3d->map_info.map[y])
		&& cub3d->map_info.map[y][x] == '1')
		return (true);
	return (false);
}
```

The `touch` function is the core mechanism. It first performs a quick boundary check, immediately returning true (indicating collision) if either coordinate is negative, which would be outside a valid map.

Next, it converts the floating-point world coordinates to integer grid coordinates by dividing by the size of each map cell (BLOCK).This transforms the player's absolute position in the world into the corresponding map grid cell.

Then a boundary check is done to ensure the calculated grid coordinates are within the map's dimensions. If the coordinates exceed the map's rows (or columns) the function returns true to indicate a collision.

As well, if the position is within bounds, the function examines the actual map grid. It checks if the grid cell contains a '1' (a wall). If a wall is detected, the function returns true; otherwise, it returns false, indicating the position is safe for the movement.
