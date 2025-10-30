# WWDAC-CODE-PROTOTYPE-A2
# Light Path - Stress Relief Game


# ==================== SPEED SELECTION VARIABLES ====================
# I wanted players to choose their own difficulty level
# These variables track which speed option the player is on
game_started = False  # becomes True when player confirms speed choice
selected_speed = 1  # stores 1, 2, or 3 depending on selection
menu_cursor = 1  # tracks which option has the yellow arrow (starts at 1)

# I set up three different speed values after testing what felt right
speed_slow = 20  # felt most calming during my testing
speed_medium = 35  # good middle ground
speed_fast = 50  # more challenging but still manageable

current_player_speed = speed_slow  # starts slow, updates when player picks

# ==================== GAME VARIABLES ====================
# For the wave animation, I needed to track each point's position in the wave cycle
pos7 = 0
pos6 = 0
pos5 = 0
pos4 = 0
pos3 = 0
pos2 = 0
pos1 = 0
cycle_length = 0  # I'll set this to 240 to make waves move slowly
wave_range = 0  # controls how tall the waves are
base_y = 0  # the center height that waves oscillate around
animation_timer = 0  # increments each frame to animate the waves

# Player input tracking
move_y = 0  # stores vertical controller input
move_x = 0  # stores horizontal controller input

# Distance calculations - needed to check if player stays in tunnel
min_dist = 0
dist7 = 0
dist6 = 0
dist5 = 0
dist4 = 0
dist3 = 0
dist2 = 0
dist1 = 0
distance_to_center = 0
ball_y2 = 0  # temporary variable for calculations
ball_x2 = 0  # temporary variable for calculations

# Rendering variables
steps = 0  # number of interpolation points between tunnel segments

# Game rule variables
path_tolerance = 0  # how many pixels away from center is acceptable
max_off_path_time = 0  # maximum frames player can be outside tunnel

# The tunnel is made of 7 points that create a path across the screen
# I positioned them evenly so the tunnel spans the full width
center7_y = 0
center7_x = 0
center6_y = 0
center6_x = 0
center5_y = 0
center5_x = 0
center4_y = 0
center4_x = 0
center3_y = 0
center3_x = 0
center2_y = 0
center2_x = 0
center1_y = 0
center1_x = 0

# Core game state
player_ball: Sprite = None  # will hold the ball sprite once created
game_over = False  # tracks if player has lost
off_path_timer = 0  # counts frames spent outside tunnel

# Tunnel dimensions - I tested different sizes and settled on these
tunnel_width = 25
tunnel_half_width = 12  # using half-width makes the math easier later

# ==================== INITIAL SETUP ====================
scene.set_background_color(1)  # dark blue creates a calm atmosphere

# Setting up the 7 tunnel center points
# I spread them evenly across the 160-pixel wide screen
center1_x = 0  # leftmost edge
center1_y = 60  # vertical center of 120-pixel tall screen
center2_x = 25
center2_y = 60
center3_x = 50
center3_y = 60
center4_x = 75  # middle of screen
center4_y = 60
center5_x = 100
center5_y = 60
center6_x = 125
center6_y = 60
center7_x = 160  # rightmost edge
center7_y = 60

# Setting game rules
max_off_path_time = 180  # at 60fps, this gives player 3 seconds
path_tolerance = tunnel_half_width - 2  # 10 pixels feels fair

# ==================== MENU FUNCTIONS ====================

def show_speed_menu():
    # This function draws the speed selection screen
    # I wanted it to look clean and easy to understand
    screen.fill_rect(0, 0, 160, 128, 1)
    screen.print("LIGHT PATH", 45, 20, 15)
    screen.print("Choose Your Speed:", 25, 35, 8)
    
    # Speed option 1 - SLOW
    if menu_cursor == 1:
        screen.print("> 1. SLOW", 40, 50, 5)
        screen.print("  (Relaxing)", 45, 58, 6)
    else:
        screen.print("  1. SLOW", 40, 50, 8)
        screen.print("  (Relaxing)", 45, 58, 6)
    
    # Speed option 2 - MEDIUM
    if menu_cursor == 2:
        screen.print("> 2. MEDIUM", 40, 68, 5)
        screen.print("  (Balanced)", 45, 76, 6)
    else:
        screen.print("  2. MEDIUM", 40, 68, 8)
        screen.print("  (Balanced)", 45, 76, 6)
    
    # Speed option 3 - FAST
    if menu_cursor == 3:
        screen.print("> 3. FAST", 40, 86, 5)
        screen.print("  (Responsive)", 45, 94, 6)
    else:
        screen.print("  3. FAST", 40, 86, 8)
        screen.print("  (Responsive)", 45, 94, 6)
    
    screen.print("UP/DOWN to select", 15, 108, 7)
    screen.print("A to confirm", 55, 118, 7)

def handle_menu_input():
    # Processes button presses in the menu
    global menu_cursor, selected_speed, game_started, current_player_speed
    
    if controller.up.is_pressed():
        pause(200)
        menu_cursor = menu_cursor - 1
        if menu_cursor < 1:
            menu_cursor = 3
    
    if controller.down.is_pressed():
        pause(200)
        menu_cursor = menu_cursor + 1
        if menu_cursor > 3:
            menu_cursor = 1
    
    if controller.A.is_pressed():
        pause(200)
        selected_speed = menu_cursor
        game_started = True
        
        if selected_speed == 1:
            current_player_speed = speed_slow
        elif selected_speed == 2:
            current_player_speed = speed_medium
        else:
            current_player_speed = speed_fast
        
        # Create the ball sprite NOW when game starts
        create_player_ball()

def create_player_ball():
    # Creates the ball sprite when gameplay starts
    global player_ball
    
    player_ball = sprites.create(img("""
            . . . 1 2 2 2 2 1 . . .
            . 1 2 2 2 2 2 2 2 2 1 .
            . 2 2 2 4 4 4 4 2 2 2 .
            1 2 2 4 2 2 2 2 4 2 2 1
            2 2 2 4 2 2 2 2 4 2 2 2
            2 2 4 2 2 1 5 2 2 4 2 2
            2 2 4 2 2 5 1 2 2 4 2 2
            2 2 2 4 2 2 2 2 4 2 2 2
            1 2 2 4 2 2 2 2 4 2 2 1
            . 2 2 2 4 4 4 4 2 2 2 .
            . 1 2 2 2 2 2 2 2 2 1 .
            . . . 1 2 2 2 2 1 . . .
            """), SpriteKind.player)
    
    player_ball.set_position(80, 60)
    player_ball.set_velocity(0, 0)

# ==================== CORE GAME FUNCTIONS ====================

def find_closest_tunnel_center(ball_x: number, ball_y: number):
    # Calculates how far the ball is from the tunnel path
    # I use Manhattan distance because it's faster than Pythagorean
    global dist1, dist2, dist3, dist4, dist5, dist6, dist7, min_dist
    
    dist1 = abs(ball_x - center1_x) + abs(ball_y - center1_y)
    dist2 = abs(ball_x - center2_x) + abs(ball_y - center2_y)
    dist3 = abs(ball_x - center3_x) + abs(ball_y - center3_y)
    dist4 = abs(ball_x - center4_x) + abs(ball_y - center4_y)
    dist5 = abs(ball_x - center5_x) + abs(ball_y - center5_y)
    dist6 = abs(ball_x - center6_x) + abs(ball_y - center6_y)
    dist7 = abs(ball_x - center7_x) + abs(ball_y - center7_y)
    
    min_dist = dist1
    if dist2 < min_dist:
        min_dist = dist2
    if dist3 < min_dist:
        min_dist = dist3
    if dist4 < min_dist:
        min_dist = dist4
    if dist5 < min_dist:
        min_dist = dist5
    if dist6 < min_dist:
        min_dist = dist6
    if dist7 < min_dist:
        min_dist = dist7
    
    return min_dist

def check_game_over():
    # Monitors if player stays in tunnel, triggers game over after 3 seconds
    global ball_x2, ball_y2, distance_to_center, off_path_timer, game_over
    
    if game_over or not game_started:
        return
    
    ball_x2 = player_ball.x
    ball_y2 = player_ball.y
    distance_to_center = find_closest_tunnel_center(ball_x2, ball_y2)
    
    if distance_to_center <= path_tolerance:
        off_path_timer = 0
    else:
        off_path_timer = off_path_timer + 1
        if off_path_timer >= max_off_path_time:
            game_over = True
            player_ball.set_velocity(0, 0)
            game.splash("Game Over!", "Press A to restart")

def restart_game():
    # Resets game state and returns to menu
    global game_over, off_path_timer, game_started
    
    if game_over and controller.A.is_pressed():
        pause(200)
        game_over = False
        off_path_timer = 0
        game_started = False

def update_player():
    # Updates ball position based on controller input
    global move_x, move_y
    
    if game_over or not game_started:
        return
    
    move_x = controller.dx()
    move_y = controller.dy()
    player_ball.set_velocity(move_x * current_player_speed, move_y * current_player_speed)
    
    if player_ball.x < 8:
        player_ball.x = 8
    if player_ball.x > 152:
        player_ball.x = 152
    if player_ball.y < 8:
        player_ball.y = 8
    if player_ball.y > 112:
        player_ball.y = 112

def update_tunnel_path():
    # Animates the tunnel wave motion
    # This creates the flowing visual effect
    global animation_timer, base_y, wave_range, cycle_length
    global pos1, pos2, pos3, pos4, pos5, pos6, pos7
    global center1_y, center2_y, center3_y, center4_y, center5_y, center6_y, center7_y
    
    animation_timer = animation_timer + 1
    base_y = 60
    wave_range = 15
    cycle_length = 240
    
    # Each point is offset to create flowing wave
    pos1 = animation_timer % cycle_length
    pos2 = (animation_timer + 40) % cycle_length
    pos3 = (animation_timer + 80) % cycle_length
    pos4 = (animation_timer + 120) % cycle_length
    pos5 = (animation_timer + 160) % cycle_length
    pos6 = (animation_timer + 200) % cycle_length
    pos7 = (animation_timer + 240) % cycle_length
    
    def get_wave_height(pos: number):
        # Triangle wave function for smooth up-down motion
        if pos < 60:
            return Math.idiv(pos * wave_range, 60)
        elif pos < 120:
            return wave_range - Math.idiv((pos - 60) * wave_range, 60)
        elif pos < 180:
            return 0 - Math.idiv((pos - 120) * wave_range, 60)
        else:
            return 0 - wave_range + Math.idiv((pos - 180) * wave_range, 60)
    
    center1_y = base_y + get_wave_height(pos1)
    center2_y = base_y + get_wave_height(pos2)
    center3_y = base_y + get_wave_height(pos3)
    center4_y = base_y + get_wave_height(pos4)
    center5_y = base_y + get_wave_height(pos5)
    center6_y = base_y + get_wave_height(pos6)
    center7_y = base_y + get_wave_height(pos7)
    
    # Keep points on screen
    if center1_y < 25: center1_y = 25
    if center1_y > 95: center1_y = 95
    if center2_y < 25: center2_y = 25
    if center2_y > 95: center2_y = 95
    if center3_y < 25: center3_y = 25
    if center3_y > 95: center3_y = 95
    if center4_y < 25: center4_y = 25
    if center4_y > 95: center4_y = 95
    if center5_y < 25: center5_y = 25
    if center5_y > 95: center5_y = 95
    if center6_y < 25: center6_y = 25
    if center6_y > 95: center6_y = 95
    if center7_y < 25: center7_y = 25
    if center7_y > 95: center7_y = 95

def draw_tunnel_walls():
    # Renders the visual tunnel by connecting the 7 points
    global steps
    
    # Color changes based on player status (visual feedback)
    if off_path_timer > Math.idiv(max_off_path_time, 2):
        wall_color = 2
        center_color = 2
    elif off_path_timer > 0:
        wall_color = 4
        center_color = 4
    else:
        wall_color = 8
        center_color = 9
    
    steps = 30  # interpolation points between centers
    
    # Draw all 6 tunnel segments
    # Each segment connects two adjacent center points
    
    # Segment 1
    i = 0
    while i < steps:
        cx = center1_x + Math.idiv((center2_x - center1_x) * i, steps)
        cy = center1_y + Math.idiv((center2_y - center1_y) * i, steps)
        screen.fill_rect(cx, cy - tunnel_half_width - 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width - 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width + 1, 1, tunnel_half_width * 2 - 1, 15)
        i += 1
    
    # Segment 2
    i = 0
    while i < steps:
        cx = center2_x + Math.idiv((center3_x - center2_x) * i, steps)
        cy = center2_y + Math.idiv((center3_y - center2_y) * i, steps)
        screen.fill_rect(cx, cy - tunnel_half_width - 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width - 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width + 1, 1, tunnel_half_width * 2 - 1, 15)
        i += 1
    
    # Segment 3
    i = 0
    while i < steps:
        cx = center3_x + Math.idiv((center4_x - center3_x) * i, steps)
        cy = center3_y + Math.idiv((center4_y - center3_y) * i, steps)
        screen.fill_rect(cx, cy - tunnel_half_width - 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width - 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width + 1, 1, tunnel_half_width * 2 - 1, 15)
        i += 1
    
    # Segment 4
    i = 0
    while i < steps:
        cx = center4_x + Math.idiv((center5_x - center4_x) * i, steps)
        cy = center4_y + Math.idiv((center5_y - center4_y) * i, steps)
        screen.fill_rect(cx, cy - tunnel_half_width - 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width - 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width + 1, 1, tunnel_half_width * 2 - 1, 15)
        i += 1
    
    # Segment 5
    i = 0
    while i < steps:
        cx = center5_x + Math.idiv((center6_x - center5_x) * i, steps)
        cy = center5_y + Math.idiv((center6_y - center5_y) * i, steps)
        screen.fill_rect(cx, cy - tunnel_half_width - 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width - 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width + 1, 1, tunnel_half_width * 2 - 1, 15)
        i += 1
    
    # Segment 6
    i = 0
    while i < steps:
        cx = center6_x + Math.idiv((center7_x - center6_x) * i, steps)
        cy = center6_y + Math.idiv((center7_y - center6_y) * i, steps)
        screen.fill_rect(cx, cy - tunnel_half_width - 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width - 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 1, 1, 1, wall_color)
        screen.fill_rect(cx, cy + tunnel_half_width + 2, 1, 1, wall_color)
        screen.fill_rect(cx, cy - tunnel_half_width + 1, 1, tunnel_half_width * 2 - 1, 15)
        i += 1
    
    # Draw center guideline (dotted)
    i = 0
    while i < steps:
        cx = center1_x + Math.idiv((center2_x - center1_x) * i, steps)
        cy = center1_y + Math.idiv((center2_y - center1_y) * i, steps)
        screen.fill_rect(cx, cy, 1, 1, center_color)
        i += 3
    
    i = 0
    while i < steps:
        cx = center2_x + Math.idiv((center3_x - center2_x) * i, steps)
        cy = center2_y + Math.idiv((center3_y - center2_y) * i, steps)
        screen.fill_rect(cx, cy, 1, 1, center_color)
        i += 3
    
    i = 0
    while i < steps:
        cx = center3_x + Math.idiv((center4_x - center3_x) * i, steps)
        cy = center3_y + Math.idiv((center4_y - center3_y) * i, steps)
        screen.fill_rect(cx, cy, 1, 1, center_color)
        i += 3
    
    i = 0
    while i < steps:
        cx = center4_x + Math.idiv((center5_x - center4_x) * i, steps)
        cy = center4_y + Math.idiv((center5_y - center4_y) * i, steps)
        screen.fill_rect(cx, cy, 1, 1, center_color)
        i += 3
    
    i = 0
    while i < steps:
        cx = center5_x + Math.idiv((center6_x - center5_x) * i, steps)
        cy = center5_y + Math.idiv((center6_y - center5_y) * i, steps)
        screen.fill_rect(cx, cy, 1, 1, center_color)
        i += 3
    
    i = 0
    while i < steps:
        cx = center6_x + Math.idiv((center7_x - center6_x) * i, steps)
        cy = center6_y + Math.idiv((center7_y - center6_y) * i, steps)
        screen.fill_rect(cx, cy, 1, 1, center_color)
        i += 3

# ==================== MAIN GAME LOOP ====================

def on_update():
    # Called every frame to update game state
    if not game_started:
        handle_menu_input()
    else:
        update_tunnel_path()
        update_player()
        check_game_over()
        restart_game()

def on_paint():
    # Called every frame to render visuals
    if not game_started:
        show_speed_menu()
    else:
        draw_tunnel_walls()
        
        if off_path_timer > 0 and not game_over:
            remaining_time = Math.idiv(max_off_path_time - off_path_timer, 60) + 1
            if remaining_time <= 3:
                screen.print("TUNNEL! " + str(remaining_time), 5, 5, 2)
        
        if game_started:
            if selected_speed == 1:
                screen.print("SLOW", 130, 5, 7)
            elif selected_speed == 2:
                screen.print("MED", 130, 5, 7)
            else:
                screen.print("FAST", 130, 5, 7)

game.on_update(on_update)
game.on_paint(on_paint)
