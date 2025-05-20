# MOVEMENT

## General flow
Other than the DISPLAY module this one may be a bit confusing. Well maybe not as much as that one, but anyway. So this module is intended to accomodate the movement of the player according to the keys that are pressed. The initial function `move_player` is the first line of the `init_render` function of the rendering function from the display mod. This function is meant to catch the key press (or release) and connect it to the movement on screen.

### Player movement
The first part of the `move_player` function just sets-up how fast the movement & rotation is along with the size of the collision buffer.

then each key press calls a corresponding movement function that calculates the proper movement vector and *attempts* to execute the movement.

Prior to any lines that register linear movement, a function to register a rotation is set up.The function first checks the rotation state flags. If `left_rotate` is active (left key is pressed), the function decreases the player's viewing angle by the provided `angle_speed` parameter (and vice versa).

The second part of the function handles angle normalization, ensuring the angle always remains within the valid range of 0 to 2π radians (a complete circle). If the angle exceeds 2π after a right rotation, it wraps back to 0. Conversely, if the angle becomes negative after a left rotation, it wraps to 2π. 

In the linear movement part, using the `move_down` as an example, the function first determines the direction components based on the player's current orientation. It examines the cosine and sine of the player's angle to set direction indicators (dir_x and dir_y). These binary values (1 or -1) capture which quadrant the player is facing in. For the x-axis, a positive cosine means the player should move right when backing up, while a negative cosine indicates movement to the left. Similarly, for the y-axis, a positive sine means downward movement, while a negative sine indicates upward movement.

After establishing these direction indicators, the function calls `try_move`, which handles the actual displacement calculation and collision detection. 

### Collision detection
The `try_move` function basically determines if the (intended) movement is valid or not. Instead of directly proceeding with the movement, the function first checks if this movement would result in a collision.

The function begins by calculating potential new coordinates for the player by adding the requested displacement values (dx and dy) to the player's current position. 

Then the function calls `touch` with coordinates that include a small buffer in the direction of movement. This buffer is calculated by multiplying the collision_buffer value (which defines how far to check ahead) by the direction indicators (dir_x and dir_y, which are either 1 or -1 depending on movement direction). This approach creates a small probing area in front of the player to detect potential collisions before they occur.

If `touch` returns false (meaning no wall or obstacle was detected), the player's coordinates are updated to the new position. Otherwise, if a collision would occur, the player's position remains.

The `touch` function is the core mechanism. It first performs a quick boundary check, immediately returning true (indicating collision) if either coordinate is negative, which would be outside a valid map.

Next, it converts the floating-point world coordinates to integer grid coordinates by dividing by the size of each map cell (BLOCK). The addition of a small value (0.001) helps prevent floating-point precision issues that might occur during the conversion. This transforms the player's absolute position in the world into the corresponding map grid cell.

Then a boundary check is done to ensure the calculated grid coordinates are within the map's dimensions. If the coordinates exceed the map's rows (or columns), or if they're negative (which should be caught by the first check but is included for robustness), the function returns true to indicate a collision.